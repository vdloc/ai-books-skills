---
name: hands-on-rag-for-production
description: "Knowledge base from \"Hands-On RAG for Production: Design, Develop, and Deploy Production-Ready RAG Applications\" by Ofer Mendelevitch & Forrest Sheng Bao. Use when applying RAG architecture patterns (retrieval, chunking, embeddings, vector search), hybrid search and reranking, RAG evaluation and hallucination detection, agentic RAG, multimodal RAG (tables/images/audio/video), knowledge graphs, deploying RAG to production, or studying the book's concepts."
---

# Hands-On RAG for Production: Design, Develop, and Deploy Production-Ready RAG Applications
**Authors**: Ofer Mendelevitch & Forrest Sheng Bao | **Pages**: ~330 | **Chapters**: 10 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference across any RAG task.
- **With a topic** — ask about `hybrid search`, `reranking`, `hallucination detection`, `agentic RAG`, `knowledge graphs`, `multimodal RAG`, or another indexed topic; the relevant chapter is read on demand.
- **With a chapter** — ask for `ch06` (or by name) to load that specific chapter in full, including code examples and worked examples.
- **Browse** — ask "what chapters do you have?" to see the full index below.

When a question touches a topic not covered in Core Frameworks, read the relevant chapter file before answering — chapters carry the code examples, worked examples, and reference tables that make answers concrete rather than generic.

---

## Core Frameworks & Mental Models

**The R-G split**: every RAG query has two steps — Retrieval (find relevant facts) then Generation (synthesize an answer grounded only in those facts). Production quality depends on optimizing both independently: diagnose failures in order — retrieval failure (low recall/precision) → generation failure (hallucination, context utilization, answer relevance) → ingestion failure (parsing errors, staleness). No amount of prompt engineering fixes a retriever with 40% recall.

**Two-Stage Retrieval Pipeline**: candidate generation (fast, recall-optimized — vector/lexical/hybrid search) followed by reranking (slow, precision-optimized — cross-encoder or business logic on the smaller candidate set). This is the standard production retrieval architecture beyond a basic POC. Bounded by stage-1 recall.

**RAG vs. fine-tuning vs. context-stuffing**: use RAG (not fine-tuning) when data changes frequently or requires per-user access control — fine-tuning bakes all data into undifferentiated weights ("the Borg effect"), destroying access segmentation. Use RAG (not raw context-stuffing) at enterprise scale — cost, latency, and document-selection all favor retrieval over dumping everything into a huge context window, even at 1M+ token windows.

**Two-stage failure diagnosis (Ch6)**: faithful-incorrect (grounded in a stale/wrong source — fix ingestion) vs. unfaithful-incorrect (ignores/contradicts good context — fix the LLM/prompt). Route the fix to the right pipeline stage rather than defaulting to "better prompting."

**Guardrail layering**: no single checkpoint (source curation, retrieval-time scoring, post-generation auditing) is sufficient alone against bias, toxicity, hallucination, or prompt injection — use guardrails at every stage of the query flow.

**Reference-free evaluation (UMBRELA, AutoNuggetizer, HHEM)**: at production scale, maintaining human-curated golden chunks/answers is infeasible — use LLM-as-judge to score retrieval relevance (UMBRELA, 0-3 scale) and generation nugget-coverage (AutoNuggetizer) without ground truth. Treat your LLM judge like production code: version its prompt/model, log full reasoning, calibrate against human-verified samples to catch self-referential bias and evaluation drift.

**DIY vs. RAG platform**: evaluate per-component (embedding model, vector DB, retrieval, prompt, LLM, hallucination detection), not as one binary. RAG sprawl (independently built, incompatible RAG stacks across an org) is a real organizational cost once you have more than one RAG use case — a platform's centralized governance prevents policy drift and duplicated spend.

**Single-agent default, multi-agent by exception**: start with a single well-prompted agent (30-50% faster, cheaper, simpler). Escalate to multi-agent only for distinct security domains, vast tool surfaces (tool dilution), or organizational ownership boundaries. Prefer the orchestrator-worker pattern (no peer-to-peer subagent communication) over full peer-to-peer for predictability.

**Multimodal strategy split**: image summarization (VLM describes once at ingestion — cheap, but "locks in" the summary) vs. shared embedding space (CLIP/SigLIP — preserves detail for visual/aesthetic search, but misses precise numeric/text detail in charts and breaks text-based hybrid search/reranking). Choose by whether the *reasoning inside* the image or the *concept/aesthetic* of the image is what matters.

