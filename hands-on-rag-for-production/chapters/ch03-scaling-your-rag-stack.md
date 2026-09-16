# Chapter 3: Scaling Your RAG Stack

## Core Idea
At enterprise scale, "making RAG work" gives way to "making RAG trustworthy": ingestion must survive millions of documents without breaking, retrieval needs a two-stage (candidate generation + reranking) architecture instead of plain vector search, and guardrails/hallucination-control become non-negotiable production requirements.

## Frameworks Introduced
- **The Two-Stage Retrieval Pipeline**: candidate generation (fast, high-recall — vector/lexical/hybrid search + metadata pre-filtering) followed by reranking (slow, high-precision — cross-encoder or business-logic reordering of the small candidate set).
  - When to use: any time plain top-k vector search degrades as chunk volume grows into the millions.
  - How: stage 1 optimizes for recall even with noise; stage 2 spends compute only on the much smaller candidate set. Bounded by stage-1 recall — if stage 1 misses a relevant chunk, stage 2 can never surface it.
- **Hybrid search fusion**: combine semantic (vector) + lexical (BM25/inverted-index) results via Reciprocal Rank Fusion (rank-based, no score normalization needed) or weighted-average scoring (needs normalized scores, simpler to implement).
- **Reranking taxonomy**: relevance reranking (cross-encoder joint query+chunk scoring), MMR/diversity reranking (1998; balances relevance vs. redundancy via a λ parameter), and custom/business-logic reranking (e.g. recency, stock status, promotions). Often chained: relevance → MMR → custom.
- **Guardrail layering**: pre-retrieval (source curation for bias), retrieval-time (bias scoring folded into reranking), and post-generation (prompt instructions + specialized auditor models like ShieldGemma/Llama Guard) — guardrails aren't a single checkpoint but a pipeline of checkpoints.
- **RAG Hallucination Taxonomy** (from FaithBench): **questionable** (ambiguous, interpretation-dependent), **benign** (unsupported by source text but consistent with common sense/reasonable inference, often harmless or helpful), **unwanted** (clearly misleading or factually wrong relative to source) — use this to calibrate how aggressively to block/flag.
- **Detect-then-correct hallucination pipeline**: a detection model/judge flags a likely hallucination → a correction model rewrites the response grounded in the same retrieved chunks → corrected response replaces the original, at the cost of extra latency.

## Key Concepts
- **Candidate generation**: stage-1 retrieval optimized for recall over precision.
- **Cross-encoder**: a reranker architecture that jointly processes query+chunk (vs. embeddings, which score independently then compare post hoc).
- **Idempotency**: a pipeline task can be safely retried without side effects — required for large-scale ingestion resilience.
- **Change data capture (CDC)**: detecting source-data changes (DB triggers, transaction log tailing) to drive incremental re-indexing instead of full re-indexing.
- **Direct vs. indirect prompt injection**: direct = malicious instructions in the user's own query; indirect = malicious instructions planted in ingested documents/web pages that get triggered when a legitimate user's query surfaces them.
- **Instruction defense**: using explicit delimiters (XML tags) plus system-prompt instructions telling the LLM to treat user/context content as data, not commands.
- **HHEM (Hughes Hallucination Evaluation Model)**: a dedicated small classifier (vs. LLM-as-judge) that scores factual consistency 0–1; faster, cheaper, more consistent than using a full LLM as judge.
- **RAG hallucination vs. general LLM hallucination**: RAG hallucination = output inaccurate *despite* being grounded in retrieved data (retrieval failure, data-quality failure, or generation-faithfulness failure) — a narrower, pipeline-specific failure mode.

