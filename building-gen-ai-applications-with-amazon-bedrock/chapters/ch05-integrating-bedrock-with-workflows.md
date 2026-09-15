# Chapter 5: Integrating Bedrock with Existing Workflows

## Core Idea
Bedrock is meant to be wired into the rest of AWS — S3, Lambda, DynamoDB, API Gateway, SageMaker — via event-driven, IAM-scoped, resilient integration patterns, not called in isolation from a notebook; the chapter's throughline is treating throttling, batch vs. streaming, and state/history management as first-class architectural decisions, not afterthoughts.

## Frameworks Introduced
- **Exponential Backoff with Jitter for rate limiting**: the AWS-recommended way to handle `ThrottlingException`.
  - When to use: any production Bedrock client, replacing manual `time.sleep()` retry loops.
  - How: configure the Boto3 client's built-in retry handler via `botocore.config.Config` using the `standard` (or `adaptive`) retry mode — wait time doubles each retry (1s, 2s, 4s, 8s...) with added random jitter, preventing a "thundering herd" where multiple throttled clients retry in lockstep and re-overload the service.
- **Batch vs. Real-Time (Streaming) Architecture Decision**: choosing invocation pattern by workload shape.
  - When to use: at architecture design time, before choosing `InvokeModel` vs `InvokeModelWithResponseStream` vs `CreateModelInvocationJob` (batch).
  - How: use real-time synchronous `InvokeModel` for low-latency single requests; use `InvokeModelWithResponseStream`/`ConverseStream` when the client needs to render tokens as they arrive (chat UIs); use Batch Inference (`CreateModelInvocationJob`) for large, non-interactive volumes (e.g. bulk summarization of a document corpus) where cost efficiency matters more than per-request latency.
- **Decouple Prompt Logic from Pipeline Control**: an architectural pattern for maintainability.
  - When to use: any application beyond a throwaway prototype.
  - How: store prompt templates externally (e.g. S3, Parameter Store, or a "Decoupled Prompt Engine") rather than hardcoding them in Lambda/application code, so prompt iteration doesn't require redeploying application logic — the book explicitly warns against "storing prompts as plain [hardcoded strings]".

## Key Concepts
- **Control Plane vs Data Plane (Runtime)**: `bedrock` client = control plane (model discovery, provisioning, guardrail/customization management); `bedrock-runtime` client = data plane (actual inference — `InvokeModel`, `Converse`, `ConverseStream`).
- **`Converse` / `ConverseStream` API**: Bedrock's unified conversation API that normalizes request/response format across different model providers, reducing the need for model-specific payload formatting that raw `InvokeModel` requires.
- **Bedrock Flows**: a visual/managed way to orchestrate multi-step Bedrock pipelines (prompt → model → post-processing) without hand-rolling the orchestration code.
- **Event-driven triggers**: the standard integration shape — e.g. an S3 `ObjectCreate` event triggers a Lambda function that reads the new object and invokes Bedrock, writing results to DynamoDB; this pattern recurs across the S3, DynamoDB, and API Gateway integration sections.
- **`DynamoDBChatMessageHistory`**: a pattern for persisting multi-turn conversation state in DynamoDB, prepending retrieved chat history to new user turns before calling Bedrock — necessary because Bedrock itself is stateless between calls.
- **Least-privilege IAM Action Prefixes**: scoping policies to specific actions (`bedrock:InvokeModel`, `bedrock:CreateGuardrail`, `bedrock:CreateModelCustomizationJob`, `bedrock:CreateProvisionedModelThroughput`) instead of broad `AmazonBedrockFullAccess`, now that the application's actual required actions are known (contrast with Ch2's "start broad" approach for initial learning).

## Mental Models
- Treat **Lambda + S3 + DynamoDB as Bedrock's default serverless integration triangle**: S3 for input/output object storage, Lambda for the invocation/glue logic, DynamoDB for state (chat history, job status) — most of the chapter's worked patterns are variations on this triangle.
- Use the **Batch vs. Streaming decision as a cost/latency trade**, not a technical default: streaming/real-time costs more per unit of throughput but gives interactivity; batch is cheaper at scale but has no interactivity — pick based on whether a human is waiting on the response.
- Think of **client-side retry configuration as infrastructure, not error handling** — exponential backoff with jitter belongs in the Boto3 `Config` object at client-creation time, not scattered as manual try/except retry logic through business code.

