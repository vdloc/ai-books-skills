# Cheatsheet — Vector Databases: A Practical Introduction

## Decision rules

- **Choosing a data store**: if vectors are secondary to existing structured/semistructured infra and scale is moderate → NoSQL + vector extension (Redis, Mongo Atlas, Elasticsearch). If you need native SQL + vector ops at small/midsize scale → hybrid RDBMS+extension (Postgres+pgvector, SQLite+vss). If you need billion-vector scale as the primary workload → purpose-built vector DB.
- **Choosing an embedding model**: production/latency-sensitive → all-MiniLM-L6-v2 (384-dim, ~80MB, fast). Accuracy-critical → all-mpnet-base-v2 (768-dim, ~420MB). Multilingual → paraphrase-multilingual-mpnet-base-v2 (768-dim, 50+ languages).
- **Choosing a FAISS/pgvector index**: exact results needed, small data → Flat. Large-scale, approximate OK → IVF or HNSW. High-dimensional → HNSW. Tight memory → PQ-based. Max speed → HNSW, then IVF.
- **When search feels "too vague"**: increase keyword_weight in hybrid search fusion (default split 0.7 semantic / 0.3 keyword).
- **When aggregating vectors with expected outliers**: use geometric median or medoid, not centroid.
- **Always**: embed queries and documents with the *same* model. Tag every embedding with its generating model. Disable `enable_load_extension` immediately after loading needed SQLite extensions.

## Similarity threshold guide (pgvector cosine, `1 - (a <=> b)`)

| Score | Meaning | Action |
|---|---|---|
| 0.8+ | Very high — same specific method/paper | High-confidence context |
| 0.7 | Sweet spot — relevant but diverse | Default retrieval threshold |
| ≤ 0.6 | Hallucinated relevance (shared vocab, not concepts) | Exclude or flag low-confidence |

## Index selection matrix (FAISS)

| Constraint | Preferred index |
|---|---|
| Small dataset, exact results | Flat (IndexFlatL2 / IndexFlatIP) |
| Large-scale, approximate OK | IVF or HNSW |
| High-dimensional data | HNSW |
| Tight memory budget | PQ-based (IndexPQ, IndexIVFPQ) |
| Max search speed | HNSW > IVF |
| GPU available | GPU-accelerated variant |

## HNSW tuning defaults (pgvector)

| Param | Meaning | Book's default | Raise when |
|---|---|---|---|
| `m` | Links per graph node | 16 | Archive/dataset grows large, need better recall |
| `ef_construction` | Build-time search breadth | 64 | Same — targets >0.95 recall |

Rule of thumb: HNSW is good up to ~10M vectors for near-instantaneous search with moderate memory.

## Hybrid search score fusion formula

```
final_score = w1 * normalize(vector_score) + w2 * normalize(text_score)
w1 + w2 = 1.0, both nonnegative
default: w1=0.7 (semantic), w2=0.3 (keyword/BM25)
```
Normalize BM25 by dividing by max score in the current result set (BM25 is unbounded; cosine similarity is already [0,1]).

## Storage/schema trade-off table

| Approach | Native similarity ops | Scales to billions | Structured filtering | Fit |
|---|---|---|---|---|
| Plain RDBMS FLOAT[] column | No | No | Yes | Prototype only |
| NoSQL + vector extension | Partial | Struggles | Yes | Existing NoSQL infra, moderate scale |
| Hybrid RDBMS + vector ext. (pgvector/sqlite-vss) | Yes | Small/midsize | Yes | This book's default |
| Purpose-built vector DB | Yes (optimized) | Yes | Limited | Billion-vector, vector-primary |

## Tells & smells

- **"Same words, different meaning" search failures** (e.g. apple fruit vs. Apple Inc.) → keyword search limitation; needs semantic search.
- **Results share vocabulary but not topic** → similarity threshold too low (below ~0.6) or missing entirely.
- **Query embeds fine but results look random** → check documents and query used *different* embedding models.
- **Fused hybrid scores dominated by one signal** → forgot to normalize BM25 (unbounded) against cosine similarity (bounded [0,1]).
- **Vector DB migration pain when swapping embedding models** → no `embedding_model` provenance column was tracked.
- **A single retrieved chat message/chunk is confusing out of context** → missing symmetric context-window retrieval (no ordering column, or window not applied).
- **IVF search accuracy degrades unpredictably** → `nprobe` too low for the query load; it's a live speed/accuracy dial, tune per workload.
- **Centroid "representative" vector looks skewed** → outliers present; switch to geometric median or medoid.
