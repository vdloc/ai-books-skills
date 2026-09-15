# Patterns & Techniques — Engineering Generative AI-Based Software

## Pre-training + Task-Specific Fine-Tuning
**When to use**: building any generative model from scratch, or understanding what a foundation model already gives you.
**How**: pre-train (or reuse a pre-trained) transformer via self-supervised masked-language-modeling on a large domain corpus; then fine-tune a smaller task-specific head (or the whole model) on labeled (instruction, output) pairs for the specific job.
**Trade-offs**: pre-training is expensive and data-hungry but reusable across tasks; fine-tuning is cheap but only works well when the pre-training domain matches the deployment domain (mismatch → high OOV rate, poor embeddings). (Ch1)

## Zero/One/Few-Shot & Chain-of-Thought Prompting
**When to use**: specializing model behavior at inference time without retraining.
**How**: zero-shot = plain instruction; one-shot = one example in-context; few-shot = several positive/negative examples; CoT = explain the reasoning pattern before the examples and ask the model to reason step by step before answering.
**Trade-offs**: cheap and instant, but session-scoped (must resupply every call) and less reliable than fine-tuning for hard tasks needing many weak-signal cues — CoT specifically helps when reasoning must integrate multiple cues. (Ch2)

## Solve-by-Delegation (proto-RAG)
**When to use**: tasks needing exactness that pure text generation can't reliably deliver (e.g., classification, arithmetic).
**How**: prompt the model to generate a program/artifact that solves the problem, execute that artifact deterministically, then have the model interpret/explain the result.
**Trade-offs**: much more exact than direct generation; adds an execution step and requires a safe sandbox for running generated code. (Ch2, generalizes to Ch5 RAG and Ch7 agents)

## MLOps + Agile Iterative Lifecycle
**When to use**: any generative AI product development, from day one.
**How**: run requirements → model training/testing → architecture → dual-track testing → deployment as one iterative loop; ship an imperfect MVP (Lean Startup) and iterate using both user-satisfaction and model-quality (benchmark) signals.
**Trade-offs**: avoids the "perfect model" trap and the "ML/SE silo" trap, but requires cross-disciplinary roles (ML engineer, data scientist, infra manager) alongside classical SE roles. (Ch3)

## Dual-Track Testing (Oracle + Benchmark)
**When to use**: testing any software that includes a probabilistic generative component.
**How**: run classical oracle-based tests (unit/integration/system, mocked responses) in parallel with model-benchmark tests (BLEU/CodeBLEU/ROUGE on a held-out dataset).
**Trade-offs**: doubles test-authoring effort but is the only way to catch both software regressions and model-quality regressions. (Ch3, extended by Ch6 metamorphic testing)

## Metric Selection by Artifact Type
**When to use**: choosing an automated evaluation metric for generated output.
**How**: BLEU for natural-language similarity; CodeBLEU (AST-based) for source code; ROUGE for summarization; always clip n-gram counts to avoid trivial repetition gaming.
**Trade-offs**: using the wrong metric (e.g., BLEU on code) rewards degenerate copy-paste behavior. (Ch3)

## Requirement-with-Acceptance-Criteria for GenAI
**When to use**: writing functional requirements for any generative feature.
**How**: write a one-line functional requirement, then attach explicit, checkable acceptance criteria describing probabilistic-content correctness (e.g., "all nouns from the prompt appear as image elements; no visible merge artifacts").
**Trade-offs**: takes more upfront specification effort than deterministic-software requirements, but is the only way to make GenAI output testable. (Ch4)

## Fault-Tolerance Triplet (condition, detection threshold, response)
**When to use**: specifying any fallback behavior for GenAI infrastructure failure.
**How**: define (1) the failure condition, (2) a numeric detection threshold (e.g., "no response within 30 seconds = connection lost"), (3) the response (fallback to smaller local model, or notify + retry).
**Trade-offs**: more requirements to write, but prevents ambiguous "handle errors gracefully" specs that can't be tested. (Ch4)

## Architecture Selection by "Where Does the Model Live"
**When to use**: choosing between monolithic, MVC, microservice, or embedded/edge architecture.
**How**: ask where the model process lives — same process (monolith), same machine separate process (MVC/local service), remote machine (microservice), or split device+cloud (embedded/edge) — then check Table 5.1's scalability/maintainability/performance/latency/security profile for that choice.
**Trade-offs**: monolith is simplest but doesn't scale; MVC gets most maintainability benefit without network cost; microservices scale/interoperate best but add latency and attack surface; embedded/edge splits trade quality for latency/privacy on a subset of tasks. (Ch5)