## Anti-patterns
- **Writing manual `time.sleep()` retry loops for `ThrottlingException`**: the book calls this "counterproductive" — synchronized retries across multiple clients create a thundering herd that worsens throttling; use the Boto3 built-in retry handler instead.
- **Hardcoding prompts inside Lambda function code**: couples prompt iteration to a full deployment cycle; decouple prompt templates into external storage so non-engineers/prompt-owners can iterate independently.
- **Granting `AmazonBedrockFullAccess` to production Lambda execution roles**: appropriate for Ch2's learning phase, but the book calls for narrowing to specific action prefixes (e.g. only `bedrock:InvokeModel` for an inference-only Lambda) once the application's real permission surface is known.
- **Using real-time `InvokeModel` for bulk, non-interactive workloads**: wastes cost/throughput budget on synchronous calls when Batch Inference is cheaper and better suited for large offline jobs like corpus-wide summarization.

## Reference Tables
**Batch vs. Real-Time Bedrock Architectures**
| Dimension | Real-Time (InvokeModel/Converse) | Batch (CreateModelInvocationJob) |
|---|---|---|
| Latency | Low, synchronous | High, asynchronous (job-based) |
| Best for | Interactive apps, chatbots | Bulk/offline processing (corpus summarization, large-scale extraction) |
| Cost model | Per-request, standard/provisioned throughput | Typically more cost-efficient at scale |
| Client API | `InvokeModel`, `InvokeModelWithResponseStream`, `Converse`/`ConverseStream` | `CreateModelInvocationJob` (job lifecycle: submitted → InProgress → Completed/Failed) |

**Common Bedrock Exceptions**
| Exception | Likely Cause |
|---|---|
| `ThrottlingException` | Rate limit exceeded — fix with exponential backoff + jitter, not manual sleep |
| `AccessDeniedException` | IAM policy missing the specific action/resource, or model access not granted (Ch2) |
| `InternalServerException` | Transient AWS-side error — safe to retry with backoff |

## Worked Example
The book walks through wiring an S3-triggered Lambda summarizer: an S3 bucket receives document uploads → an "All object create events" trigger fires a Lambda function (`BedrockSummarizer`) with an execution role (`BedrockLambdaRole`) scoped to `AmazonS3ReadOnlyAccess` + a custom `BedrockInvokePolicy` (least-privilege `bedrock:InvokeModel` only) → the Lambda reads the new S3 object's text content, invokes Bedrock for summarization, and writes the result to a DynamoDB table (`BedrockSummaries`) keyed by `DocumentKey` — illustrating the full event-driven integration triangle (S3 → Lambda → Bedrock → DynamoDB) in one pipeline.

## Key Takeaways
1. Configure exponential backoff with jitter via the Boto3 client's retry config — never hand-roll `time.sleep()` retry logic for throttling.
2. Choose invocation pattern (real-time InvokeModel/Converse, streaming, or Batch Inference) based on whether the workload is interactive or bulk/offline, not by default habit.
3. Decouple prompt templates from application code so prompt changes don't require redeployment.
4. Narrow IAM policies from Ch2's broad learning-phase grants to specific action prefixes once the application's actual permission needs are known.
5. Bedrock is stateless between calls — persist conversation history explicitly (e.g. DynamoDB) if multi-turn context is required.
6. S3 → Lambda → Bedrock → DynamoDB is the default serverless integration shape for most document-processing use cases.

## Connects To
- **Ch2**: IAM setup here narrows the broad policies established in initial environment setup.
- **Ch4**: the RAG pipeline's ingestion stage (S3, chunking) connects directly to this chapter's S3-Lambda integration pattern.
- **Ch6**: batch vs. real-time and throughput/cost decisions here are expanded into full scaling strategy.
