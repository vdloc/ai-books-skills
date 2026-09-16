# Chapter 6: Production Deployment and Enterprise Integration

## Core Idea
Choosing where to run an agent in production (Lambda, ECS, or AgentCore Runtime) is a session-model decision, not a preference: match statelessness/event-driven short tasks to Lambda, always-on/shared-state/multi-agent workloads to ECS, and per-user isolated multi-turn sessions needing built-in auth/policy/observability to AgentCore.

## Frameworks Introduced
- **Production-readiness checklist**: before deploying, verify error handling beyond `try/except: pass`, request-level observability, graceful degradation on model-provider outage, resource limits, that someone else has run the code, rollback capability, and a cost estimate.
  - When to use: as a gate before any production deployment — most "it worked in dev, broke in prod" incidents trace back to skipping one of these.
- **AWS Lambda (serverless/event-driven)**: agent code as a `lambda_handler` triggered by an event (S3 upload, API Gateway request, schedule); model/agent objects initialized *outside* the handler so they persist across warm invocations.
  - When to use: short-lived, stateless, unpredictable-traffic tasks (e.g. per-document contract analysis) — scales to zero when idle.
  - Constraints: statelessness (persist conversation state externally, e.g. DynamoDB, if needed across invocations), cold starts (2-5s on first/scaled invocation; container images are slightly slower than zip but necessary since agent deps routinely exceed the 250MB zip limit — use `PackageType: Image` for up to 10GB), 15-minute execution ceiling (Lambda durable functions with checkpoint-and-replay can extend this, at added design complexity).
