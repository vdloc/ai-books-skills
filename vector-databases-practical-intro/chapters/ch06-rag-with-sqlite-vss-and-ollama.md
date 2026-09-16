# Chapter 6: Building a Retrieval-Augmented Generation System with SQLite VSS and Ollama

## Core Idea
A robust RAG system fuses **semantic** search (vector similarity) with **keyword** search (BM25/FTS5) via weighted score normalization, then feeds the fused top-k results plus the query to a local LLM (Ollama) — hybrid retrieval beats either search mode alone.

## Frameworks Introduced
- **Hybrid Search with Score Fusion**: `hybrid_score = semantic_weight × normalized_semantic_score + keyword_weight × normalized_keyword_score`, with default `semantic_weight=0.7`, `keyword_weight=0.3`.
  - When to use: whenever pure semantic search misses exact technical terms/names, or pure keyword search misses conceptually-related-but-differently-worded content — i.e., almost every production RAG system.
  - How: (1) run semantic search, convert cosine distance → similarity via `1 - distance`; (2) run keyword search via BM25 (FTS5), scores are unbounded and often negative in SQLite so invert sign; (3) normalize keyword scores by dividing by the max score in the current result set (bounds them to comparable [0,1]-ish range against cosine similarity); (4) merge by ID with weighted sum; (5) sort descending, take top `limit`.
  - Tuning rule: "if search is too vague, increase keyword_weight" — the 70/30 split is a starting point, not a fixed constant.
- **Overfetch-for-fusion**: fetch `limit * 2` candidates from *each* search method before fusing, so that even if semantic and keyword search return disjoint document sets, there's enough overlap/depth to produce a meaningful merged top-k.

## Key Concepts
- **BM25 (Best Matching 25)**: probabilistic ranking function used by FTS5; unlike raw term frequency, it models saturation (mentioning a word 100× isn't 100× more relevant) and document-length normalization (rewards concise, on-topic documents).
- **Cosine distance vs. cosine similarity**: SQLite VSS returns *distance* (0 = identical); convert to similarity via `1 - distance` before combining with other "higher is better" scores.
- **Score dominance pitfall**: BM25 scores are unbounded (often 0–20+) while cosine similarity is bounded [0,1] — combining them without normalization lets BM25 numerically dominate the fused score regardless of intended weighting.
- **FTS5**: SQLite's full-text search virtual table extension, providing the `bm25()` ranking function used for the keyword component.

## Mental Models
- Treat **semantic and keyword search as complementary signals, not competitors** — semantic captures paraphrase/meaning, keyword captures exact technical terms/names/identifiers that embeddings can blur.
- Think of the **0.7/0.3 semantic/keyword split as a dial, not a law**: raise keyword_weight when results feel "too vague" (semantic search returning topically-related-but-imprecise matches); raise semantic_weight when users phrase queries very differently from document wording.
- Use a **try/except around the keyword search call** — a missing/unpopulated FTS5 index shouldn't crash hybrid search; degrade gracefully to semantic-only results.

## Anti-patterns
- **Combining BM25 and cosine similarity scores without normalization**: unbounded BM25 values swamp the bounded cosine similarity signal, silently defeating the intended weighting.
- **Fetching only `limit` candidates per search method before fusing**: if semantic and keyword result sets barely overlap, you can end up with fewer than `limit` usable fused results — overfetch (`limit * 2`) first.
- **Treating BM25 score sign literally**: SQLite's `bm25()` returns typically-negative values (lower = better match) — must invert before treating "higher is better" in fusion.
- **No fallback when FTS5 index is unpopulated**: raises `sqlite3.OperationalError` — must be caught, not left to crash the whole RAG pipeline.

## Reference Tables

| Component | Score direction (raw) | Normalization applied | Weight (default) |
|---|---|---|---|
| Semantic (VSS cosine distance) | 0 = identical (lower better) | `1 - distance` → similarity [0,1] | 0.7 |
| Keyword (FTS5 BM25) | Negative, unbounded | Invert sign, divide by max in result set | 0.3 |

**System architecture** (5 layers): Vector DB layer (SQLite VSS) → Embedding engine (SentenceTransformers) → Hybrid search system → LLM integration (Ollama) → RAG pipeline orchestrator.

**Known limitations flagged by the book** (useful anti-scope checklist): no comment threads/full post metadata/media/live ingestion; no semantic chunking, query expansion, cross-encoder reranking, or metadata filtering; risk of dimensionality mismatch, duplicate insertions, deletion orphans if not handled explicitly.

## Worked Example
```python
def hybrid_search(query_text, query_vector, limit=5, semantic_weight=0.7):
    sem_res = get_semantic_results(query_vector, limit)
    key_res = get_keyword_results(query_text, limit)

    # Normalize keyword scores against the max in this result set
    max_key = max([r['score'] for r in key_res]) if key_res else 1.0
    for r in key_res:
        r['score'] /= max_key

    # Weighted merge: (Semantic * 0.7) + (Keyword * 0.3)
    merged = {}
    for r in sem_res:
        merged[r['id']] = r['score'] * semantic_weight
    for r in key_res:
        merged[r['id']] = merged.get(r['id'], 0) + (r['score'] * (1 - semantic_weight))

    sorted_ids = sorted(merged.items(), key=lambda x: x[1], reverse=True)[:limit]
    return [id for id, score in sorted_ids]

def get_semantic_results(query_vector, limit=5):
    cursor.execute("""
        SELECT rowid, vss_cosine_distance(embedding, ?) as distance
        FROM vss_posts WHERE vss_search(embedding, ?)
        ORDER BY distance ASC LIMIT ?
    """, (query_vector, query_vector, limit * 2))
    return [{"id": r[0], "score": 1 - r[1]} for r in cursor.fetchall()]  # distance → similarity

def get_keyword_results(query_text, limit=5):
    try:
        cursor.execute(
            "SELECT rowid, bm25(posts_fts) FROM posts_fts WHERE posts_fts MATCH ? LIMIT ?",
            (query_text, limit * 2)
        )
        return [{"id": r[0], "score": -r[1]} for r in cursor.fetchall()]  # invert BM25 sign
    except sqlite3.OperationalError as e:
        print(f"Warning: Keyword search failed. Ensure FTS5 index is populated. Error: {e}")
        return []
```
LLM step: format fused top-k chunks as context, send query + context to a local Ollama model for generation — kept simple/robust per the book (health-check function before calling, straightforward prompt formatting).

## Key Takeaways
1. Hybrid search (semantic + keyword, weighted fusion) outperforms either method alone — this is the chapter's central, reusable pattern.
2. Always normalize scores from different ranking systems (bounded cosine similarity vs. unbounded BM25) before combining them — otherwise one signal dominates unintentionally.
3. Overfetch (2× limit) from each search method before fusing, so disjoint result sets still yield a full top-k.
4. Treat the semantic/keyword weight split as tunable per use case, not fixed — 0.7/0.3 is a documented starting point.
5. Handle keyword search failures gracefully (missing/empty FTS5 index) rather than crashing the pipeline.
6. Local LLM inference (Ollama) trades some capability for privacy and zero API cost — a legitimate default for certain deployments.

## Connects To
- **Ch 4**: builds directly on the SQLite VSS schema and overfetch pattern established there, adding FTS5 keyword search alongside it.
- **Ch 7**: reimplements this hybrid RAG pattern on PostgreSQL/pgvector for a "Scientific RAG System" at larger scale.
- **Reranking (forward reference)**: the book explicitly flags cross-encoder reranking as a future enhancement beyond this chapter's score-fusion approach.
