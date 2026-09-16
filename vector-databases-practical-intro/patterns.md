# Patterns — Vector Databases: A Practical Introduction

## Hybrid Database Architecture
**When to use**: any real vector-search system needing both semantic retrieval and structured filtering/sorting.
**How**: separate vector component (similarity search, FAISS-style indexing) from metadata component (relational/NoSQL) joined by a foreign key; add an integration layer that combines similarity-ranked candidates with SQL-style predicate filtering.
**Trade-offs**: more moving parts than a single store, but avoids the dead-end of storing metadata as vectors (poor filtering) or vectors as plain SQL columns (poor similarity performance).

## Overfetch-Then-Filter
**When to use**: a vector index/extension whose native filter pushdown into the nearest-neighbor search is unreliable or unsupported.
**How**: fetch `limit × candidate_multiplier` (book uses 10×) candidates by similarity first, join to metadata, then apply `WHERE` filters, then truncate to `limit`.
**Trade-offs**: costs extra retrieval work per query, but produces stable, correct results where native pushdown would silently degrade or fail.

## Hybrid Search Score Fusion
**When to use**: semantic search alone misses exact technical terms/names; keyword search alone misses paraphrased/conceptually-related content.
**How**: normalize each method's scores into a comparable range (cosine similarity is already [0,1]; BM25 is unbounded — divide by max in result set), then combine `final = w1*semantic + w2*keyword` (book default 0.7/0.3, weights sum to 1.0). Overfetch (2×) from each method before fusing.
**Trade-offs**: adds a tuning parameter (the weight split); default 0.7/0.3 is a reasonable starting point — raise keyword weight if results feel too vague/generic.

## Sliding-Window Chunking with Overlap
**When to use**: any document too long for a single embedding to represent faithfully.
**How**: split into 512–1,024 token chunks (book's academic-paper number) with ~20% overlap between consecutive chunks so boundary-spanning information isn't lost.
**Trade-offs**: overlap increases storage/embedding cost proportionally; too little overlap loses boundary context, too much wastes compute on duplicated content.

## Multilevel Semantic Search
**When to use**: a domain with both summary-level and detail-level content (paper abstracts + sections; doc overviews + API pages) and users ask both discovery and detail questions.
**How**: embed the query once, branch to parallel searches at each granularity (e.g. top-3 abstracts + top-5 sections), merge by similarity, sort descending, take combined top-k.
**Trade-offs**: doubles the search work per query but avoids forcing users to specify which granularity they mean.

## Similarity Threshold Filtering
**When to use**: whenever "return top-k regardless of actual relevance" risks feeding irrelevant context to an LLM (a direct hallucination contributor).
**How**: filter out results below a calibrated similarity threshold (book's default 0.7 for pgvector cosine on scientific text) *before* ranking/limiting, not just deprioritizing them.
**Trade-offs**: can return fewer than `k` results (or zero) when nothing is truly relevant — treat this as a correct, informative outcome, not a bug to paper over.

## Model-Tagged Embeddings
**When to use**: any system where the embedding model might change over the system's lifetime.
**How**: store an `embedding_model` column alongside each vector; on model swap, `WHERE embedding_model != 'new-model'` identifies exactly which rows need re-embedding.
**Trade-offs**: trivial storage cost; avoids an all-or-nothing full re-embed later.

## Three-Table Normalization (metadata / text / vectors)
**When to use**: systems where text is frequently read/displayed but vectors are only used for math (chat history, document archives).
**How**: split into `<entity>` (high-level metadata), `<entity>_content` (bulky text), and `<entity>_embeddings` (fixed-width vector) tables, joined by shared key.
**Trade-offs**: extra joins per query, but keeps the hot-path text table lean and makes re-indexing after a model swap a targeted operation on one small table.

## Symmetric Context-Window Retrieval
**When to use**: a single retrieved unit (message, line, chunk) is only meaningful with immediate neighbors.
**How**: given a matched item at ordinal position N, retrieve items in `[N − window, N + window]` (book uses window=3, i.e. 7 total), flag the actual match with `is_target`.
**Trade-offs**: requires an explicit ordering column (`message_index` or equivalent) designed in from the start.

## Processing-Status State Machine
**When to use**: any multi-stage async ingestion pipeline (download → extract → chunk → embed) that must be resumable and inspectable.
**How**: boolean flags per stage (`pdf_downloaded`, `pdf_processed`, `embedding_generated`) plus a `processing_queue` table (status, priority, retry_count, error_message) as the single source of truth for "what work remains."
**Trade-offs**: more schema surface than a fire-and-forget script, but makes partial failures recoverable via a query instead of a manual audit.

## Index Selection by Constraint (FAISS/pgvector)
**When to use**: choosing any vector index type.
**How**: work through the checklist — dataset size (small→flat, large→IVF/HNSW), dimensionality (high→HNSW), accuracy (exact→flat, approximate OK→IVF/HNSW), memory (tight→PQ), speed (max→HNSW then IVF), hardware (GPU available→GPU variants).
**Trade-offs**: no single "best" index; the right choice is workload-specific and may need revisiting as data grows.

## Geometric Median / Medoid for Robust Aggregation
**When to use**: computing a "representative vector" for a cluster/category where outliers are expected.
**How**: use geometric median (`argmin Σ‖x−vᵢ‖`, iterative) instead of centroid (`argmin Σ‖x−vᵢ‖²`, closed-form) when robustness matters; use medoid when you need an actual data point, not a synthetic one.
**Trade-offs**: geometric median/medoid cost more to compute (iterative/search-based) than the closed-form centroid.
