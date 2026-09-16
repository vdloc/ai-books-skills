# Glossary

**Active learning** — iteratively improving a model by capturing user feedback, labeling failed/uncertain responses, and remediating via fine-tuning or retrieval fixes (Ch 8)

**Agent registry** — structured catalog of agents with metadata (name, task, I/O schema, engine, capabilities, endpoint) enabling dynamic discovery and orchestration (Ch 4)

**Agentic AI** — AI systems that act autonomously, make decisions, set goals, and dynamically interact with their environment, going beyond generative AI's content creation (Ch 4)

**Agentic workflow** — automation paradigm driven by AI agents capable of perceiving, reasoning, acting, and adapting (Ch 4)

**Bias detection & mitigation** — design pattern/practice identifying and reducing bias in training data and model outputs (Ch 3, Ch 7)

**Blue-Green Deployment** — deploying a new model version alongside the existing one for zero-downtime rollback (Ch 3)

**BLEU / ROUGE / METEOR** — traditional reference-based text-generation metrics measuring n-gram overlap, recall, and synonym/order-aware overlap respectively (Ch 6)

**Canary Deployment** — gradually releasing a new model to a subset of users before full rollout (Ch 3)

**Cache-Augmented Generation (CAG)** — using cached/precomputed context instead of a fresh retrieval call, as a lighter alternative to RAG (Ch 8)

**Chain of Responsibility Pattern** — passing requests along a chain of handlers, each deciding based on predefined criteria; used in agentic decision pipelines (Ch 3)

**Composite Pattern** — combining predictions from multiple models into one cohesive output (ensembling) (Ch 3)

**Cross-tier risk** — a risk spanning multiple risk tiers simultaneously, triggering cascading effects (e.g., hallucination → brand damage) (Ch 7)

**Data provenance / lineage** — tracing how data transforms and flows from source to consumption layer (Ch 5)

**Differential Privacy** — mathematically guaranteeing an individual's presence in a dataset can't be inferred from published statistics, via calibrated noise (Ch 3, Ch 7)

**Embedding dimensionality** — the vector size used to represent text/code semantically; lower dimensions (e.g., 256 vs. 3072) trade some semantic richness for speed/memory in code-heavy retrieval (Ch 5)

**Ethical AI** — aligning AI systems with moral values and societal norms (fairness, justice, human rights) (Ch 7)

**Federated Learning Pattern** — training a shared model across decentralized regions/devices without moving raw data; only model updates are shared (Ch 3, Ch 7)

**GOFAI (Good Old-Fashioned AI)** — classical, rule/logic-based symbolic AI requiring manually encoded knowledge, contrasted with connectionist/neural approaches (Ch 1)

**Hallucination** — model generates fluent, plausible, but factually incorrect or ungrounded output (Ch 1, Ch 6, Ch 7)

**HELM (Holistic Evaluation of Language Models)** — Stanford CRFM's multimetric, standardized, transparent benchmarking framework for foundation models (Ch 6)

**Hybrid search** — combining semantic (embedding) search with keyword (Lucene) search for both conceptual and exact-term retrieval precision (Ch 5)

**Inherent / Transient / Performance risk** — three risk-tiering categories: baked-in model/data risk, context-dependent situational risk, and inference-time accuracy/reliability risk (Ch 7)

**LLMOps** — specialized extension of MLOps addressing the operational demands unique to large language models (scale, context-sensitivity, prompt management) (Ch 6)

**Long-term memory** — agent memory stored outside the context window (vector DBs, knowledge bases) retrieved on demand (Ch 4)

**Medallion architecture** — layered data pipeline pattern (raw/bronze → masked → segmented → analytical/silver → gold) ensuring reprocessability and progressive enrichment (Ch 5)

**Microservice Architecture Pattern** — decomposing an AI pipeline into independently deployable/scalable services (Ch 3, Ch 4)

**MLOps** — disciplined practices automating and governing the ML system life cycle from development through deployment and maintenance (Ch 6)

**Model Compression Pattern** — pruning/quantization/knowledge distillation to shrink model size without significant accuracy loss (Ch 3)

**Model Context Protocol (MCP)** — standardized client-server protocol for agents/LLMs to discover and consume external resources/tools instead of ad hoc integrations (Ch 4)

**Model drift** — decrease in model effectiveness over time as underlying data/concepts change (Ch 3, Ch 6)

**MoSCoW** — requirement prioritization framework: Must have / Should have / Could have / Won't have (Ch 5)

**Multi-agent system** — architecture where multiple specialized agents collaborate, communicate, and coordinate toward shared goals (Ch 4)

**NIST AI Risk Management Framework (AI RMF)** — voluntary US framework organizing AI risk management into Govern, Map, Measure, Manage functions (Ch 7)

**Observer Pattern** — publisher-subscriber pattern letting an agent monitor environment state changes and react accordingly (Ch 3, Ch 4)

**Orchestrator** — LLM-driven "brain" of an agentic system that plans, decides which tools/agents to invoke, and consolidates results (Ch 4)

**pass@k** — code-generation metric checking whether any of k generated samples passes a suite of unit tests (Ch 6)

**Pipeline Pattern** — structuring data ingestion into a sequential, reliable workflow (Ch 3)

**Prompt compression** — rewriting verbose prompts into shorter, semantically equivalent versions to cut token cost/latency (Ch 5)

**Prompt tuning** — iteratively refining LLM instructions (not model weights) to optimize task-specific output; contrasted with fine-tuning (Ch 5)

**RACI matrix** — Responsible/Accountable/Consulted/Informed task-ownership assignment tool (Ch 5)

**RAG (Retrieval-Augmented Generation)** — grounding LLM generation in retrieved context from an indexed knowledge base (Ch 4, Ch 5)

**Requirement Traceability Matrix (RTM)** — maps requirement ID → business objective → technical component → test case → status (Ch 5)

**Responsible AI (RAI)** — comprehensive approach to AI development emphasizing ethics, transparency, accountability, fairness, safety, privacy, inclusiveness, and sustainability (Ch 7)

**Risk tiering** — classifying AI risks into inherent/transient/performance tiers with weighted scoring to prioritize governance investment (Ch 7)

**Self-reflection pattern** — an agent critiques its own output against a checklist/threshold and iteratively regenerates until quality is met (Ch 4)

**Shadow Testing** — running a new model silently alongside production without exposing output to users, for safe validation (Ch 3)

**State Pattern** — enabling an object/agent to alter behavior when its internal state changes (Ch 3)

**Strategy Pattern** — defining interchangeable algorithms/prompt strategies selectable dynamically based on context (Ch 3)

**Transformer architecture** — 2017 Google architecture using multi-head attention to process tokens in parallel, foundational to modern LLMs (Ch 1)

**Trustworthy AI** — building confidence in AI systems via transparency, reliability, safety, and user trust (Ch 7)

**Turing test** — operational definition of machine intelligence based on a human judge's inability to distinguish machine from human text responses (Ch 1)

**Working memory** — an agent's short-term, in-context information retained during one reasoning loop, bounded by the LLM's context window (Ch 4)

**Zero Trust Pattern** — default-deny access control granting permissions strictly on a need basis (Ch 3, Ch 5)