- **Amazon ECS (always-on/container-driven)**: agent code as a persistent web server (e.g. FastAPI) behind a load balancer; model/boto3 session initialize once at container start and persist for the container's life.
  - When to use: multi-turn conversations needing warm state, persistent connections (DB pools, OAuth sessions), or multi-agent systems where agents call each other in-process without network hops.
  - How: Fargate (AWS manages servers — start here) vs. EC2 launch type (you manage instances — only if you need something Fargate can't, like GPU); an ECS *service* keeps N replicas running and replaces crashed containers automatically; a *task definition* specifies image, CPU/memory, logging.
- **Amazon Bedrock AgentCore Runtime**: each user session gets a dedicated, framework-agnostic microVM (Strands, LangGraph, CrewAI, Claude Agent SDK, or custom) that terminates and sanitizes memory when the session ends — zero cross-session data leakage.
  - When to use: multi-turn, per-user-isolated workloads needing managed auth/policy/observability without building that infrastructure yourself.
  - How: wrap the agent with `BedrockAgentCoreApp`, decorate the entrypoint with `@app.entrypoint`, call `app.run()` — this auto-creates `/invocations` and `/ping` HTTP endpoints; deploy via `Runtime.configure()` (generates Dockerfile, IAM role, ECR repo) then `Runtime.launch()`. Requires ARM64 (Graviton) container images if bringing your own.
  - Session lifecycle: Active (processing, billed) → Idle (waiting, zero CPU charge, state preserved, default 15-min timeout) → Terminated. Execution ceiling: 15 min sync, 60 min streaming, 8 hours async (signal long jobs via `HealthyBusy` + `add_async_task`/`complete_async_task`).
- **AgentCore Gateway**: point it at an OpenAPI spec / Smithy model / Lambda function / remote MCP server (a "target"), and it auto-generates MCP-compatible tools — no wrapper/integration code, no server to maintain.
  - When to use: enterprises with hundreds of existing REST APIs that would otherwise each need a hand-built MCP wrapper.
  - How: `create_gateway(protocolType='MCP', authorizerType=...)` then `create_gateway_target(targetConfiguration={'mcp': {'openApiSchema': {'s3': {...}}}})` — Gateway parses the spec and creates one tool per endpoint.
  - Semantic search at scale: `x_amz_bedrock_agentcore_search` lets the agent query for relevant tools by embedding similarity instead of loading all N tool definitions into the prompt — essential past ~100+ tools where context bloat and selection accuracy both degrade.
- **AgentCore Identity**: handles both inbound auth (JWT via Cognito/Okta/Entra ID validating callers before they reach the agent) and outbound auth (OAuth 2.0 tokens/API keys for the agent calling external services, stored in a KMS-encrypted token vault, never in code/env vars).
  - How: `@requires_access_token` / `@requires_api_key` decorators handle credential retrieval, injection, and refresh automatically; built-in providers for Google, GitHub, Slack, Salesforce, Atlassian, or custom OAuth 2.0 providers for anything else.
  - Two outbound patterns: user-delegated access (OAuth 2.0 authorization code grant — user clicks "Allow" once, tokens refresh automatically after) vs. machine-to-machine (OAuth 2.0 client credentials grant — agent authenticates as itself for shared/system resources).

## Key Concepts
- **AgentCore Policy**: Cedar-language rules that intercept every tool call *before* execution, enforced at the infrastructure level — "no refunds over $1,000" cannot be overridden by prompt engineering because it's not inside the reasoning loop.
- **Target**: a registered backend (API spec, Lambda, MCP server) on an AgentCore Gateway that becomes one or more auto-generated MCP tools.
- **microVM session isolation**: AgentCore's per-session sandboxing — unlike ECS's shared container (where another user's request could theoretically touch leftover state) or Lambda's stateless-but-shared execution environment.
- **RDS Proxy**: sits between agent and database, multiplexing hundreds of agent requests over a smaller connection pool — fixes concurrent-user connection-pool exhaustion without changing agent code.
- **Circuit breaker**: stops an agent from repeatedly retrying a clearly-down downstream service, avoiding wasted timeouts.
- **Dead letter queue (DLQ)**: catches messages that fail processing 3 times in an event-driven pipeline, triggering a CloudWatch alarm for investigation and later replay.
- **EventBridge Scheduler → ECS `RunTask`**: for scheduled batch work, spin up a task, run it, terminate — pay for minutes of Fargate instead of 24/7 uptime.

## Mental Models
- Diagnose deployment target the way the book frames it: a *document analysis agent* (short-lived, unpredictable traffic, no state) → Lambda; a *hospital scheduling agent* (persistent connections, multi-turn, always-on) → an always-on option (ECS or AgentCore).
- Think of AgentCore as "the parts of production hardening that have nothing to do with your agent's actual job, built for you" — auth, observability, tool integration, policy enforcement, per-user isolation are the same undifferentiated work you'd build identically for a REST API or batch processor; AgentCore packages it so you don't rebuild it per project.
- Use "the last two rows decide it" (from the comparison table): if you need full infrastructure control (VPC, security groups, GPU), pick ECS; if you'd rather have auth/observability/policy as managed services and accept less low-level control, pick AgentCore.

## Anti-patterns
- **Skipping the production-readiness checklist before deploying**: "This is a normal trajectory when you skip the production readiness conversation" — the book's own cautionary story (a document-analysis agent that worked on one document at a time, then choked, filled disk, and crashed silently under 200 concurrent users with zero alerting) is the direct consequence.
- **Initializing the agent/model object inside the Lambda handler**: pays the full setup cost (library load, Bedrock connection, prompt parsing) on every single invocation instead of once per warm environment.
- **Using Lambda for workloads with persistent connections or long multi-turn sessions**: fighting the platform — you'd be rebuilding warm-state semantics on top of a stateless primitive instead of using ECS/AgentCore where that's native.
- **Loading all tool definitions into the prompt at scale (200+ tools)**: consumes excessive context and degrades tool-selection accuracy — use Gateway's semantic search instead.
- **Storing secrets/API keys in environment variables or agent code**: "that's how breaches happen" — use Secrets Manager (self-managed) or AgentCore Identity's token vault (managed).
- **Reusing an inbound auth token as the outbound credential to downstream APIs**: breaks the security boundary auditors specifically check for; Gateway/Identity keep inbound and outbound tokens fully separate by design.

## Reference Tables

| | Lambda | ECS (Fargate) | AgentCore Runtime |
|---|---|---|---|
| Session model | Stateless, independent invocations | Shared container across requests | Dedicated microVM per session |
| State across turns | Persist externally (DynamoDB), reload every call | In-memory within the container | Maintained automatically in the microVM |
| Max execution | 15 min/invocation | No limit (container stays running) | 15 min sync, 60 min streaming, 8 hr async |
| Scaling unit | Per request | Per container (task count) | Per session |
| Pricing model | Per invocation + duration | Per allocated vCPU/memory (always on) | Active processing time only |
| Auth, observability, policy | You build it | You build it | Managed services you configure |
| Infrastructure control | Moderate (VPC, layers, concurrency) | Full (networking, security groups, instance types) | Abstracted — less control, less to manage |

**Industry deployment-fit examples from the chapter**:

| Industry / Use Case | Workload Shape | Deployment Fit |
|---|---|---|
| Fraud detection (financial services) | Independent per-transaction, no state, spiky | Lambda |
| Surgical scheduling (healthcare) | Multi-turn (~5 messages), per-coordinator sessions | AgentCore Runtime |
| Predictive maintenance (manufacturing) | Continuous monitoring, never stops, no user sessions | ECS |
| Customer support (e-commerce) | Multi-turn, per-customer isolation needed, shared tools/catalog | AgentCore Runtime |

## Worked Example
Deploying a document analysis agent to AgentCore Runtime end-to-end:
1. Write the agent exactly as in prior chapters (Strands `Agent` with `tools=[extract_clauses]`), then add 4 lines: `from bedrock_agentcore.runtime import BedrockAgentCoreApp`, `app = BedrockAgentCoreApp()`, decorate the handler with `@app.entrypoint`, and call `app.run()`.
2. Test locally: `python agent.py`, then `curl -X POST http://localhost:8080/invocations -d '{"prompt": "..."}'` — the SDK already exposes `/invocations` and `/ping` with no server code written.
3. Deploy via the Starter Toolkit:
   ```python
   from bedrock_agentcore_starter_toolkit import Runtime
   runtime = Runtime()
   runtime.configure(entrypoint="agent.py", requirements_file="requirements.txt",
       auto_create_execution_role=True, auto_create_ecr=True, region=region, agent_name="doc-analysis-agent")
   launch_result = runtime.launch()
   ```
   `configure()` generates the Dockerfile, IAM role, and ECR repo; `launch()` builds, pushes, and deploys.
4. Invoke via `runtime.invoke()` for testing, or `boto3`'s `invoke_agent_runtime` in production — each call runs inside an isolated per-session microVM, billed only for Active time.
The same 4-line wrapper pattern applies whether the underlying agent is Strands, LangGraph, CrewAI, or a full Claude Agent SDK setup with `CLAUDE.md`, `skills/`, and sub-agent orchestration — Runtime is framework-agnostic and manages only session/isolation/scaling around whatever is inside the microVM.

## Key Takeaways
1. Match deployment target to session shape: short/stateless/event-driven → Lambda; always-on/shared-state/multi-agent → ECS; per-user isolated/multi-turn/needs managed auth-policy-observability → AgentCore Runtime.
2. Always initialize the model/agent object outside the Lambda handler — this is the single highest-leverage performance decision for Lambda-hosted agents.
3. AgentCore's value is that auth, policy, observability, memory, and tool integration are managed services you compose (Identity, Gateway, Memory, Policy, Observability, Evaluations) rather than infrastructure you build per project — but you trade away low-level control (VPC placement, GPU access) for that.
4. AgentCore Gateway turns existing OpenAPI specs/Lambda functions/MCP servers into a unified MCP tool catalog with zero wrapper code — use semantic search (`x_amz_bedrock_agentcore_search`) once you're past roughly 100 tools.
5. AgentCore Identity separates inbound (who's calling) from outbound (what the agent authenticates as downstream) tokens by design — never conflate the two.
6. For database/API/event integrations outside Gateway's coverage: pool connections (RDS Proxy), retry with backoff and circuit breakers, decouple with EventBridge/SQS + dead-letter queues, and use `RunTask` instead of a Service for scheduled batch work.
7. Run through the production-readiness checklist (error handling, observability, graceful degradation, resource limits, peer-tested, rollback plan, cost estimate) before any real-user deployment — most production incidents trace back to skipping one of these.

## Connects To
- **Ch1**: AgentCore was introduced there as part of the AWS Agentic Stack's "Managed Solutions" tier; this chapter details its full composable architecture.
- **Ch2**: AgentCore Gateway auto-generates the same kind of `@tool`-style capabilities Ch2 taught you to hand-build.
- **Ch3**: AgentCore Memory (short-term/long-term) is the same service from Ch3, now wired directly into Runtime.
- **Ch4**: the ECS multi-agent example (six specialized agents in one container) directly reuses the graph pattern from Ch4.
- **Ch5**: AgentCore Gateway and Runtime both expose/consume MCP and A2A exactly as covered in Ch5, just at enterprise scale.
- **Ch7**: observability, guardrails, cost optimization, and IAM least-privilege — deployed-but-not-yet-production-ready gaps flagged at the end of this chapter — are closed next.
