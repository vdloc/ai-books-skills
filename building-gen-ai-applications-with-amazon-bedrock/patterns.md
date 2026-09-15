## Prompt Engineering → RAG → Fine-Tuning Escalation
**When to use**: any new AI-engineering requirement, before committing engineering effort.
**How**: try prompt design (zero-shot/few-shot/chain-of-thought) first — cheapest, fastest iteration. If the gap is missing/outdated knowledge, add RAG. Only fine-tune (PEFT before full fine-tuning) if the gap is a style/format/skill neither prompting nor retrieval can fix.
**Trade-offs**: skipping straight to fine-tuning wastes data/compute/maintenance cost on a problem prompting or retrieval would have solved for free.

## RAG Pipeline (5-Stage)
**When to use**: grounding generation in proprietary/current data the base model wasn't trained on.
**How**: (1) Document Ingestion & Preprocessing (load, clean, chunk), (2) Embedding (Titan Embeddings), (3) Vector Storage/Indexing (OpenSearch Service or Aurora+pgvector), (4) Retrieval (semantic search top-k), (5) Augmented Generation (inject retrieved chunks into prompt).
**Trade-offs**: quality is usually bottlenecked by chunking/retrieval, not generation — debug from ingestion forward, not by tweaking the final prompt.

## Chunking Strategy Selection
**When to use**: designing a RAG pipeline's ingestion stage.
**How**: fixed-size (simplest) → recursive (semantic separators) → structural (document structure) → semantic (embedding-based breakpoints) → agentic (LLM-determined boundaries). Choose based on document nature and retrieval precision needs.
**Trade-offs**: chunks too large dilute relevance with noise; too small lose necessary context. More sophisticated strategies cost more (compute/latency) but improve retrieval precision.

## Sampling Strategy Selection
**When to use**: configuring `temperature`/`top_k`/`top_p` for any `invoke_model` call.
**How**: greedy decoding for deterministic/structured tasks; temperature sampling for global creativity tuning; top-k for a fixed randomness ceiling; top-p (nucleus) as the default for high-quality open-ended generation.
**Trade-offs**: mismatching strategy to task produces either robotic repetitive output or unreliable structured output.

## Adaptation Method Ladder
**When to use**: customizing a model for a domain/task.
**How**: zero-shot → few-shot → PEFT (LoRA/QLoRA) → full fine-tuning, in increasing cost/data/compute order.
**Trade-offs**: full fine-tuning gives maximum task specificity but is the most expensive and requires ongoing retraining maintenance; escalate only when cheaper rungs plateau.

## Exponential Backoff with Jitter
**When to use**: handling `ThrottlingException` and other transient Bedrock errors.
**How**: configure the Boto3 client's built-in retry handler (`botocore.config.Config`, `standard`/`adaptive` mode) — wait time doubles each retry with added random jitter.
**Trade-offs**: manual `time.sleep()` retry loops synchronize retries across clients into a "thundering herd" that worsens throttling — always prefer the built-in handler.

## Circuit Breaker for Persistent Failures
**When to use**: layered on top of retries, for a Bedrock dependency that's down (not just rate-limited).
**How**: 3-state machine — Closed (normal, count failures) → Open (fail fast, no calls) after threshold exceeded → Half-Open (trial requests) after a timeout, returning to Closed on success or Open on failure.
**Trade-offs**: adds implementation complexity but prevents wasted calls/bandwidth and gives a failing service room to recover.

## Model Fallback Chain
**When to use**: when the preferred model is unavailable, throttled, or its circuit is Open.
**How**: define an ordered list of alternative models (e.g. premium → cheaper/faster) so the application degrades gracefully instead of failing.
**Trade-offs**: fallback models may produce lower-quality output — communicate degraded mode to users/logs where it matters.

## Batch vs. Real-Time Invocation
**When to use**: choosing `InvokeModel`/`Converse` vs. `InvokeModelWithResponseStream` vs. Batch Inference (`CreateModelInvocationJob`).
**How**: real-time for low-latency interactive needs; streaming when tokens should render as they arrive; batch for large non-interactive volumes where cost efficiency beats per-request latency.
**Trade-offs**: batch is cheaper at scale but has no interactivity; real-time/streaming cost more per unit throughput.

## Least-Privilege IAM Scoping (Progressive)
**When to use**: moving an application from learning/prototype to production.
**How**: start broad (`AmazonBedrockFullAccess`) to unblock initial learning (Ch2), then narrow to specific action prefixes (`bedrock:InvokeModel`, `bedrock:CreateGuardrail`, etc.) once the application's real permission surface is known (Ch5).
**Trade-offs**: broad policies speed up initial development but are a security liability if carried into production unchanged.

## Decouple Prompt Logic from Pipeline Control
**When to use**: any application beyond a throwaway prototype.
**How**: store prompt templates externally (S3, Parameter Store, or a prompt registry) rather than hardcoding them in Lambda/application code.
**Trade-offs**: adds a small amount of infrastructure, but decouples prompt iteration from full deployment cycles.

## Semantic Versioning for Prompts
**When to use**: any production prompt that will be iterated on after initial deployment.
**How**: MAJOR bump for behavior/output-contract changes, MINOR for backward-compatible enhancements, PATCH for wording/formatting fixes. Track `PromptTemplateID` + `PromptTemplateVersion`.
**Trade-offs**: requires discipline/tooling overhead, but is what makes past outputs reproducible/debuggable.

## Audit Log Schema for LLM Invocations
**When to use**: any regulated or business-critical Bedrock use case.
**How**: log InvocationID, TimestampUTC, UserID/SystemID, ModelID, PromptTemplateID/Version, InputVariables, FinalPrompt, InferenceParameters, RawModelResponse, GuardrailEvaluation, InvocationLatency, TraceID per call.
**Trade-offs**: logging overhead and storage cost, but is required for compliance review and reproducibility (e.g. explaining a fraud-flag decision).

## ReAct Loop (Plan → Action → Observation)
**When to use**: designing or debugging any Bedrock Agent.
**How**: the agent iteratively reasons about the next step (Plan), invokes a tool/Action Group/MCP tool (Action), receives the result (Observation), and repeats until the goal is met.
**Trade-offs**: gives autonomy and multi-step task execution, but requires a secure/observable execution environment (AgentCore) — reasoning capability alone is not sufficient.

## Experience Engineering for Branded Agents
**When to use**: customer-facing or brand-representing AI agents/copilots.
**How**: engineer a persistent persona via system prompts, craft brand-specific tone/vocabulary, use fine-tuning for brand voice where prompting can't consistently capture nuance, and tune temperature per interaction type (low for factual/validating, higher for creative/brand-voice responses).
**Trade-offs**: adds design/engineering effort beyond functional correctness, but is what produces consistent brand alignment across interactions.
