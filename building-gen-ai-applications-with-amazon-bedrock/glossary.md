**Action Group** — a Bedrock Agent component defining a tool's schema, backed by a Lambda function that executes the operation (Ch8)

**AgentCore / AgentCore Gateway** — Bedrock's managed secure/scalable/observable execution environment connecting an agent's LLM reasoning to enterprise systems (Ch8)

**Agentic AI** — the architectural pattern fusing LLM reasoning with secure cloud execution infrastructure so a model can autonomously take real-world actions, not just generate text (Ch1, Ch8)

**Batch Inference** — asynchronous, job-based Bedrock invocation (`CreateModelInvocationJob`) for large non-interactive workloads, cheaper at scale than real-time calls (Ch5, Ch6)

**Bedrock Agents** — Amazon's managed framework for building agents: model + instructions + Action Groups + optional Knowledge Base (Ch8)

**Bedrock Guardrails** — configurable content-filtering/safety layer applied consistently across models to block harmful content, PII, and off-topic responses (Ch1)

**Chain-of-Thought Prompting** — a prompt pattern instructing the model to reason step-by-step before producing a final answer, improving multi-step reasoning accuracy (Ch4)

**Chunking** — splitting large documents into smaller segments before embedding in a RAG pipeline, to fit context limits and improve retrieval precision (Ch4)

**Circuit Breaker Pattern** — a 3-state (Closed/Open/Half-Open) resilience pattern preventing an application from repeatedly calling a persistently failing service (Ch6)

**Context Window (Context Length)** — the maximum number of tokens a model can attend to in one call, bounding RAG context and conversation history (Ch3)

**Control Plane vs. Data Plane** — `bedrock` client (model discovery/management) vs. `bedrock-runtime` client (actual inference calls) (Ch2, Ch5)

**Converse / ConverseStream API** — Bedrock's unified conversation API normalizing request/response format across model providers (Ch5)

**Cross-Region Inference (Inference Profiles)** — routing inference across AWS regions for throughput, availability, and disaster-recovery resilience (Ch6)

**DPO (Direct Preference Optimization)** — an RLHF alternative that directly optimizes the LLM policy on preference pairs without training a separate reward model (Ch3)

**Embeddings** — vector representations of text (e.g. via Amazon Titan Embeddings) used for semantic search/retrieval in RAG (Ch4)

**Experience Engineering** — the book's framework for engineering an AI's persona, brand voice, and interaction "feel" beyond functional prompt correctness (Ch8)

**Few-Shot Prompting** — a prompt pattern providing several input-output example pairs before the real input, to guide output format/style (Ch4)

**Fine-Tuning (Full / PEFT)** — adapting a model's parameters to a task or domain; Full Fine-Tuning adjusts all parameters (highest cost), PEFT (e.g. LoRA/QLoRA) adjusts only a small injected subset (Ch3, Ch4)

**Foundation Model (FM)** — a large-scale model pretrained on massive datasets, adaptable to many downstream tasks via prompting or fine-tuning (Ch1, Ch3)

**Greedy Decoding** — a sampling strategy always selecting the single most probable next token; deterministic, used for structured/constrained output (Ch3)

**GQA / MQA (Grouped/Multi-Query Attention)** — attention variants reducing Key/Value head count to cut inference latency/cost at a small quality cost (Ch3)

**IAM (Identity and Access Management)** — AWS's access-control service governing Bedrock permissions via policies attached to users/roles (Ch2)

**Knowledge Base (Bedrock)** — Bedrock's managed RAG component connecting to data sources (S3, Confluence, SharePoint, etc.) for retrieval (Ch4, Ch8)

**KTO (Kahneman-Tversky Optimization)** — an RLHF alternative using simple binary good/bad preference signals, robust to noisy labels (Ch3)

**LoRA / QLoRA** — Parameter-Efficient Fine-Tuning techniques injecting small low-rank adapter matrices (QLoRA additionally quantizes the base model) instead of updating all parameters (Ch3)

**MCP (Model Context Protocol)** — an open standard (Anthropic, late 2024) providing a universal interface for AI models to access external tools/data/APIs — the "USB-C port for AI" (Ch8)

**Model Fallback** — a resilience pattern defining an ordered chain of alternative models to use when the preferred model is unavailable or the circuit is open (Ch6)

**ORPO (Odds Ratio Preference Optimization)** — an RLHF alternative combining instruction-tuning and preference alignment in a single training step (Ch3)

**Prompt Caching (Bedrock)** — caching repeated prompt prefixes to reduce latency/token cost on subsequent calls sharing that prefix (Ch6)

**Provisioned Throughput** — a Bedrock capacity model reserving dedicated model throughput for predictable high-volume workloads (or required for custom/fine-tuned models) (Ch1, Ch4, Ch6)

**RAG (Retrieval-Augmented Generation)** — grounding generation by retrieving relevant external documents at query time and injecting them into the model's context (Ch1, Ch4)

**ReAct Loop (Reasoning and Acting)** — the iterative Plan → Action → Observation cycle underlying Bedrock Agent autonomy (Ch8)

**RLHF (Reinforcement Learning from Human Feedback)** — a 3-stage alignment process (collect rankings → train reward model → PPO fine-tune) used to align models with human preferences (Ch3)

**Self-Attention** — the Transformer mechanism letting each token weigh every other token when building its representation (Ch3)

**Semantic Caching** — application-level caching keyed on semantic similarity of queries, avoiding redundant Bedrock calls for near-duplicate requests (Ch6)

**Semantic Versioning for Prompts** — applying MAJOR.MINOR.PATCH discipline to production prompt templates for traceability and reproducibility (Ch7)

**Top-k Sampling** — a decoding strategy limiting token choices to the k most probable, improving coherence over pure temperature sampling (Ch3)

**Top-p (Nucleus) Sampling** — sampling from the smallest token set whose cumulative probability ≥ p; the book's default recommendation for high-quality open-ended generation (Ch3)

**Vibe Coding** — using tuned system prompts + inference parameters to drive rapid product design/prototyping with a foundation model as co-design partner (Ch8)

**Zero-Shot Prompting** — a prompt pattern relying on instruction alone (no examples), testing the model's pretrained generalization (Ch4)
