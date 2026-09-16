# Chapter 7: Building a Scientific RAG System with PostgreSQL and pgvector

## Core Idea
Scientific queries need **multilevel semantic search** — searching abstracts (paper-level discovery) and sections (technical-detail retrieval) in parallel, merging by similarity — plus a calibrated similarity threshold to avoid pulling in vocabulary-similar-but-conceptually-unrelated papers.

## Frameworks Introduced
- **Multilevel Semantic Search**: run abstract-level and section-level searches in parallel from one query embedding, merge results, sort by similarity descending, take the combined top-k.
  - When to use: any domain where documents have both a high-level summary and detailed substructure (papers/abstracts+sections, docs/overviews+API-detail-pages, etc.) and users ask both discovery-type and detail-type questions.
  - How: sequence is query → embed (384-dim) → branch to abstract search (e.g. top 3) + section search (e.g. top 5) → merge (8 total) → sort by similarity DESC → take top 5 as context.
- **Similarity Threshold Calibration** (pgvector cosine, via `1 - (embedding <=> query)`):
  - **0.8+**: very high similarity — likely the same specific method or paper.
  - **0.7**: "sweet spot" for relevant-but-diverse scientific papers — the book's default threshold.
  - **≤0.6**: high risk of "hallucinated relevance" — shared vocabulary but not shared concepts.
  - Preserve these exact numeric bands — they're a directly reusable decision rule for any pgvector cosine-similarity RAG system.

## Key Concepts
- **`<=>` operator (pgvector)**: computes cosine distance, range [0, 2], where 0 = identical. Convert via `1 - distance` for an intuitive [−1, 1]-ish similarity score (in practice mostly [0,1] for normalized text embeddings).
- **Abstract-level vs. section-level search**: abstract search is precise but coarse (good for "which papers are relevant"); section search is granular but higher-volume (good for "what does this paper say about X specifically").
- **Threshold as precision filter**: filtering results below a similarity threshold before ranking, not just limiting count — improves precision by excluding marginal matches outright rather than just deprioritizing them.

## Mental Models
- Treat **abstract search and section search as two different retrieval intents**, not redundant duplicate searches — discovery queries favor abstract search, detail queries favor section search; running both and merging covers both intents without the user having to specify which they meant.
- Use the **0.7 similarity threshold as a default guardrail, not a universal constant** — recalibrate per embedding model and domain (the book's numbers assume 384-dim all-MiniLM-L6-v2 style embeddings on scientific text); below ~0.6, treat retrieved results as suspect.
- Think of a **JOIN back to the parent table (papers) from section results** as mandatory context, not decoration — a section without its paper title is hard for a downstream LLM (or user) to ground.

## Anti-patterns
- **Single-level search only (sections OR abstracts, never both)**: misses either the "which paper" discovery use case or the "what exactly does it say" detail use case, depending on which level you skipped.
- **No similarity threshold (returning top-k regardless of score)**: guarantees returning *something* even when nothing is actually relevant — the LLM then generates from irrelevant context, a direct contributor to hallucination.
- **Trusting similarity scores below ~0.6 as relevant**: the book explicitly calls this range "hallucinated relevance" — same vocabulary, different concepts.

## Reference Tables

| Similarity (1 − cosine distance) | Interpretation | Action |
|---|---|---|
| 0.8+ | Very high — same specific method/paper | High-confidence context |
| 0.7 | Sweet spot — relevant but diverse | Default retrieval threshold |
| 0.6 or lower | Hallucinated relevance risk (shared vocab, not concepts) | Exclude or flag as low-confidence |

| Search level | Table | Granularity | Best for |
|---|---|---|---|
| Abstract | `papers.abstract_embedding` | Whole-paper | Discovery: "which papers are relevant" |
| Section | `paper_sections.section_embedding` | Sub-document | Detail: "what does this say about X" |

Evaluation dimensions the book names for judging RAG output quality: **retrieval depth**, **context relevance**, **faithfulness** — use as a lightweight rubric when reviewing RAG answers.

## Worked Example
Abstract-level search with threshold filtering:
```python
def search_papers_by_abstract(db_config, query, limit=3, threshold=0.7):
    conn = psycopg2.connect(**db_config)
    cursor = conn.cursor()
    query_embedding = generate_embedding(query)

    cursor.execute("""
        SELECT paper_id, title, abstract, authors,
               1 - (abstract_embedding <=> %s::vector) as similarity
        FROM papers
        WHERE abstract_embedding IS NOT NULL
          AND 1 - (abstract_embedding <=> %s::vector) > %s
        ORDER BY similarity DESC
        LIMIT %s
    """, (query_embedding.tolist(), query_embedding.tolist(), threshold, limit))

    return [{'paper_id': r[0], 'title': r[1], 'abstract': r[2],
             'authors': r[3], 'similarity': r[4]} for r in cursor.fetchall()]
```
Section-level search, joined back to parent paper for context:
```python
def search_paper_sections(db_config, query, limit=5, threshold=0.7):
    ...
    cursor.execute("""
        SELECT ps.paper_id, p.title, ps.section_number, ps.content,
               1 - (ps.section_embedding <=> %s::vector) as similarity
        FROM paper_sections ps
        JOIN papers p ON ps.paper_id = p.paper_id
        WHERE ps.section_embedding IS NOT NULL
          AND 1 - (ps.section_embedding <=> %s::vector) > %s
        ORDER BY similarity DESC
        LIMIT %s
    """, (query_embedding.tolist(), query_embedding.tolist(), threshold, limit))
    ...
```
Pipeline sequence: query → embed → parallel abstract (top 3) + section (top 5) search → merge (8) → sort by similarity DESC → top 5 → format as context → local LLM (Ollama, e.g. `llama3.1:8b`) generates grounded answer.

## Key Takeaways
1. Run abstract-level and section-level (or equivalent coarse/fine) search in parallel and merge — don't pick just one granularity.
2. Calibrate and enforce a similarity threshold (book's default 0.7 for pgvector cosine) — don't return top-k unconditionally.
3. Below ~0.6 similarity, treat matches as vocabulary-coincidence, not conceptual relevance.
4. Always JOIN detail-level results back to their parent record for citation/context — a bare section/chunk without provenance is less useful downstream.
5. Evaluate RAG output on retrieval depth, context relevance, and faithfulness as three distinct dimensions.

## Connects To
- **Ch 5**: shares the ArXiv/PostgreSQL/pgvector foundation, extended here with abstract-level embeddings and multilevel search.
- **Ch 6**: shares the Ollama local-LLM integration pattern; this chapter adds threshold-based precision filtering on top of Ch 6's hybrid-search fusion.
- **Ch 8**: multimodal RAG extends "what counts as a searchable chunk" beyond text sections to tables/images.
