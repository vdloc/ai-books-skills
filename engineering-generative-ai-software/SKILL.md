---
name: engineering-generative-ai-software
description: "Knowledge base from \"Engineering Generative AI-Based Software\" by Miroslaw Staron. Use when applying software-engineering practices to generative AI systems: architecting (monolith/MVC/microservice/RAG), requirements engineering for probabilistic software, deployment (cloud levels, ONNX, embedded), agentic AI, metamorphic testing, or API/ecosystem design for GenAI products."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Engineering Generative AI-Based Software
**Author**: Miroslaw Staron | **Pages**: ~208 | **Chapters**: 10 | **Generated**: 2026-09-15

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `RAG`, `metamorphic testing`, `API design`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch07`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

**The MLOps + Agile iterative lifecycle** (Ch3): generative AI software development is one loop — requirements → model training/testing → architecture → dual-track testing → deployment — not two separate ML and SE pipelines. Ship an imperfect MVP (Lean Startup) and iterate on both user-satisfaction and model-quality (benchmark) signals.

**Functional requirements are easy; acceptance criteria carry the weight** (Ch4): for GenAI software, write a one-line functional requirement, then attach explicit, checkable acceptance criteria describing probabilistic-content correctness (e.g., "all nouns from the prompt appear as image elements"). Non-functional requirements (latency, privacy, fault tolerance, interoperability) dominate the real engineering effort.

**Architecture selection = "where does the model live?"** (Ch5): same process → monolith (prototypes only, poor scale); same machine/separate process → MVC (best maintainability-to-complexity ratio, swap UI without touching model logic); remote machine → microservice (best scale/interoperability, worst latency/security surface); split device+cloud → embedded/edge (use only for latency- or privacy-sensitive sub-tasks). Use RAG (embed → vector search → LLM summarize) to ground answers and reduce hallucination — but always filter by relevance, since nearest-neighbor search always returns *something*.

**Metric selection by artifact type** (Ch3): BLEU (clipped) for natural language, CodeBLEU (AST-based) for source code, ROUGE for summarization. Using the wrong metric (e.g., BLEU on generated code) actively rewards degenerate copy-paste output.

**Metamorphic testing replaces oracle-based testing for generative correctness** (Ch6): when exact-match/BLEU testing would pass despite a wrong answer (e.g., swapping "Paris"↔"Stockholm" barely moves BLEU on a 100-word response), define a metamorphic relation instead — non-equivalence (perturb the meaningful input, expect output to change) or equivalence (perturb irrelevant input, expect output to stay the same).

**Prompting ladder** (Ch2): zero-shot → one-shot → few-shot → Chain-of-Thought, each trading prompt complexity for reliability without retraining. Use CoT specifically when the answer requires integrating multiple weak signals. Conversation history is cumulative and persists even after "forget the previous answer" — start a fresh context if a clean slate is needed.

**Agentic AI needs external grounding and bounded iteration** (Ch7): a bare `AgentAI` class (system role + message history + retry-safe HTTP call) hallucinates and never self-terminates. Multi-agent conversations converge in ~10-20 iterations or never — always cap iterations and define a stop condition. Mixing model families across roles (reasoning model as critic, plain LLM as generator) beats reusing one model twice. Tool-in-the-loop (compiler/validator + bounded retry) substitutes for using a bigger model.

**Model portability via ONNX** (Ch6, Ch8): export to ONNX to move a model across languages/platforms; quantize 32-bit→8-bit (~4x memory savings) or →4-bit (~8x) for roughly a 10% performance drop. ONNX Runtime deployment requires hand-rolling the autoregressive generation loop — no high-level `generate()` convenience.

**API design for GenAI services** (Ch9): converge on the OpenAI-compatible API shape; version every endpoint path (`/v1/`, `/v2/`) from day one — "public APIs are forever," and removing surface is nearly impossible. Provide heartbeat/capabilities/restart diagnostics. Use HTTP 200/400/500 families deliberately (e.g., 503 = model still loading). Never expose a GenAI endpoint without token auth + TLS — unauthenticated servers draw 100+ attacks/hour.

**Cloud deployment levels** (Ch8): app-level (finished product), capability-level (expose a trained model, OpenAI-API style), service-level (expose data, BigQuery style), infrastructure-level (expose compute). Pick based on what you're actually selling to customers.

**Hybrid architecture: classical software vs. AI in the driving seat** (Ch10): prefer "classical software in the driving seat" (deterministic code invokes AI for bounded, validated sub-tasks) when the workflow is enumerable — it's testable and controllable. "AI in the driving seat" (model decides which tools to invoke and when to stop) remains an open research problem around objective definition and stop conditions.

**Data quality for GenAI** (Ch4): ISO/IEC 25000 predates concept drift, fairness, and explainability — use a GenAI-aware data quality model (Foidl et al. influencing-factors model, or Bayram et al. DQSOps) instead when auditing training/RAG data.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-introduction.md) | Introduction — Generative AI and Software Engineering | Pre-training vs. fine-tuning, embeddings/latent space |
| [ch02](chapters/ch02-generative-ai-basics.md) | Generative AI Basics — How Instruct Models Work | Zero/one/few-shot, Chain-of-Thought, solve-by-delegation |
| [ch03](chapters/ch03-constructing-generative-ai-software.md) | Constructing Generative AI Software — Developing | MLOps+Agile lifecycle, Lean Startup iteration, dual-track testing, BLEU/CodeBLEU/ROUGE |
| [ch04](chapters/ch04-functional-nonfunctional-requirements.md) | Functional and Non-Functional Requirements | Requirement+acceptance-criteria pattern, data quality models, latency/quality/size triangle |
| [ch05](chapters/ch05-architecting-generative-ai-software.md) | Architecting Generative AI Software | Monolith, MVC, microservice, RAG, embedded/edge, architectural tactics |
| [ch06](chapters/ch06-implementation-quality-assurance.md) | Implementation and Quality Assurance | Framework layering, ONNX portability, DirectML, metamorphic testing |
| [ch07](chapters/ch07-handling-data-agentic-ai.md) | Handling Data for Generative AI Systems — Agentic AI | Agent class pattern, multi-agent conversation, ChromaDB RAG, tool-in-the-loop |
| [ch08](chapters/ch08-deployment.md) | Deployment of Generative AI Software | Cloud deployment levels, CI/CD, ONNX Runtime embedding, NanoLLM, model-size benchmarking |
| [ch09](chapters/ch09-generative-ai-ecosystems.md) | Generative AI Ecosystems | API design principles, versioning, status codes, auth/TLS, ecosystem layering, coopetition |
| [ch10](chapters/ch10-summary-current-trends.md) | Summary and Current Trends | Hybrid architectures, MoE/RLHF reasoning-model shift, EU AI Act, business-model shift |

## Topic Index

- **Acceptance criteria** → ch04
- **Agents / Agentic AI** → ch02, ch07
- **API design / versioning** → ch09
- **Architecture styles (monolith, MVC, microservice)** → ch05
- **Authentication / security (APIs)** → ch09
- **BLEU / CodeBLEU / ROUGE** → ch03, ch08
- **Chain-of-Thought prompting** → ch02
- **ChromaDB / vector databases** → ch05, ch07
- **CI/CD** → ch03, ch08
- **Cloud deployment levels** → ch08
- **Concept drift** → ch03, ch04
- **Data quality models** → ch04
- **DirectML / NPUs** → ch06
- **Ecosystems / coopetition** → ch09
- **Embeddings / latent space** → ch01, ch07
- **Embedded / edge models** → ch05, ch08
- **Fault tolerance requirements** → ch04
- **Few-shot / zero-shot / one-shot prompting** → ch02
- **Hybrid architectures (AI vs. classical software driving)** → ch10
- **Instruct models** → ch02
- **Metamorphic testing** → ch06
- **MLOps** → ch03
- **Model portability (ONNX)** → ch06, ch08
- **Multi-agent conversation** → ch07
- **Non-functional requirements** → ch04
- **Ollama / LangChain** → ch05
- **Pre-training / fine-tuning** → ch01
- **Quantization** → ch06, ch08
- **RAG (Retrieval-Augmented Generation)** → ch05, ch07
- **Regulation (EU AI Act)** → ch10
- **Requirements engineering** → ch04
- **Reasoning models / MoE / RLHF** → ch10
- **Roles (new/classical SE roles for GenAI)** → ch03
- **Tokenizers / BPE** → ch01

## Supporting Files

- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only (Chapters 1–10). Per project instruction, the Appendix (Additional Online Resources), References, and Index were excluded, and no glossary.md was generated. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills or ask the agent directly.
