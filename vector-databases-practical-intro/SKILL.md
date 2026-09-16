---
name: vector-databases-practical-intro
description: "Knowledge base from \"Vector Databases: A Practical Introduction\" by Nitin Borwankar. Use when applying vector similarity search, embeddings, FAISS indexing, pgvector/sqlite-vss hybrid architectures, hybrid search score fusion, or building RAG/semantic-search systems, studying the book, or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Vector Databases: A Practical Introduction
**Author**: Nitin Borwankar | **Pages**: ~293 | **Chapters**: 9 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `hybrid search`, `HNSW tuning`, `chunking`, `similarity threshold`, or another indexed topic; the relevant chapter is read on demand
- **With a chapter** — ask for `ch05`; the specific chapter is loaded
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, the relevant chapter file is read before answering.

---

## Core Frameworks & Mental Models

**Hybrid Database Architecture** — pair a vector component (similarity search/indexing) with a metadata component (relational/NoSQL) via an integration layer. Use this whenever a system needs both semantic retrieval and structured filtering/sorting — which is nearly every real system. Pure vector databases are for billion-vector, vector-primary scale only; the book's own default stack throughout is PostgreSQL+pgvector or SQLite+sqlite-vss at small/midsize scale.

**Semantic vs. Keyword Search** — keyword search finds pages containing given words (syntax); semantic search finds pages that answer the query regardless of wording (meaning). Neither subsumes the other — combine them (see Hybrid Search Fusion below) rather than picking one.

**Embedding Model Selection** — match the model to the constraint: all-MiniLM-L6-v2 (384-dim, ~80MB, fast) for production/latency-sensitive work; all-mpnet-base-v2 (768-dim) when accuracy is critical; paraphrase-multilingual-mpnet-base-v2 for cross-lingual needs. Always use the *same* model for documents and queries — mismatched models silently break similarity search, the single most common integration bug.

**FAISS/pgvector Index Selection** — no single "best" index; choose by dataset size (small→flat, large→IVF/HNSW), dimensionality (high→HNSW), accuracy needs (exact→flat), memory (tight→PQ), speed (max→HNSW then IVF), hardware (GPU→GPU variants). `nprobe` (IVF) and HNSW's `m`/`ef_construction` are the live tuning knobs — `m=16, ef_construction=64` is the book's default for up to ~10M vectors.

**Hybrid Search Score Fusion** — `final_score = w1*normalize(vector_score) + w2*normalize(text_score)`, weights summing to 1.0, default 0.7 semantic / 0.3 keyword (BM25). Critical: BM25 is unbounded, cosine similarity is bounded [0,1] — normalize (divide by max score in result set) before combining, or one signal silently dominates. Raise keyword weight if results feel too vague.

**Similarity Threshold Calibration** — for pgvector cosine similarity on text: 0.8+ = very high confidence (same specific content), 0.7 = the sweet spot for relevant-but-diverse results (book's default), ≤0.6 = "hallucinated relevance" (shared vocabulary, not shared concepts) — filter these out rather than returning top-k unconditionally.

**Chunking with Sliding-Window Overlap** — 512–1,024 token chunks with ~20% overlap for long documents (academic-paper tuning; recalibrate per domain). Prevents information loss at chunk boundaries.

**Multilevel Semantic Search** — for domains with both summary and detail content (abstracts+sections), run parallel searches at each granularity, merge by similarity, sort, take top-k. Covers both discovery-type and detail-type user queries without requiring the user to specify intent.

**Model-Tagged Embeddings & Processing-Status Tracking** — store which embedding model generated each vector, and track multi-stage ingestion pipeline state (download/process/embed flags, or a processing_queue table) explicitly. Both are cheap insurance: the first avoids all-or-nothing re-embeds on model swap; the second makes partial pipeline failures recoverable via a query instead of a manual audit.

**Robust Aggregation** — centroid (mean, closed-form, `argmin Σ‖x−vᵢ‖²`) is easy but outlier-sensitive; geometric median (`argmin Σ‖x−vᵢ‖`, iterative) and medoid (an actual data point) are robust alternatives when outliers are expected in a vector cluster.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-introduction-to-vector-databases.md) | Introduction to Vector Databases | Hybrid DB architecture, semantic vs. keyword search, vector arithmetic |
| [ch02](chapters/ch02-embeddings.md) | Embeddings | Word2Vec, transformer families, sentence-transformers, embedding model selection |
| [ch03](chapters/ch03-similarity-search-with-faiss.md) | Similarity Search with FAISS | Index selection heuristic, Flat/IVF/LSH/PQ/HNSW, nprobe tuning |
| [ch04](chapters/ch04-semantic-search-with-sqlite3.md) | Semantic Search with SQLite3 | sqlite-vss, overfetch-then-filter, model-tagged embeddings |
| [ch05](chapters/ch05-arxiv-paper-search-with-pgvector.md) | Building an ArXiv Paper Search System with PostgreSQL pgvector | Chunking w/ overlap, HNSW tuning (m, ef_construction), processing-status tracking |
| [ch06](chapters/ch06-rag-with-sqlite-vss-and-ollama.md) | Building a RAG System with SQLite VSS and Ollama | Hybrid search score fusion, BM25 normalization |
| [ch07](chapters/ch07-scientific-rag-with-postgresql-pgvector.md) | Building a Scientific RAG System with PostgreSQL and pgvector | Multilevel semantic search, similarity threshold calibration |
| [ch08](chapters/ch08-complete-conversation-search-and-rag-system.md) | Building a Complete Conversation Search and RAG System | Three-table normalization, symmetric context-window retrieval |
| [ch09](chapters/ch09-vector-query-language.md) | Vector Query Language | VQL data model (proposal), centroid vs. geometric median |

## Topic Index

- **BM25 / FTS5** → ch06
- **Centroid / geometric median / medoid** → ch09
- **Chunking (sliding window, overlap)** → ch05
- **Context window (conversation retrieval)** → ch08
- **Distance metrics (L2, cosine, inner product)** → ch01, ch03
- **Embedding models (selection, comparison)** → ch02
- **FAISS index types** → ch03
- **Hybrid database architecture** → ch01
- **Hybrid search / score fusion** → ch06, ch09
- **HNSW tuning (m, ef_construction)** → ch03, ch05, ch08
- **Multilevel semantic search** → ch07
- **Overfetch-then-filter pattern** → ch04, ch06
- **pgvector** → ch05, ch07, ch08
- **Processing-status / ingestion pipeline state** → ch05, ch08
- **RAG pipeline (end-to-end)** → ch02, ch06, ch07, ch08
- **Similarity threshold calibration** → ch07
- **sqlite-vss** → ch04, ch06
- **Three-table normalization** → ch08
- **Vector Query Language (VQL)** → ch09
- **Word2Vec / transformers** → ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills or ask the agent directly.
