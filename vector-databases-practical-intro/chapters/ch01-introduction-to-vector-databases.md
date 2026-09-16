# Chapter 1: Introduction to Vector Databases

## Core Idea
Vector databases add a semantic data type — the vector — to database systems, enabling similarity/semantic search (finding meaning-similar content) instead of only syntax/keyword matching; most practical systems are **hybrid**, pairing a vector component with a traditional relational metadata store.

## Frameworks Introduced
- **Hybrid Database Architecture**: vector component (similarity search, e.g. HNSW/LSH indexing) + metadata component (traditional RDBMS/NoSQL) + integration layer (unified query interface).
  - When to use: any real system needing both semantic retrieval and structured filtering/sorting (dates, categories, access control) — i.e., almost all production RAG/search systems.
  - How: keep vectors and business metadata in separate, purpose-optimized stores joined by a foreign key; query engine combines a similarity-ranked candidate set with SQL-style predicate filtering.
- **Keyword vs. Semantic Search distinction**: keyword search = "find pages containing these words" (syntax); semantic search = "find pages that answer this, regardless of wording" (meaning). Preserve this exact framing — it's the book's core justification for vector DBs.

## Key Concepts
- **Vector (as data type)**: a list of floats produced by embedding, supporting native similarity operations (unlike a BLOB, which is opaque).
- **Embedding**: (verb) the process of mapping unstructured data to a vector space; (noun) the resulting vector or vector set. Analogy: painting (verb) produces a painting (noun).
- **Cosine similarity**: cosine of the angle between two vectors; near 1 = similar meaning, low = dissimilar.
- **Nearest-neighbor search / ANN**: finding vectors close to a query vector; Approximate Nearest Neighbors (ANN) trades exactness for speed — acceptable because GenAI applications are inherently probabilistic.
- **Vector arithmetic**: semantic relationships expressible as vector math, e.g. `Vec("Queen") ≈ Vec("King") − Vec("Man") + Vec("Woman")` (Mikolov et al. 2013).
- **HNSW / LSH**: hierarchical navigable small world graphs / locality-sensitive hashing — indexing techniques that make high-dimensional similarity search fast and scalable.

## Mental Models
- Think of vector databases as **the semantic counterpart to relational databases**: relational DBs are the foundational tech for structured business data (accounting arithmetic — tabular, CRUD, joins); vector DBs are the foundational tech for unstructured semantic data.
- Use **NoSQL-with-vector-extensions** (Redis RediSearch, MongoDB Atlas, Elasticsearch, Cassandra+plugin) when vectors are a secondary concern layered onto existing structured/semi-structured infrastructure at moderate scale — not when vector search is the primary workload at billion-vector scale.
- Use **pure/specialized vector databases** when scale exceeds what a hybrid RDBMS+vector-extension setup (like Postgres+pgvector) can handle — i.e., billions of vectors.

## Anti-patterns
- **Storing vectors as plain FLOAT[] arrays in a vanilla RDBMS column**: works technically but has no native similarity/ANN operators, degrades badly as dimensionality and volume grow, and forces application-side distance computation.
- **Storing metadata as vectors**: wastes storage, loses filtering/sorting flexibility, and loses relational integrity features (constraints, transactions) that structured metadata needs.
- **Assuming keyword search is "good enough"**: literal matching misses semantically equivalent but differently worded queries (e.g., "get my money back" vs. "refund policy").

## Reference Tables

| Approach | Native similarity ops | Scales to billions | Structured filtering | Best fit |
|---|---|---|---|---|
| Plain RDBMS (FLOAT[] column) | No | No | Yes | Prototyping only |
| NoSQL + vector extension (Redis, Mongo Atlas, Elasticsearch, Cassandra) | Partial | Struggles at extreme scale | Yes (native) | Existing NoSQL infra, vectors secondary, moderate scale |
| Hybrid RDBMS + vector extension (Postgres + pgvector) | Yes | Small/midsize (dept-level) | Yes (native SQL) | This book's default — small/midsize enterprise apps |
| Purpose-built vector DB | Yes (optimized) | Yes (billions) | Limited/bolt-on | Billion-vector scale, vector-primary workloads |

## Worked Example
Query: *"How do I get my money back?"* against a company knowledge base.
- Keyword search: matches literal "money"/"back" — may miss pages titled "Refund policy," "Return and exchange," "Reimbursement," or return false positives like "cash-back rewards."
- Semantic search: embeds the query, finds nearest vectors to pages like "Refunds and Returns" or "How to request a refund after purchase" — correct match despite zero literal word overlap.
- Illustrative hybrid query combining both similarity and metadata filtering:
```sql
FIND TOP 10 SIMILAR VECTORS TO [0.1, 0.2, ..., 0.5]
WHERE metadata.category = 'science'
AND metadata.publish_date > '2023-01-01'
ORDER BY similarity DESC, metadata.views DESC
```
(Informal syntax — illustrates the shape of vector-similarity + SQL-style filter/sort combined, not a real product's syntax.)

## Key Takeaways
1. Vector databases exist because SQL matches syntax, not semantics — the vector data type plus similarity search closes that gap.
2. Embedding is both a verb (the mapping process) and a noun (the resulting vector/vector set) — keep the distinction straight when discussing pipelines.
3. ANN (approximate nearest neighbor), not exact search, is the right target for GenAI applications — exactness is unnecessary and can even hurt generation flexibility.
4. Most real systems need a **hybrid** architecture: vectors for semantics, relational/NoSQL metadata for structured filtering — pick pure vector DBs only at extreme (billion-vector) scale.
5. This book's default stack is PostgreSQL + pgvector, chosen for enterprise department-level scale, not billion-vector scale.

## Connects To
- **Ch 2**: goes deeper into embeddings — how vectors are actually produced (Word2vec and beyond).
- **Ch 5, Ch 7**: build on the Postgres+pgvector hybrid architecture introduced here for real RAG systems.
- **RAG (external concept)**: vector similarity search is the retrieval half of retrieval-augmented generation pipelines.
