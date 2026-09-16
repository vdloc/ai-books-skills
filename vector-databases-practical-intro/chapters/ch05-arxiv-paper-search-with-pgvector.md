# Chapter 5: Building an ArXiv Paper Search System with PostgreSQL pgvector

## Core Idea
A production-grade semantic search system needs more than a vector column — it needs a full relational schema (papers, chunks, authors, categories, processing queue) combining pgvector's HNSW index with GIN/trigram indexes for hybrid semantic + metadata + fuzzy-text search, demonstrated end-to-end on ArXiv papers.

## Frameworks Introduced
- **Chunking with sliding-window overlap**: split documents into 512–1,024 token chunks (~1–3 paragraphs) with ~20% overlap between consecutive chunks.
  - When to use: any document long enough that a single embedding can't represent it faithfully (papers, long articles).
  - How: pick target_tokens (book uses 768) and overlap_tokens (128, ~20% of 640 effective step) so chunk boundaries don't silently cut off information relevant to a nearby query.
- **Processing-status flags as a state machine**: track `pdf_downloaded`, `pdf_processed`, `embedding_generated` booleans (plus a `processing_queue` table with status/priority/retry_count) per record — the "gears" that let async pipeline stages know what work remains.
  - When to use: any multi-stage ingestion pipeline (download → extract → chunk → embed) that must be resumable and inspectable.
- **HNSW parameter tuning (`m`, `ef_construction`)**: `m` = bidirectional links per node (index size vs. search quality trade-off); `ef_construction` = search breadth during index build (recall vs. build time trade-off). Book's values (m=16, ef_construction=64) target >0.95 recall for collections up to ~10M vectors.

## Key Concepts
- **pgvector `vector(384)` column type**: native Postgres column type for embeddings; supports `vector_cosine_ops`/`vector_l2_ops` operator classes for indexing.
- **GIN index**: used here for array-containment queries (`categories`, `authors` as `TEXT[]`) and, with `gin_trgm_ops`, for fuzzy/trigram text matching.
- **Hybrid search**: combining vector similarity (semantic) with structured/fuzzy filters (GIN, trigram, B-tree) in one query — e.g., "neural network papers by Yoshua Bengio published after 2020."
- **HNSW vs. IVFFlat (pgvector)**: HNSW gives logarithmic search complexity and near-instantaneous search at moderate scale (up to ~10M vectors) with acceptable memory overhead — the book's chosen default over IVFFlat for this use case.

## Mental Models
- Treat the **relational schema as the backbone**, with vector search as one capability layered on top — papers, chunks, authors, categories, search_history, and processing_queue are all ordinary relational tables; only `paper_chunks.embedding` and `search_history.query_embedding` are vector-typed.
- Use **processing-status flags/queues** to make a multi-stage ingestion pipeline restartable and observable — don't rely on the pipeline running start-to-finish in one pass.
- Pick **chunk size for retrieval relevance, not convenience**: too large loses precision (multiple ideas per chunk diluting the embedding); too small loses context (a chunk with no surrounding meaning). 512–1,024 tokens with 20% overlap is the book's tuned answer for academic papers specifically — recalibrate per document type.

## Anti-patterns
- **Storing only a flat vector column with no metadata schema**: makes hybrid queries (semantic + author + date + category) impossible without external joins or duplicated logic.
- **Chunking without overlap**: risks silently cutting relevant information at a chunk boundary, causing retrieval to miss content that spans two chunks.
- **Using IVFFlat by default at moderate scale**: the book explicitly prefers HNSW for up to ~10M vectors due to superior query performance, reserving IVFFlat consideration for different scale/memory trade-offs.
- **No processing-status tracking on ingestion records**: makes partial-failure recovery ("which papers still need embeddings?") a manual, error-prone audit instead of a `WHERE embedding_generated = FALSE` query.

## Reference Tables

| Table | Purpose | Key indexes |
|---|---|---|
| `papers` | ArXiv paper metadata, processing status flags | GIN on `categories`, `authors`; B-tree on `published_date` |
| `paper_chunks` | Chunked text + 384-dim embedding per chunk | HNSW (`vector_cosine_ops`, m=16, ef_construction=64); GIN trigram on `chunk_text` |
| `authors` | Normalized author records | GIN trigram on `name` (fuzzy author search) |
| `paper_authors` | Many-to-many papers↔authors | composite PK, position index |
| `categories` | ArXiv taxonomy | PK on `code` |
| `search_history` | Query log + query embedding | HNSW on `query_embedding` (enables "similar past queries") |
| `processing_queue` | Async pipeline task tracking | composite index on `(status, priority)` |

| HNSW param | Meaning | Book's value | Trade-off |
|---|---|---|---|
| `m` | Bidirectional links per graph node | 16 | Index size ↔ search quality |
| `ef_construction` | Search breadth at build time | 64 | Build time ↔ recall (targets >0.95) |

## Worked Example
Core chunk table with hybrid-ready indexing:
```sql
CREATE TABLE paper_chunks (
    id SERIAL PRIMARY KEY,
    paper_id INTEGER REFERENCES papers(id) ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    chunk_text TEXT NOT NULL,
    chunk_tokens INTEGER,
    embedding vector(384),              -- all-MiniLM-L6-v2
    section_name VARCHAR(255),
    has_math BOOLEAN DEFAULT FALSE,
    has_code BOOLEAN DEFAULT FALSE,
    UNIQUE (paper_id, chunk_index)
);

CREATE INDEX idx_chunks_embedding ON paper_chunks
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

CREATE INDEX idx_chunks_text_trgm ON paper_chunks
USING GIN (chunk_text gin_trgm_ops);   -- fuzzy text alongside semantic search
```
Chunking function signature (sliding window, 20% overlap):
```python
def calculate_chunk_parameters(text: str,
                                target_tokens: int = 768,
                                overlap_tokens: int = 128) -> List[dict]:
    """Calculate optimal chunking parameters for text."""
```
Papers table tracks pipeline state so ingestion is resumable:
```sql
pdf_downloaded BOOLEAN DEFAULT FALSE,
pdf_processed BOOLEAN DEFAULT FALSE,
embedding_generated BOOLEAN DEFAULT FALSE,
processing_error TEXT
```
The full system is packaged with Docker Compose for reproducible local deployment and includes an interactive CLI for querying.

## Key Takeaways
1. A real vector search system is a full relational schema with one (or a few) vector-typed columns — not a standalone vector store.
2. Chunk size and overlap are tunable relevance parameters, not implementation details — 512–1,024 tokens / 20% overlap is a good academic-paper starting point.
3. HNSW (m=16, ef_construction=64) is the book's default for up to ~10M vectors, chosen over IVFFlat for superior query performance at this scale.
4. Combine GIN (array containment, trigram fuzzy text) with HNSW (semantic) indexes to support true hybrid queries.
5. Track ingestion pipeline state explicitly (boolean flags + a processing_queue table) so multi-stage pipelines are resumable and auditable.
6. Store query embeddings too (`search_history.query_embedding`) — enables analyzing or reusing past queries semantically, not just logging text.

## Connects To
- **Ch 3**: HNSW parameters (m, ef_construction) here are the same FAISS/HNSW concepts introduced there, now exposed through pgvector's SQL interface.
- **Ch 2**: uses all-MiniLM-L6-v2 (384-dim) as established in the embeddings chapter.
- **Ch 6, Ch 7**: extend this same pgvector pattern into full RAG systems (Ollama, scientific RAG).