## RAG (Retrieval-Augmented Generation)
**When to use**: reducing hallucination by grounding generation in retrieved documents.
**How**: embed the user query → nearest-neighbor search a vector database (ChromaDB, etc.) → LLM summarizes the retrieved documents into the final answer; optionally fall back to live web search when the database has no good match, or chain in a reasoning/tool step.
**Trade-offs**: quality is bounded by what's actually in the vector store; embedding search always returns *something*, so pair with a relevance/where-clause filter. (Ch5, Ch7)

## Metamorphic Testing
**When to use**: testing generative model correctness where exact-match/BLEU testing would pass despite wrong answers.
**How**: define a metamorphic relation — non-equivalence (perturb the meaningful part of input, e.g., country name, and assert output changes accordingly) or equivalence (perturb an irrelevant part, e.g., swap 2 characters, and assert output stays semantically the same).
**Trade-offs**: requires designing relations per task, but is the only reliable way to catch factually-wrong-but-textually-similar outputs. (Ch6)

## Model Portability via ONNX
**When to use**: deploying a model trained in one language/platform (typically Python) into another (C#, C++, embedded).
**How**: export the model to ONNX intermediate format; optionally quantize (32-bit → 8-bit/4-bit, ~4-8x memory reduction for ~10% performance loss); load via ONNX Runtime, DirectML, or an NPU-specific backend on the target platform.
**Trade-offs**: enables cross-platform, IP-protecting (compiled) deployment; but ONNX Runtime deployments require hand-rolling the autoregressive generation loop (no high-level `generate()` convenience). (Ch6, Ch8)

## Agent Class (stateful role-wrapper)
**When to use**: giving a model a fixed role/persona and clean multi-turn conversation state.
**How**: encapsulate `server_address`, `model_name`, a system-role string, and a `messages` list behind a class with a `get_response(prompt)` method that appends turns and calls the model API (with retry/timeout handling).
**Trade-offs**: simple and reusable, but the conversation history grows unbounded — plan for context-length limits in long-running agents. (Ch7)

## Multi-Agent Conversation with Bounded Iteration
**When to use**: having two or more agents collaborate on a task (e.g., programmer + designer).
**How**: feed each agent's output as the next agent's input, in a loop, with a hard iteration cap and an explicit success/stop condition (e.g., "compiles successfully").
**Trade-offs**: agents converge within ~10-20 iterations or never — do not rely on them to self-terminate; mixing different model families across roles outperforms one model in two roles. (Ch7)

## Tool-in-the-Loop Agent (compiler/validator feedback)
**When to use**: any code-generation (or other checkable-output) agent.
**How**: extract the generated artifact, run it through a deterministic validator (compiler, linter, test suite), and if it fails, feed the error back to the model for a bounded number of retries.
**Trade-offs**: substantially improves output quality even with small/non-reasoning models — can substitute for using a larger model. (Ch7)

## Cloud Deployment Level Selection
**When to use**: deciding what to expose to customers — a full product, model access, data access, or raw compute.
**How**: pick app-level (web app/container) for end-user products, capability-level (REST-exposed model) for developer-facing model access, service-level (exposed datasets) for data-collection businesses, or infrastructure-level (pre-configured compute) for power users with their own stack.
**Trade-offs**: higher levels give customers more convenience but you more operational responsibility; lower levels give customers more flexibility but you less control over how your asset is used. (Ch8)

## Versioned, Authenticated, Status-Coded API
**When to use**: exposing any generative AI capability as a service.
**How**: path-version every endpoint (`/v1/`, `/v2/`); provide heartbeat/capabilities/restart diagnostic endpoints; use HTTP 200/400/500 families deliberately (e.g., 503 for "model still loading"); gate with a token header (`API-Key`) checked via `abort(401)`; serve over TLS in production.
**Trade-offs**: more upfront design work, but avoids the costly retrofits the book documents (e.g., OpenAI's own pre-`/v1/` pain) and closes the "100+ attacks/hour on an unauthenticated endpoint" risk. (Ch9)

## Hybrid Architecture: Classical Software vs. AI in the Driving Seat
**When to use**: deciding how autonomous an agentic feature should be.
**How**: choose "classical software in the driving seat" (deterministic code invokes AI for bounded sub-tasks, then validates the result) when the workflow is enumerable; choose "AI in the driving seat" (the model decides which tools/functions to invoke) only when the sequence of steps is genuinely unknown ahead of time.
**Trade-offs**: classical-software-driven hybrids are far more testable/controllable today; AI-driven autonomy remains an open research problem around stop-conditions and objective definition. (Ch10)
