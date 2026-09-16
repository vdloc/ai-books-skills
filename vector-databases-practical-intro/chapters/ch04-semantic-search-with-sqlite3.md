# Chapter 4: Semantic Search with SQLite3

## Core Idea
The `sqlite-vss` extension turns SQLite into a hybrid vector database (FAISS under the hood, exposed as a virtual table), letting you combine vector similarity search with ordinary SQL metadata filtering in a single lightweight, file-based system — demonstrated end-to-end via a Reddit knowledge-base search engine.

## Frameworks Introduced
- **Overfetch-then-filter pattern**: run the vector search for `limit × candidate_multiplier` candidates first, then apply SQL metadata filters (subreddit, score, date) on that candidate set in a second step, rather than pushing filters into the nearest-neighbor search itself.
  - When to use: whenever the vector index's native filter pushdown is unreliable or unsupported (true of many sqlite-vss/FAISS-backed setups) — improves result stability at the cost of some overfetch overhead.
  - How: pick a `candidate_multiplier` (book uses 10×), run `ORDER BY distance ASC LIMIT k` where `k = max(limit*multiplier, limit)`, join to metadata table, then apply `WHERE` filters, then truncate to `limit`.
- **Model-tagged embeddings**: store an `embedding_model` column alongside each vector so that switching embedding models later lets you identify and regenerate only affected rows instead of a full re-embed.

## Key Concepts
- **sqlite-vss**: SQLite extension pair (`vector0` + `vss0`) providing a virtual table type for vector similarity search, backed internally by FAISS.
- **Virtual table (`vss0`)**: created with `CREATE VIRTUAL TABLE posts_vss USING vss0(embedding(dimension))`; indexes vectors keyed by SQLite `rowid` for joining back to content tables.
- **`INTEGER PRIMARY KEY` as rowid alias**: gives a stable, explicit ID that the VSS index joins against — must be preserved across updates.
- **WAL journal mode**: `PRAGMA journal_mode=WAL` — recommended operational pragma for concurrent read/write access patterns typical of a search index.
- **Default index type**: sqlite-vss defaults to a "Flat" (exact, brute-force) index; approximate index types are selectable via FAISS factory strings like `"IVF100,Flat"` or `"HNSW16"` for larger datasets (connects directly to Ch 3's FAISS index catalog).

## Mental Models
- Treat the **embedding BLOB column as redundant-but-valuable**: it duplicates what's in the VSS index, but lets you rebuild the index (e.g., switching Flat→IVF) without regenerating embeddings from the source text.
- Use **metadata indexes (subreddit, created_utc, score) alongside the vector index** — SQLite's query planner can use them to shrink the candidate set either before or after the vector search depending on query shape; don't rely on the vector index alone for filtering.
- Think of sqlite-vss as **"FAISS with a SQL front door"** — you get FAISS's index types (Ch 3) but interact with them via ordinary SQL joins and virtual tables instead of the FAISS Python API directly.

## Anti-patterns
- **Pushing filters directly into `vss_search`**: many sqlite-vss versions don't reliably push predicate filters into the nearest-neighbor search itself — results can be unstable; overfetch-then-filter in SQL is the safer pattern.
- **Not tracking which embedding model produced each vector**: makes migrating to a new embedding model an all-or-nothing re-embed instead of a targeted, incremental update.
- **Leaving `enable_load_extension(True)` on after loading `vss0`/`vector0`**: the book explicitly disables it immediately after (`conn.enable_load_extension(False)`) as a security measure — leaving it enabled is a code-injection surface.

## Reference Tables

| Schema element | Purpose |
|---|---|
| `posts.id` (INTEGER PRIMARY KEY) | Stable rowid, joined against VSS index |
| `posts.embedding` (BLOB) | Redundant vector copy — enables index rebuild without re-embedding |
| `posts.embedding_model` (TEXT) | Tracks provenance — enables targeted re-embedding on model change |
| `idx_posts_subreddit/created/score` | Metadata indexes for post-vector-search filtering |
| `posts_vss` (VIRTUAL TABLE, vss0) | FAISS-backed vector index, joined by rowid |

## Worked Example
End-to-end: Reddit knowledge base with semantic search.

Schema + index setup:
```python
conn = sqlite3.connect(db_path)
conn.enable_load_extension(True)
conn.load_extension(f"{extension_path}/vector0")
conn.load_extension(f"{extension_path}/vss0")
conn.enable_load_extension(False)  # disable immediately — security

conn.execute("PRAGMA journal_mode=WAL;")
conn.execute("PRAGMA synchronous=NORMAL;")
conn.execute("PRAGMA foreign_keys=ON;")

conn.execute("""
    CREATE TABLE IF NOT EXISTS posts (
        id INTEGER PRIMARY KEY, post_id TEXT UNIQUE NOT NULL,
        title TEXT NOT NULL, selftext TEXT, url TEXT,
        subreddit TEXT NOT NULL, author TEXT,
        score INTEGER DEFAULT 0, num_comments INTEGER DEFAULT 0,
        created_utc INTEGER NOT NULL,
        embedding BLOB, embedding_model TEXT,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
""")
# + metadata indexes on subreddit, created_utc, score

conn.execute(f"CREATE VIRTUAL TABLE posts_vss USING vss0(embedding({dimension}))")
```

Search with overfetch-then-filter:
```python
k = max(limit * candidate_multiplier, limit)  # candidate_multiplier default 10

sql = """
    WITH candidates AS (
        SELECT rowid, distance FROM posts_vss
        WHERE vss_search(embedding, vector_from_json(?))
        ORDER BY distance ASC LIMIT ?
    )
    SELECT p.id, p.post_id, p.title, p.selftext, p.subreddit, p.author,
           p.score, p.num_comments, p.created_utc, c.distance
    FROM candidates c INNER JOIN posts p ON p.id = c.rowid
    WHERE 1=1
"""
# ... then dynamically append " AND p.subreddit IN (...)", " AND p.score >= ?", date filters
```
Note: raw L2 distance from sqlite-vss is unnormalized — the book wraps it in a `similarity_score` property for display rather than showing raw distance to users.

## Key Takeaways
1. sqlite-vss gives file-based hybrid vector+SQL search without standing up a separate vector DB server — good for small/midsize, single-node use cases.
2. Overfetch-then-filter (candidate_multiplier ~10×) is the reliable pattern when the vector extension's own filter pushdown is unstable.
3. Tag every embedding with its generating model — this is cheap insurance against costly full re-embeds later.
4. sqlite-vss's default Flat index is exact; switch to `"IVFx,Flat"` or `"HNSWx"` factory strings (per Ch 3) as data volume grows.
5. Always disable `enable_load_extension` immediately after loading needed extensions — a security hygiene habit, not just a style choice.
6. Raw vector distance is not a user-facing quality signal — normalize/wrap it before display.

## Connects To
- **Ch 3**: sqlite-vss's index types (Flat, IVF, HNSW) are the exact FAISS index types cataloged there.
- **Ch 2**: uses all-MiniLM-L6-v2 (384-dim) as the embedding generator feeding this schema.
- **Ch 5, Ch 7**: the pgvector chapters build a more scalable version of this same hybrid pattern on PostgreSQL instead of SQLite.
