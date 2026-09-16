# Glossary — Vector Databases: A Practical Introduction

**ANN (Approximate Nearest Neighbor)** — search that trades exactness for speed; the standard target for GenAI similarity search, since exact matches aren't required (Ch 1, Ch 3)

**BM25 (Best Matching 25)** — probabilistic keyword-ranking function used by FTS5; models term saturation and document-length normalization (Ch 6)

**Centroid** — the mean vector of a set; closed-form (argmin Σ‖x−vᵢ‖²), but easily dragged by outliers (Ch 9)

**Collection (VQL)** — top-level container for vector data, analogous to a database (Ch 9)

**Context window (conversation)** — a symmetric span of surrounding messages (e.g. ±3) retrieved around a matched message so it remains understandable (Ch 8)

**Cosine similarity** — cosine of the angle between two vectors; 1 = identical, near 0 = unrelated; the default metric for normalized text embeddings (Ch 1, Ch 3, Ch 5, Ch 7)

**Distance metric** — L2 (Euclidean), inner-product, or cosine distance; choice must match what the embedding model was optimized for (Ch 1, Ch 3)

**Embedding** — (verb) the process of mapping unstructured data to a vector; (noun) the resulting vector or vector set (Ch 1, Ch 2)

**Embedding drift / model versioning** — the need to track which embedding model produced each vector (`embedding_model` column) so switching models doesn't silently break similarity comparisons (Ch 4)

**`ef_construction` (HNSW)** — search breadth during index build; higher = better recall, slower build (Ch 3, Ch 5, Ch 8)

**FAISS (Facebook AI Similarity Search)** — library implementing flat, IVF, LSH, PQ, and HNSW index types for vector similarity search (Ch 3)

**FTS5** — SQLite's full-text search virtual table extension, providing the `bm25()` ranking function (Ch 6)

**Geometric median** — the vector minimizing Σ‖x−vᵢ‖ (no square); no closed form, computed iteratively, robust to outliers unlike the centroid (Ch 9)

**GIN index** — Postgres index type used for array-containment queries and (with `gin_trgm_ops`) trigram fuzzy text matching (Ch 5)

**HNSW (Hierarchical Navigable Small World)** — graph-based ANN index; generally fastest search, best for high-dimensional data, at higher memory cost than PQ (Ch 3, Ch 5, Ch 8)

**Hybrid database architecture** — vector component + metadata component + integration layer, combining semantic and structured search (Ch 1)

**Hybrid search** — combining semantic (vector) and keyword (BM25/FTS) search via weighted, normalized score fusion (Ch 6, Ch 9)

**HYSWNSW `m` parameter** — max bidirectional links per graph node; trades index size for search quality (Ch 3, Ch 5, Ch 8)

**IVF (Inverted File) index** — clusters vector space into `nlist` partitions, searches only `nprobe` clusters at query time (Ch 3)

**Knowledge distillation** — training a small embedding model to mimic a larger one (e.g. all-MiniLM-L6-v2) for speed/size gains (Ch 2)

**LSH (Locality-Sensitive Hashing)** — hashes similar vectors into shared buckets via random projections/rotations (Ch 1, Ch 3)

**Medoid** — the actual data point (not a computed point) minimizing sum of distances to all others in a collection (Ch 9)

**Metadata (vector DB)** — structured data associated with a vector but outside the vector space itself (title, date, category) (Ch 1, Ch 9)

**Multilevel semantic search** — parallel search across granularities (e.g. abstract-level + section-level), merged and re-ranked (Ch 7)

**`nprobe` (IVF)** — number of clusters searched at query time; the primary IVF accuracy/speed tuning knob (Ch 3)

**Overfetch-then-filter** — fetching more candidates than needed (e.g. `limit × 10`) before applying SQL metadata filters, improving stability when native filter pushdown is unreliable (Ch 4, Ch 6)

**pgvector** — PostgreSQL extension adding a native `vector` column type and HNSW/IVFFlat indexing with cosine/L2/inner-product operator classes (Ch 5, Ch 7, Ch 8)

**PQ (Product Quantization)** — splits vectors into subspaces, quantizes each via a learned codebook, for large memory savings (Ch 3)

**Quantization** — reducing a vector's representational resolution (bits/dimensions) to trade accuracy for space/speed (Ch 3)

**RAG (Retrieval-Augmented Generation)** — embed query → retrieve similar content from vector DB → pass as context to an LLM for generation (Ch 2, Ch 6, Ch 7, Ch 8)

**Semantic search** — retrieval by meaning ("find pages that answer this") vs. keyword search ("find pages containing these words") (Ch 1)

**sentence-transformers** — Python library of pretrained embedding models (all-MiniLM-L6-v2, all-mpnet-base-v2, paraphrase-multilingual-mpnet-base-v2) (Ch 2)

**Sliding-window chunking** — splitting documents into overlapping token chunks (e.g. 512–1,024 tokens, ~20% overlap) so information at boundaries isn't lost (Ch 5)

**sqlite-vss** — SQLite extension pair (`vector0`+`vss0`) providing FAISS-backed vector virtual tables (Ch 4, Ch 6)

**Three-table normalization (conversation schema)** — separating metadata / text / embeddings into distinct tables to keep hot-path text lean and simplify re-embedding (Ch 8)

**Threshold (similarity)** — minimum acceptable similarity score before a result is considered relevant; ~0.7 is the book's "sweet spot" for pgvector cosine on scientific text, ≤0.6 risks "hallucinated relevance" (Ch 7)

**Vector (data type)** — an ordered list of floats supporting native similarity operations, unlike an opaque BLOB (Ch 1)

**Vector arithmetic** — semantic relationships expressible as vector math, e.g. `king − man + woman ≈ queen` (Ch 1, Ch 2)

**VQL (Vector Query Language)** — a proposed, not-yet-implemented SQL-inspired query language for vector databases (Ch 9)

**Word2Vec** — Mikolov et al.'s 2013 model producing dense word vectors via skip-gram/CBOW training; the conceptual ancestor of modern embeddings (Ch 2)

**Zero-shot learning (via embeddings)** — embedding models generalize to unseen categories without task-specific fine-tuning, because semantic proximity alone drives classification (Ch 2)
