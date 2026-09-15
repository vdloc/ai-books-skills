# Patterns — Using Amazon Bedrock

Concrete techniques and design patterns extracted across all 10 chapters.

## Prompt Engineering Patterns

### Chain of Thought Reasoning
**When to use**: multistep problems where reasoning transparency matters (math, logic, complex analysis) and understanding *why* an answer was reached is as important as the answer.
**How**: instruct the model to "describe your process step-by-step" rather than jumping to a conclusion; the model narrates intermediate reasoning steps before the final answer.
**Trade-offs**: slower and costlier (more tokens generated), but easier to debug and more accurate on complex tasks.

### Meta Prompting
**When to use**: when you're unsure how to phrase a prompt for a creative or open-ended task and want the model to help design the prompt itself.
**How**: ask the model to propose the best prompt/strategy for a goal before generating the final output (related to the Plan-and-Act paradigm — separate planning from execution).
**Trade-offs**: adds a round-trip but often yields significantly better-targeted output than a first-attempt prompt.

### Templating
**When to use**: content generation tasks requiring consistent structure/format across many outputs (reports, emails, structured documents).
**How**: define a fixed skeleton (sections, headers) and let the model fill in content within those constraints; use Bedrock Prompt Management to version and reuse templates.
**Trade-offs**: reduces output randomness and formatting drift; slightly limits creative flexibility by design (that's the point).

### Few-Shot Prompting with PromptTemplate
**When to use**: task benefits from pattern demonstration (classification, structured generation) but doesn't justify fine-tuning.
**How**: embed 2-3 worked examples directly in the prompt via LangChain's `PromptTemplate`, then present the new input for the model to complete in the same pattern.
**Trade-offs**: larger prompts (more tokens/cost) but meaningfully better format/quality adherence than zero-shot.

## Application Architecture Patterns

### Conversational Memory Chain
**When to use**: any multi-turn chatbot/assistant that needs to reference prior exchanges.
**How**: use LangChain's `RunnableWithMessageHistory` + `InMemoryChatMessageHistory` keyed by `session_id`; seed system prompt only on the first turn, subsequent turns send only the new user message.
**Trade-offs**: in-memory history doesn't survive process restarts — swap in DynamoDB or another persistent store for production.

### Streaming Response Handling
**When to use**: chat interfaces or any UX where perceived latency matters more than total completion time.
**How**: use `invoke_model_with_response_stream` (or `InvokeModelWithResponseStream`), iterate over chunk events, decode and append incrementally; wrap in try/except for stream interruption handling.
**Trade-offs**: more complex client-side code (partial-state handling) vs. simple synchronous `invoke_model`.

### Prompt Caching
**When to use**: repeated requests sharing a stable prefix (system instructions, few-shot examples) — high-volume or multi-turn conversational apps.
**How**: mark cache checkpoints at token boundaries; cache only static content, never dynamic user input; monitor `CacheReadInputTokens`/`CacheWriteInputTokens` via CloudWatch.
**Trade-offs**: up to 90% token cost reduction and 85% latency reduction, but 5-minute sliding TTL means low-frequency requests won't benefit.

## RAG Patterns

### Standard RAG Pipeline
**When to use**: need current, domain-specific, or proprietary-data-grounded responses beyond the model's training data.
**How**: chunk documents (e.g., 1,000 chars, 100-char overlap) → embed (Titan Embeddings) → index in a vector store (FAISS/OpenSearch) → embed the query → similarity search top-N chunks → augment prompt → generate.
**Trade-offs**: adds retrieval latency; quality bounded by source-data quality and chunking strategy.

### Knowledge Base as Schema Grounding (Text-to-SQL)
**When to use**: natural-language-to-structured-query systems (SQL, API calls) where the LLM needs to know exact table/column names and types.
**How**: instead of storing prose documents, store structured metadata (table_name, columns with name/type/description) in a Bedrock Knowledge Base; retrieve-and-generate against this metadata to ground SQL generation.
**Trade-offs**: requires maintaining accurate, up-to-date metadata as schema evolves.

### Retry-with-Error-Feedback
**When to use**: any LLM-generated structured output (SQL, code, JSON) prone to syntax/logic errors.
**How**: execute the generated output; on failure, feed the error message back into a regenerated prompt ("the previous query resulted in this error: ..."); bound retries with a max count.
**Trade-offs**: adds latency on failure paths but significantly improves reliability of structured generation vs. one-shot.

## Fine-Tuning Patterns

### Instruction-Formatted JSONL Dataset
**When to use**: preparing any Bedrock fine-tuning dataset.
**How**: format each example as `{"prompt": "<instruction + input>", "completion": "<expected output>"}` on its own JSONL line; filter by max character length; shuffle and cap at target sample count.
**Trade-offs**: requires upfront data engineering investment; quality of this dataset directly bounds fine-tuning outcome quality.

### PEFT (Parameter-Efficient Fine-Tuning)
**When to use**: need behavioral adaptation but full-model fine-tuning cost/time isn't justified.
**How**: use LoRA, adapters, or prompt tuning to update <1% of model parameters instead of full weights.
**Trade-offs**: dramatically cheaper/faster than full fine-tuning, at potentially slightly lower adaptation depth for very complex domain shifts.

## Security Patterns

### Guardrails + Contextual Grounding
**When to use**: any production chatbot/agent in a regulated or high-stakes domain (finance, healthcare, legal).
**How**: define denied topics with natural-language descriptions + example phrases, set content filter thresholds (LOW/MEDIUM/HIGH per category), and use contextual grounding tags to score responses on grounding (factual alignment) and relevance (answers the query).
**Trade-offs**: stricter guardrails reduce harmful/off-topic output but increase false positives and degrade UX for borderline-legitimate queries.

### Least-Privilege Agent IAM Scoping
**When to use**: every Bedrock agent deployment, always.
**How**: never attach `AmazonBedrockFullAccess`; create custom policies scoping agents to exactly the actions they need (e.g., `bedrock:InvokeAgent`, `bedrock:PrepareAgent`); use different agents with different IAM roles for different permission levels (e.g., read-only FinanceAgent vs. write-capable DevOpsAgent).
**Trade-offs**: more upfront IAM policy design work, but dramatically reduces blast radius if an agent is compromised or misconfigured.

### STRIDE Threat Modeling
**When to use**: architecture design phase for any Bedrock deployment, before production rollout.
**How**: walk each of Spoofing/Tampering/Repudiation/Information Disclosure/DoS/Elevation of Privilege against your specific architecture; write a threat statement (actor + asset + method + impact) for each identified risk, prioritize (High/Medium/Low), design targeted mitigations.
**Trade-offs**: upfront time investment, but catches architecture-level vulnerabilities that ad-hoc security review misses.

## Performance & Cost Patterns

### Prompt Chaining via Step Functions
**When to use**: complex generation tasks decomposable into sequential sub-tasks (e.g., overview → plot → themes → style → synthesis for a book review).
**How**: each Step Functions state invokes Bedrock (directly via `arn:aws:states:::bedrock:invokeModel`, or via a Lambda intermediary); outputs from earlier states feed into later states' prompts via `ResultPath` and `States.Format`.
**Trade-offs**: more moving parts than a single call, but each stage is independently debuggable/retriable and the final synthesis benefits from structured intermediate outputs.

### LLM Cascading
**When to use**: high-volume workloads where most requests are simple but a minority need advanced capability.
**How**: route requests to a smaller/cheaper model first; escalate to a larger model only when the smaller model's output is insufficient.
**Trade-offs**: reduces average cost significantly; requires a reliable mechanism to detect when escalation is needed.

### Batch Inference for Non-Urgent Bulk Work
**When to use**: large-scale content generation that can tolerate scheduled/delayed processing (product descriptions, catalog updates).
**How**: submit a CSV/dataset of prompts as a single batch job rather than individual real-time calls.
**Trade-offs**: 50% cost discount vs. on-demand, but unsuitable for latency-sensitive or event-driven use cases.

### Zero-ETL Data Integration
**When to use**: need near-real-time analytics on transactional data without building custom ETL pipelines.
**How**: use native integrations (e.g., Aurora → Redshift Zero-ETL Integration) that replicate data automatically; query the replicated data directly for RAG/generation grounding.
**Trade-offs**: only available for supported source/target service pairs; eliminates pipeline maintenance overhead where it applies.
