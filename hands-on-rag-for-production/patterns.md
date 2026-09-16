# Patterns & Techniques

## Two-Stage Retrieval Pipeline
**When to use**: chunk volume is large enough that plain vector top-k search degrades in precision.
**How**: stage 1 (candidate generation) casts a wide, recall-optimized net via vector/lexical/hybrid search + metadata pre-filtering; stage 2 (reranking) applies a cross-encoder or business-logic reranker to the much smaller candidate set.
**Trade-offs**: bounded by stage-1 recall — a chunk missed in stage 1 can never be recovered in stage 2. Adds latency but is the standard architecture for production-scale retrieval accuracy.

## Hybrid Search (Semantic + Lexical Fusion)
**When to use**: queries include exact-match needs (IDs, names, rare tokens, typo tolerance) alongside conceptual/semantic ones.
**How**: run vector search and BM25/lexical search in parallel; fuse results via Reciprocal Rank Fusion (rank-based, no normalization needed) or weighted-score averaging (needs normalized scores).
**Trade-offs**: RRF adds more latency than weighted averaging but avoids cross-system score-scale mismatches.

## Reranking Chain (Relevance → MMR → Custom)
**When to use**: initial retrieval returns redundant or business-context-blind results.
**How**: apply a relevance cross-encoder reranker first, then MMR for diversity (reduce near-duplicates), then custom logic (recency, stock status, promotions) last.
**Trade-offs**: each stage adds latency/cost; only add the stages your use case actually needs.

## Staging-Verify-Promote Ingestion
**When to use**: any production RAG system, always — never write ingestion output directly to the live index.
**How**: ingest into a staging collection, run automated retrieval unit tests, promote to production only after tests pass.
**Trade-offs**: adds pipeline complexity and a promotion step, but prevents a single bad ingestion run from silently corrupting live results for all users.

## Detect-Then-Correct Hallucination Pipeline
**When to use**: high-stakes domains where a wrong-but-confident answer is costly (finance, legal, healthcare).
**How**: a detection model/judge (HHEM or LLM-as-judge) flags a likely hallucination; a correction model rewrites the response grounded in the same retrieved chunks, with per-span explanations.
**Trade-offs**: extra latency for both detection and correction calls; worth it when trust/compliance outweighs speed.

## Entity-Aware (Typed) Redaction
**When to use**: PII/PHI must be removed from ingested content while preserving usability.
**How**: replace sensitive values with category tokens (`[PERSON]`, `[MEDICATION]`) instead of generic masking/nulling, using tools like Microsoft Presidio.
**Trade-offs**: more engineering than blanket masking, but preserves the semantic relationships needed for the RAG response to remain useful.

## Semantic Caching
**When to use**: users frequently phrase the same underlying question differently.
**How**: cache keyed on query-embedding cosine similarity (threshold ~0.85) rather than exact string match; invalidate via event-driven purge (ingestion pipeline publishes document-update events) rather than TTL alone.
**Trade-offs**: adds embedding-comparison overhead per cache lookup; dramatically improves hit rate for paraphrased repeat queries.

## Cascading Model Routing
**When to use**: cost control across a high query-volume production system.
**How**: route every query first to a small/cheap/fast LLM; escalate to a frontier model only if the cheap model reports low confidence or the query is flagged complex.
**Trade-offs**: requires a reliable confidence signal from the cheap model; can dramatically cut average per-query cost.

## Table-as-Structured-Context
**When to use**: any table embedded in a source document.
**How**: extract table → convert to JSON (row-dict-of-columns) or Markdown → summarize at ingestion as a "pointer chunk" → at query time inject the full structured JSON (not flattened prose) into the generation prompt.
**Trade-offs**: naive text-chunking of tables produces "headless" chunks that lose header-to-cell relationships — this pattern avoids that entirely.

## Multi-Page Table Stitching
**When to use**: tables that span multiple PDF pages.
**How**: detect consecutive-page fragments with matching column counts, strip duplicate repeated headers, concatenate into one master dataframe/JSON before embedding.
**Trade-offs**: requires a post-processing heuristic layer; without it, multi-page tables fragment into duplicate-header noise or headless data.

## Image Summarization vs. Shared Embedding Space
**When to use**: summarization for reasoning-heavy visuals (charts, diagrams with precise data); shared embedding (CLIP/SigLIP) for concept/aesthetic visual search.
**How**: summarization — VLM describes the image once at ingestion into a text chunk, store raw image separately with a reference ID. Shared embedding — map image and text into one contrastive-trained latent space, retrieve via cosine similarity.
**Trade-offs**: summarization "locks in" whatever detail the summary captured; shared embeddings miss fine-grained numeric/textual detail and break text-based hybrid search/reranking.

## Blind Verification Workflow (Multimodal Hallucination Mitigation)
**When to use**: high-stakes visual judgments (insurance claims, industrial inspection) where a leading question risks visual sycophancy.
**How**: pass the image to the VLM with a neutral prompt first ("describe the condition") to get an unbiased description; a secondary text-only LLM compares the user's leading question against that neutral description and flags discrepancies.
**Trade-offs**: doubles VLM calls per query; necessary when false-positive confirmation bias carries real cost.

## Orchestrator-Worker Multi-Agent Pattern
**When to use**: task complexity justifies multi-agent decomposition (distinct security domains, vast tool surfaces, organizational boundaries) but full peer-to-peer coordination is too unpredictable.
**How**: a central orchestrator decomposes the task and calls specialized subagents in parallel; subagents never communicate directly with each other.
**Trade-offs**: more predictable and traceable than peer-to-peer multi-agent systems, while still gaining context isolation and specialization benefits.

## Chunk Enrichment (Graph-Augmented RAG)
**When to use**: vector search finds the right text but the chunk lacks context (names, dates, relationships) needed to answer.
**How**: identify entities in the retrieved chunk, perform a graph lookup to pull missing facts, inject the enriched context (chunk + graph facts) into the generation prompt.
**Trade-offs**: near-zero latency/risk (simple indexed lookup) — the recommended first step before investing in full hybrid-graph retrieval.

## Reference-Free Evaluation (UMBRELA / AutoNuggetizer)
**When to use**: production-scale evaluation where maintaining golden chunks/answers is infeasible.
**How**: use an LLM judge to score retrieval relevance per-chunk (UMBRELA, 0-3 scale) or decompose expected facts into "nuggets" and check generation coverage (AutoNuggetizer) — no pre-curated ground truth needed.
**Trade-offs**: trades human labeling cost for LLM inference cost and judge-model bias risk; requires periodic calibration against human-verified samples.
