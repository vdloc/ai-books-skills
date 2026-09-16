# Chapter 8: Best Practices

## Core Idea
Enterprise GenAI success comes from disciplined engineering across three areas — Design (right-fit LLM use, modularity, security-by-design, latency/accuracy/resilience trade-offs), Implementation (efficient inference routing, retrieval quality, freshness, auditability, adaptability), and Evaluation (continuous monitoring + active learning) — not from a fine-tuned model alone.

## Frameworks Introduced
- **LLM Fit Assessment**: before building, explicitly weigh inference cost, hallucination risk, memory constraints, latency, and maintenance overhead against expected benefit — many tasks (e.g., extracting invoice fields) are better solved by a deterministic parser or classical ML than an LLM.
  - When to use: at problem-scoping time, always, before any architecture decision — "avoid tech-first approaches," start from the business problem.
- **Modular-by-Design Architecture**: decompose into independently deployable, loosely coupled blocks (ingestion, retrieval, LLM interaction layer) with defined I/O interfaces; use the adapter pattern for LLM interaction to enable model/provider swapping without touching the rest of the codebase; use semantic versioning ([Major.Minor.Patch]) for module compatibility tracking.
  - When to use: anytime you expect model providers, prompts, or retrieval strategies to change over the system's life (i.e., always, for production GenAI).
- **Security-by-Design checklist**: RBAC at every layer (data lake, blob, vector DB, APIs, LLM endpoints) via cloud-native identity (Entra ID); encryption at rest and TLS in transit; never expose inference endpoints publicly (route via API gateway/private endpoint); field-level data classification (PII/PCI/PHI/confidential) with masking before it reaches the LLM; full audit logging into SIEM; secrets in a vault, never hardcoded; select "no data retention" model endpoints where available.
  - How: apply principle of least privilege at every layer — data stores, UI, model, services, APIs — and never assume trust between services.
- **Design-for-What-Matters trade-off selection**: explicitly choose whether latency, accuracy, or resilience is the priority for a given use case, then apply the matching technique set (see Reference Tables) — don't try to globally optimize all three.
- **Conditional LLM Routing**: pre-LLM filter layer (keyword detection, regex, metadata tagging, or a lightweight classifier) that bypasses the LLM entirely when a cached vector-store result, a structured-DB lookup, or a prewritten function can answer the query — cuts cost and latency by reserving LLM calls for genuinely novel, language-understanding-requiring queries.
- **Multi-index Retrieval Strategy**: separate indexes by topic/data-type/function rather than one monolithic index; tag with metadata at indexing time for pre-retrieval filtering; chunk along semantic boundaries (never split a table or a problem-solution pair mid-way); route queries dynamically to the right sub-index via keyword-to-index mapping.
- **Active Learning Loop**: capture user feedback (thumbs up/down, ratings) → label failed/uncertain responses → remediate (fine-tune if errors are pattern-based; enrich/re-chunk the index if errors are retrieval-based) → align remediation cadence to sprint cycles (daily shadow evals, weekly error aggregation, biweekly retrain/reindex).

## Key Concepts
- **Cache-Augmented Generation (CAG)**: a lighter-weight alternative to RAG that uses cached/precomputed context instead of a fresh retrieval call — apply conditional routing so not every query pays the full RAG cost.
- **Delta vector updates**: recompute and update only the embeddings for changed/new source documents (via timestamp/checksum change detection) rather than full reindexing — keeps a vector DB (e.g., Azure AI Search) fresh without prohibitive recompute cost.
- **Document versioning log**: a metadata-store record of document versions used to invalidate/deprioritize stale embeddings — a freshness-and-trust mechanism distinct from delta updates.
- **Traceability chain**: for every generated answer, log the chunk IDs, source documents, and versions that contributed — this is what makes a GenAI system auditable/explainable in regulated industries (healthcare, finance, legal).
- **RPM/TPM thresholds**: requests-per-minute / tokens-per-minute caps set proactively to avoid overloading an inference endpoint, distinct from provider-imposed rate limits.

## Mental Models
- **"Not every problem needs a generative solution"**: the chapter's single most load-bearing principle — treat LLM/GenAI as one tool among several (rule engines, classical ML, cached lookups), selected only when the task genuinely requires language understanding, generation, or reasoning.
- **Multipass reasoning as "paying extra for proofreading"**: higher per-call inference cost is often net-cheaper than the downstream cost of human rework/escalation caused by single-pass errors — always compute total cost of ownership (inference + rework), not just per-call inference cost, before rejecting a more expensive but more accurate approach.
- **"What you retrieve is only as good as what and how you've stored it"**: retrieval quality is a storage/indexing design problem as much as a query-time problem — poor chunking or a monolithic index caps your ceiling on accuracy no matter how good the LLM is.
- **Adaptability is a designed property, not an emergent one**: configuration-driven architecture (external prompt libraries, feature flags, pluggable chunking logic) is what lets an operational team evolve a system post-deployment without new engineering cycles — plan for this explicitly at design time.