## Mental Models
- Think of the two-stage pipeline as "wide net, then fine sieve" — cast recall-optimized stage 1 broadly, then let stage 2's expensive cross-attention do the precision work only on survivors.
- Diagnose a hallucination by asking, in order: (1) did retrieval fail to surface the right chunk? (2) is the ingested data itself wrong/stale/conflicting? (3) did the LLM ignore/misinterpret good retrieved context? — each has a different fix (better retrieval vs. data cleanup vs. prompt/guardrail).
- Treat guardrails like defense-in-depth in security: no single layer (curation, retrieval-time scoring, post-generation auditor) is sufficient alone; combine them.
- Cost at scale is a "multilayered architecture" problem, not a single LLM-rate problem — storage, embedding compute, retrieval compute, and LLM inference each need their own optimization lever (caching, quantization, dynamic model routing).

## Anti-patterns
- **Single-threaded, monolithic ingestion scripts at scale**: brittle (one corrupt document crashes a multi-day job) and slow (should-be-hours jobs stretch to weeks). Use parallelization (Ray/Dask/Spark) + orchestration (Airflow) + idempotent, restartable tasks instead.
- **Loading giant documents entirely into memory**: a 17,000-page manual will OOM-crash a naive pipeline; stream/process page-by-page or split into page-range chunks assigned to parallel workers (being careful not to split mid-table).
- **Full re-indexing on every data change**: prohibitively slow/expensive at scale; use CDC-driven incremental updates instead.
- **Relying on relevance reranking alone when diversity matters**: e.g. summarizing customer reviews — pure relevance reranking can return near-duplicate chunks; add MMR to force diverse perspectives.
- **Using a general LLM as reranker by default**: works but is slower, more expensive, and can hallucinate compared to a dedicated cross-encoder reranker — reserve for cases where you already have LLM infra and reranking quality bar is low.
- **Assuming RAG's grounding eliminates hallucination risk**: RAG *reduces* hallucination but the LLM can still misinterpret, over-extrapolate, ignore context in favor of parametric knowledge, or handle conflicting chunks poorly — detection/correction guardrails are still required.
- **Skipping instruction defense (XML delimiters) in RAG prompts**: without clear boundaries between system instructions, context, and user query, prompt injection (direct or indirect, e.g. hidden white-on-white text in a PDF) is far easier.

## Code Examples

```python
from sentence_transformers.cross_encoder import CrossEncoder

model = CrossEncoder('BAAI/bge-reranker-v2-m3')
sentence_pairs = [[query, doc] for doc in documents]
scores = model.predict(sentence_pairs, show_progress_bar=True)

docs_with_scores = list(zip(documents, scores))
reranked = sorted(docs_with_scores, key=lambda x: x[1], reverse=True)
```
- **What it demonstrates**: cross-encoder reranking — jointly scoring (query, chunk) pairs re-orders candidates far more accurately than embedding similarity alone (e.g. correctly promoting "transformer architecture handles long-range dependencies" over an RNN-focused chunk for a transformer-benefit question).

```python
# HHEM hallucination scoring
from transformers import pipeline, AutoTokenizer

prompt = "<pad> Determine if the hypothesis is true given the premise?\n\nPremise: {text1}\n\nHypothesis: {text2}"
input_pairs = [prompt.format(text1=pair['article'], text2=pair['summary']) for pair in example_pairs]

classifier = pipeline(
    "text-classification",
    model='vectara/hallucination_evaluation_model',
    tokenizer=AutoTokenizer.from_pretrained('google/flan-t5-base'),
    trust_remote_code=True
)
full_scores = classifier(input_pairs, top_k=None)
hhem_scores = [
    round(d['score'], 4) for s in full_scores for d in s if d['label'] == 'consistent'
]
# [0.9182, 0.0823] — high score for faithful summary, low for one with a fabricated detail
```
- **What it demonstrates**: a dedicated hallucination classifier scores factual consistency (premise=source, hypothesis=generated summary) far more cheaply/consistently than an LLM-as-judge call.

```python
# Instruction-defense prompt template (mitigates prompt injection)
"""
Here is a user query:
<query>
{query}
</query>

And relevant context:
<context>
{context}
</context>

Please respond to the user query using the context
"""
```
- **What it demonstrates**: XML delimiters separate untrusted user/context text from system instructions, making it harder for injected text to be mistaken for a command.