**Knowledge graphs for deterministic queries**: vector/hybrid search fails on time-bound facts, multi-constraint intersections, and multi-hop chains because embeddings deal in similarity, not deterministic relationships. Start with chunk enrichment (cheap, low-risk graph lookup enriching a vector-retrieved chunk); escalate to hybrid-graph retrieval (LLM-generated Cypher/SPARQL) only when query patterns demand true relational discovery. Use the 6-question adoption checklist (cheatsheet.md) before committing — KG maintenance is a persistent, often-underestimated operational cost.

**Production is a different project than POC**: response quality, latency, security/privacy, vendor integration, team expertise, and TCO must all be solved simultaneously — a POC's "it works" doesn't test any of these. DIY RAG TCO commonly overruns initial estimates by 3-5x. Staging-verify-promote (never write ingestion output directly to production) and CI/CD evaluation gates (no faithfulness/latency regression) are non-negotiable production disciplines.

**RAG's future role (Ch10)**: RAG becomes the "attention mechanism" for long-context LLMs — a high-precision filter deciding what deserves the model's expensive focus — not a workaround for small context windows. Context engineering (dynamic assembly of facts + history + instructions) succeeds prompt engineering as the more general framing.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-introduction-to-rag.md) | Introduction to RAG | R-G split, closed/open-book analogy, Borg effect (fine-tuning access control) |
| [ch02](chapters/ch02-the-base-rag-stack.md) | The Base RAG Stack | Ingestion/query flows, chunking strategies, ANN/HNSW, vector DBs |
| [ch03](chapters/ch03-scaling-your-rag-stack.md) | Scaling Your RAG Stack | Two-stage retrieval, hybrid search fusion, guardrails, hallucination taxonomy |
| [ch04](chapters/ch04-deploying-rag-to-production.md) | Deploying RAG to Production | Staging-verify-promote, microservice latency architecture, TCO, security surfaces |
| [ch05](chapters/ch05-the-rag-platform.md) | The RAG Platform | DIY vs. platform per-component, RAG sprawl, deployment models |
| [ch06](chapters/ch06-evaluating-your-rag-application.md) | Evaluating Your RAG Application | RAG failure taxonomy, UMBRELA/AutoNuggetizer, LLM-as-judge, offline/online eval |
| [ch07](chapters/ch07-from-rag-to-ai-agents.md) | From RAG to AI Agents | Agentic loop, agentic stack, MCP/A2A, single vs. multi-agent, agentic observability |
| [ch08](chapters/ch08-multimodal-rag.md) | Multimodal RAG | Table structured-context, image summarization vs. shared embedding, video keyframe extraction |
| [ch09](chapters/ch09-knowledge-enhanced-rag.md) | Knowledge-Enhanced RAG | Chunk enrichment, hybrid-graph retrieval, GraphRAG, entity linking, KG adoption checklist |
| [ch10](chapters/ch10-the-future-of-rag.md) | The Future of RAG | Context engineering, federated retrieval, proactive RAG, SLMs at the edge, compliance-by-design |

## Topic Index

- **Agentic loop / MCP / A2A** → ch07
- **Agentic observability / tracing** → ch07
- **Chunking strategies** → ch02, ch08 (tables)
- **Cost management / TCO** → ch03, ch04, ch05
- **Data ingestion at scale** → ch03, ch04
- **DIY vs. platform decision** → ch05
- **Embedding models** → ch02, ch08 (multimodal)
- **Entity linking / knowledge graphs** → ch09
- **Evaluation metrics (retrieval/generation)** → ch06
- **Fine-tuning vs. RAG** → ch01
- **GraphRAG (Microsoft)** → ch09
- **Guardrails / prompt injection** → ch03, ch04, ch08 (visual injection)
- **Hallucination detection & correction** → ch03, ch05 (Vectara), ch06, ch08 (multimodal)
- **Hybrid search** → ch02, ch03
- **Knowledge graphs** → ch09
- **Latency optimization** → ch03, ch04
- **Multi-agent systems** → ch07
- **Multimodal RAG (tables/images/audio/video)** → ch08
- **PII redaction / privacy** → ch04
- **Prompt engineering** → ch02, ch04
- **RAG evaluation offerings (Ragas, DeepEval, Open RAG Eval, Bedrock)** → ch06
- **RAG platforms (Vectara example)** → ch05
- **Reranking** → ch02, ch03
- **Retrieval-augmented generation (basics)** → ch01
- **Security & compliance at scale** → ch04, ch10
- **Small language models (SLMs)** → ch10
- **User experience / citations** → ch03, ch08 (visual citations)
- **Vector databases** → ch02, ch05
- **Vision-language models (VLMs)** → ch02, ch08

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills or ask the agent directly.