## Anti-patterns
- **Force-fitting LLMs where rule-based/classical ML suffices**: higher cost, hallucination risk, and maintenance burden for no accuracy gain on structured, well-defined extraction tasks.
- **Hardcoding model-specific logic**: creates vendor lock-in and makes model/provider swaps expensive — always go through an adapter/abstraction layer.
- **Publicly exposed inference endpoints**: a basic but recurring security gap — always route through API gateways/private endpoints with network rules.
- **Monolithic vector indexes mixing unrelated document types**: degrades retrieval precision and doesn't scale as data grows — always separate by topic/type/function.
- **Full reindexing on every data change**: wastes compute and money — use change detection + delta updates instead.
- **No traceability from output back to source**: makes root-cause analysis of content disputes impossible in regulated-industry deployments — log chunk IDs/sources/versions from day one, not after an audit finding.
- **Retraining/reindexing ad hoc, disconnected from sprint cycles**: active-learning remediation without a defined cadence (daily shadow eval → weekly labeling → biweekly retrain) becomes reactive firefighting instead of continuous improvement.

## Reference Tables
**Design-for-what-matters technique map**
| Priority | Techniques |
|---|---|
| Latency | Smaller/quantized models (e.g., text-embedding-3-small), lightweight models (GPT-4o-mini), minimize token input (context filtering, summary chaining, memory pruning), parallelize retrievals (async multi-index calls) |
| Accuracy | Fine-tune/instruct-tune on domain data, multipass reasoning + self-verification/chain-of-thought, high-quality RAG indexes (metadata filtering, semantic scoring) |
| Resilience | Retrain/reprompt pipelines triggered by failure patterns, automated benchmark regression per release, rule-based fallback routing when model confidence is below threshold |

**Multipass reasoning ROI example (from the book)**
| Item | Single-pass | Multipass | Difference |
|---|---|---|---|
| Inference cost | $1,000/mo | $1,500/mo | +$500 |
| Human rework/escalation | $12,000/mo | $6,000/mo | −$6,000 |
| Total monthly cost | $13,000 | $7,500 | **−$5,500 net benefit** |

(Assumptions: $0.05/API call, multipass ≈1.5× calls, ~2,000 tasks/mo, $20/task rework cost, error rate 30%→15%.)

**Security-by-design checklist (condensed)**
| Layer | Control |
|---|---|
| Identity/access | RBAC + managed identity (e.g., Entra ID) at every layer |
| Data at rest | Server-side encryption |
| Data in transit | TLS enforced on all component communication |
| Inference endpoints | Never public; route via API gateway/private endpoint |
| Sensitive fields | Classify as PII/PCI/PHI/confidential; mask before LLM context |
| Secrets | Vault-managed (e.g., Key Vault), never hardcoded |
| Logging | Full audit trail (query, model version, context, user ID, timestamp, output) into SIEM |
| Model endpoint config | Prefer "no data retention"/no-log endpoints |

## Worked Example
**Conditional LLM routing decision flow** (implementation section): an incoming query passes through a pre-LLM filter (keyword/regex/metadata classifier). If a valid cached vector-store result exists (e.g., in Redis), or the answer can be fetched directly from a structured database (e.g., Snowflake), or a prewritten function already covers it — the LLM is bypassed entirely. Only queries that genuinely require language understanding/generation reach the LLM layer. RPM/TPM thresholds are additionally enforced to prevent overload. This is the concrete mechanism behind "not every functionality needs to be LLM-powered."

**Active learning remediation branch**: a failed/low-confidence query is labeled. If the error is *pattern-based* (the model consistently mishandles a certain phrasing or domain term), the fix is fine-tuning via the provider's fine-tuning API (e.g., Azure OpenAI). If the error is *retrieval-based* (missing or irrelevant documents surfaced), the fix is enriching the index or improving chunking/metadata — never fine-tune to fix a retrieval problem, and never re-chunk to fix a genuine model-reasoning gap.

## Key Takeaways
1. Scope every GenAI project by first asking "does this need an LLM at all?" — default to the cheapest tool (rule-based/classical ML) that meets the accuracy bar.
2. Architect for modularity and provider-independence from day one (adapter pattern, config-driven prompts, semantic versioning) — model landscape churn is a certainty, not a risk.
3. Security-by-design means RBAC + encryption + private endpoints + field-level PII/PHI/PCI masking + full audit logging, applied at every layer, from the first commit — not retrofitted after a security review.
4. Explicitly choose latency, accuracy, or resilience as your primary optimization target per use case, and apply the matched technique set — trying to maximize all three simultaneously is how systems become both slow and expensive.
5. Compute total cost of ownership (inference + human rework) before rejecting a higher-latency, higher-accuracy approach like multipass reasoning — the book's own numbers show a net $5,500/month savings despite 50% higher inference cost.
6. Treat retrieval quality as a storage/indexing design problem: separate indexes by type, tag with metadata, chunk along semantic boundaries, and use delta updates instead of full reindexing.
7. Close the loop with active learning on a defined cadence (daily shadow eval → weekly error labeling → biweekly retrain/reindex aligned to sprints) — continuous improvement needs a schedule, not ad hoc firefighting.

## Connects To
- **Ch 3**: reuses and operationalizes the design-pattern vocabulary (microservices, adapter-style prompt strategy patterns) into concrete "modular by design" guidance.
- **Ch 5**: the security-by-design checklist here is the generalized version of IDFE's specific RBAC/Key Vault/Entra ID/encryption implementation.
- **Ch 6**: continuous evaluation practices here (HELM, G-Eval, MT-Bench, human-in-the-loop) directly extend the evaluation framework chapter into an ongoing operational discipline.
- **Ch 7**: security-by-design and auditability practices here are concrete implementations of the Privacy & Security and Accountability RAI principles.
- **Ch 4**: the LLM-fit assessment here reprises the "not every agent needs an LLM" principle from the agentic AI chapter, generalized to the whole system-design stage.