## Reference Tables

| Vector DB | Instant-indexing support | Notes |
|---|---|---|
| Qdrant | Very High | Rust-built, explicitly designed for real-time updates |
| Pinecone | High | Fully managed; latency varies with load/pod config |
| Weaviate | High | HNSW-based near real-time; depends on config/hardware |
| Milvus | Medium-high | Multiple index types (HNSW, IVF); latency sensitive to index choice |
| Elasticsearch/OpenSearch | Medium | KNN via Lucene HNSW; refresh interval default 1s |

| Reranker | License/cost | Notes |
|---|---|---|
| Sentence Transformers cross-encoders | Open source (Apache 2.0) | BERT/RoBERTa-based, highly customizable |
| BGE Reranker (BAAI) | Open source (Apache 2.0) | Efficient, strong multilingual |
| Cohere Rerank | Commercial | Managed API, production-optimized, multilingual |
| Vectara rerankers | Commercial/turnkey | Platform-only, 100+ languages |
| Voyage AI rerankers | Commercial | Managed API, domain-adapted |

## Worked Example
Cost model for a mid-size enterprise deployment: 2M documents × 20 pages = 40M pages ≈ 800 tokens/page ⇒ 3.2B tokens total corpus. Embedding with `text-embedding-large-3` at $0.13/1M tokens ⇒ ~$4,160 initial embedding cost (+5–10%/month for ongoing updates). At 150K queries/month with ~4K input + 1K output tokens per query, LLM generation cost ≈ $3,000/month. Add ~$500/month each for vector DB, general compute, and monitoring/DevOps ⇒ total steady-state cost ≈ $4,916/month plus the one-time ~$4,160 embedding investment. This shows cost at scale is dominated by generation (LLM calls), not embedding, and that infrastructure (DB + compute + observability) is a comparable line item to the LLM API bill itself — reinforcing the chapter's point that cost control requires a multilayered approach (caching, model routing, quantization), not just picking a cheaper LLM.

## Key Takeaways
1. Scale breaks naive single-threaded ingestion — design for parallelism, idempotency, and restartability from the start, not as an afterthought.
2. Vector search alone degrades as chunk count grows; production-grade retrieval needs the two-stage pipeline (candidate generation + reranking), and often hybrid search to catch exact-match/rare-token cases semantic search misses.
3. Guardrails must operate at multiple points (source curation, retrieval-time scoring, post-generation auditing) — no single checkpoint is sufficient against bias, toxicity, or prompt injection.
4. RAG reduces but does not eliminate hallucination — classify failures by root cause (retrieval, data quality, or generation faithfulness) to pick the right fix, and use the questionable/benign/unwanted taxonomy to calibrate response strictness.
5. Dedicated hallucination-evaluation models (HHEM) beat LLM-as-judge on cost, speed, and consistency for high-volume production checks.
6. Cost at scale is dominated by LLM inference, not embedding — optimize via caching (responses, embeddings, retrieved chunks), dynamic model routing, and quantization rather than only shopping for cheaper LLMs.
7. UX is not an afterthought: latency sensitivity, clear source attribution/citations, process transparency, and feedback mechanisms (thumbs up/down, stored for evaluation) materially affect user trust and system improvability.

## Connects To
- **Ch2**: extends the base vector-search retrieval and prompt-engineering foundations with hybrid search, reranking, and guardrails.
- **Ch4**: security/privacy topics only briefly introduced here (prompt injection) are covered in depth for production deployment; also covers TCO in more detail.
- **Ch6**: hallucination detection (HHEM, LLM-as-judge) here is the seed for the full RAG evaluation framework.
- **Ch7**: prompt injection risk is noted as "even more dangerous" once agents can take actions — expanded in the agentic RAG chapter.
