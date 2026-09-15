# Chapter 6: Scaling Generative AI Applications

## Core Idea
Scaling Bedrock applications requires resilience patterns beyond simple retries — circuit breakers, model fallback, cross-region inference, and prompt caching — combined with active cost/observability management (CloudWatch dashboards, AWS Budgets, model-level cost tracking), because at scale both failure modes and cost overruns compound quickly.

## Frameworks Introduced
- **Circuit Breaker Pattern**: prevents cascading failures when Bedrock (or a downstream dependency) is persistently unavailable, as opposed to just transiently throttled.
  - When to use: any production Bedrock integration, layered on top of (not instead of) the exponential-backoff retry logic from Ch5 — retries handle transient issues, circuit breakers handle persistent/systemic ones.
  - How: implement as a 3-state machine — **Closed** (normal, requests pass through, failures counted; trips to Open once failures exceed a threshold within a time window) → **Open** (all calls immediately rejected with an error/fallback, no wasted network calls, until a timeout elapses) → **Half-Open** (after the timeout, a limited number of trial requests are allowed through; success returns to Closed, failure returns to Open). This prevents an application from hammering an already-failing Bedrock endpoint and gives it time to recover.
- **Balancing Performance and Price in Model Selection**: an explicit cost/capability matrix across Bedrock's model catalog.
  - When to use: choosing a default production model, or deciding when to fall back to a cheaper model under load/budget pressure.
  - How: compare candidate models (e.g. Amazon Titan Text Express, Anthropic Claude 3.5 Sonnet, Meta Llama 3 70B Instruct) across Model Type, Max Context Window, input/output token pricing, and observed latency — cheaper/smaller models for high-volume, latency-tolerant or simpler tasks; premium models reserved for complex reasoning where quality directly drives business value.
- **Model Fallback**: a resilience pattern distinct from circuit breaking.
  - When to use: when a preferred model is unavailable, rate-limited, or the circuit is Open.
  - How: define an ordered fallback chain (e.g. Claude 3.5 Sonnet → a cheaper/faster model) so the application degrades gracefully (lower quality but still functional) rather than failing outright.

## Key Concepts
- **Cross-Region Inference (Bedrock Inference Profiles)**: routing inference requests across multiple AWS regions to increase effective throughput/availability and provide disaster-recovery resilience against a regional outage, using an Inference Profile ID rather than a single-region model ID.
- **Bedrock Prompt Caching**: caching repeated prompt prefixes (e.g. a long system prompt or few-shot examples reused across many calls) to reduce latency and token cost on subsequent invocations that share the cached prefix.
- **Application-Level Semantic Caching**: caching at the application layer based on semantic similarity of queries (not just exact string match), avoiding redundant Bedrock calls for near-duplicate requests.
- **Horizontal Scaling — Lambda vs. ECS**: Lambda suits bursty, event-driven, short-duration invocation patterns (matches Ch5's event-driven triangle); Amazon ECS with Fargate suits sustained, heavy, long-running workloads where Lambda's execution-time/memory limits or cold-start costs become a bottleneck.
- **Model-Level and Pipeline-Level Observability**: tracking metrics (InvocationLatency, InputTokenCount, OutputTokenCount, Invocations, Errors) per-model and across the full pipeline via CloudWatch dashboards, plus third-party LLM observability tools (Datadog LLM Observability, Prometheus integration) for deeper tracing.
- **AWS Budgets and Cost Explorer for Bedrock**: proactive cost governance — AWS Budgets for threshold alerts, Cost Explorer for retrospective analysis broken down by model/service, both explicitly recommended over reactive bill-shock discovery.

## Mental Models
- Layer resilience patterns rather than choosing one: **retries with backoff (Ch5) for transient errors → circuit breaker for persistent failures → model fallback for graceful degradation → cross-region inference for regional-outage-level resilience** — each addresses a different failure severity/duration.
- Treat **model selection as dynamic, not static**: the same application may route to different models based on load, cost budget remaining, or circuit-breaker state — not a single hardcoded model ID.
- Use the **Lambda vs. ECS decision as a workload-shape question**: "does this need to scale to zero and handle unpredictable bursts" (Lambda) vs "does this run continuously and need more control over resource allocation" (ECS/Fargate) — not a general-purpose preference for one over the other.

## Anti-patterns
- **Relying solely on retries for a fully down Bedrock endpoint**: wastes resources and can slow the failing service's own recovery — this is exactly the scenario the Circuit Breaker pattern is designed to short-circuit (pun intended by the book).
- **Hardcoding a single model ID with no fallback path**: a rate limit or regional issue on that specific model takes down the whole application; define a fallback chain instead.
- **Treating cost monitoring as a monthly bill review**: the book positions AWS Budgets (proactive alerting) and Cost Explorer (analysis) as required from day one of scaling, not a retrospective exercise after an unexpected bill.
- **Defaulting to the largest/most capable model everywhere at scale**: directly costs more per request; combine with Ch1's model-bucket mental model — route simple/high-volume requests to cheaper models under the Balancing Performance and Price framework.

## Reference Tables
**Circuit Breaker States**
| State | Behavior | Transition |
|---|---|---|
| Closed | Requests pass through normally; failures counted | → Open when failure count exceeds threshold within time window |
| Open | All calls immediately rejected (fast fail / fallback) | → Half-Open after configured timeout |
| Half-Open | Limited trial requests allowed through | → Closed on success; → Open on failure |

**Scaling Compute Choice**
| Dimension | AWS Lambda | Amazon ECS with Fargate |
|---|---|---|
| Best for | Bursty, event-driven, short invocations | Sustained, heavy, long-running workloads |
| Scaling | Automatic, scale-to-zero | Manual/auto-scaling group, always-on baseline |
| Cost model | Pay per invocation/duration | Pay for provisioned/running capacity |

## Worked Example
The book walks through a serverless Circuit Breaker implementation for Bedrock: a `CircuitBreakerState` item (Closed/Open/Half-Open) stored in a fast key-value store, with `IncrementFailureCount`/`GetCircuitState`/`IsCircuitOpen` operations wrapping every Bedrock `InvokeModel` call — before each call, `IsCircuitOpen` is checked (fail fast if Open); after each call, success resets the failure count while failure increments it and trips the circuit to Open once `CheckFailureThreshold` is exceeded, with an `OpenUntil` timestamp governing the transition to Half-Open.

## Key Takeaways
1. Layer retries (transient) → circuit breaker (persistent failures) → model fallback (graceful degradation) → cross-region inference (regional outage) — each targets a different failure class.
2. Route requests to models dynamically using the performance/price matrix rather than hardcoding one model for the whole application.
3. Use prompt caching (Bedrock-native) and application-level semantic caching together to cut both cost and latency on repeated/similar requests.
4. Choose Lambda for bursty event-driven scaling, ECS/Fargate for sustained heavy workloads — match compute to workload shape, not habit.
5. Set up AWS Budgets and Cost Explorer for Bedrock proactively; cost governance is a day-one scaling concern, not a retrospective fix.
6. Track model-level metrics (latency, token counts, error rate) via CloudWatch, supplemented by LLM-specific observability tooling (Datadog, Prometheus) for tracing across a full RAG/agent pipeline.

## Connects To
- **Ch5**: extends the exponential-backoff retry logic with circuit breaking and fallback for more severe/persistent failure modes.
- **Ch1/Ch3**: the model selection matrix here is the cost/performance-focused counterpart to Ch1's capability-focused selection table.
- **Ch7**: scaling patterns here underpin the multi-model, multi-region architectures used in the advanced/industry use cases.
