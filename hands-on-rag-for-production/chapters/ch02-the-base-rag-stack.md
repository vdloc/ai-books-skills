# Chapter 2: The Base RAG Stack

## Core Idea
Every RAG system decomposes into two flows — ingestion (parse → chunk → embed → index) and query (rewrite → retrieve → rerank → generate) — and each layer has real trade-offs that determine end-to-end quality, latency, and cost.

## Frameworks Introduced
- **Four-step ingestion pipeline**: parsing → chunking → embedding → indexing.
  - When to use: for any document source that isn't already a clean structured DB record.
  - How: parse to strip non-knowledge content (formatting, coordinates) → chunk to fit context windows and focus embeddings on single topics → embed to enable semantic matching → index (vector DB + optional inverted index) for fast lookup.
- **Query rewriting**: transforming the raw user query into a retrieval query before searching.
  - When to use: whenever the user query mixes a command ("write a poem") with a context need ("about my 2025 travels") — searching on the full raw query pollutes retrieval with irrelevant hits.
  - How: split intent from context; use only the context portion (or a rewritten form of it) as the retrieval query.
- **Semantic search vs. keyword search vs. hybrid search**: dense vector similarity (semantic) captures meaning but misses exact/rare tokens; sparse/lexical (TF-IDF, BM25) captures exact matches but misses paraphrase; hybrid combines both. Base stack (this chapter) = pure semantic; hybrid is Chapter 3.
- **Reranking as a second-pass filter**: initial retrieval (embedding similarity) is necessary but not sufficient — apply MMR (maximum marginal relevance, 1998) to cut redundancy, or a cross-attention transformer reranker to jointly score query+chunk pairs.
- **ANN vs. ENN**: exact nearest neighbor (brute-force, O(n), a.k.a. FlatIndex search) doesn't scale; approximate nearest neighbor (ANN, e.g. HNSW) trades a small accuracy loss for massive speedup at scale.

## Key Concepts
- **Parsing**: separating knowledge-bearing text from formatting/layout noise in a source file (PDF, DOCX, HTML, etc.).
- **Chunking**: splitting parsed text into smaller pieces sized for LLM/embedding context windows and topic focus.
- **Embedding (dense/vector embedding)**: mapping text to a fixed-length float vector capturing semantic meaning.
- **Dense retriever**: a retriever based on dense embeddings, as opposed to sparse/lexical methods.
- **HNSW (Hierarchical Navigable Small World)**: the dominant ANN algorithm — multi-layer graph, coarse-to-fine greedy search.
- **Metadata filtering**: filtering candidate vectors by non-embedding fields (timestamps, IDs, permissions) before/during/after vector search — a `WHERE`-clause-like operation, not semantic search.
- **k (top-k)**: number of chunks retrieved; too small misses the answer, too large adds noise/cost/latency and can exceed context windows.
- **Matryoshka Representation Learning (MRL)**: embedding training technique that lets a single model's output vector be truncated to smaller dimensions (typically powers of two) with graceful quality degradation.
- **"Lost in the middle" effect**: LLMs pay more attention to context at the beginning and end of a prompt, degrading use of information buried in the middle — a confound to control for when evaluating chunking/ranking changes.

## Mental Models
- Use the "closed-book vs. open-book" framing (from Ch1) at the component level too: parsing/chunking/embedding are how you build the "open book" the LLM will consult.
- Think of ANN search (HNSW) as the "narrow down the city, then the neighborhood, then the block" heuristic — coarse layers first, fine layers last — rather than checking every address in the country.
- When picking chunk size, think about *two* context windows, not one: the embedding model's (often much smaller, e.g. 8k tokens) and the LLM's (much larger). Your chunking must respect the *smaller* of the two ceilings.
- Treat prompt templates as an empirical artifact, not a guess: build an eval set + LLM-as-judge (or a dedicated NLG evaluator) and pick the template/LLM combo that scores best, rather than debating "the right" placement of query vs. context in the abstract.

## Anti-patterns
- **Discarding all markup/metadata during parsing**: some tags (e.g. chat-log speaker attribution) encode information needed to answer downstream queries (e.g. "According to AI, ..."). Record such structure as metadata instead of throwing it away.
- **Using a VLM as the default parser**: VLMs hallucinate, are slow and expensive at enterprise scale — reserve them for hard document types (complex layouts, slides) or as a fallback after classical parsers (PDF libs, HTML parsers, OCR) fail. Follow a "cheapest successful parser first" strategy.
- **Fixed-size chunking without overlap**: breaks sentences/words mid-token, degrading embedding quality; mitigate with overlapping windows if you must use fixed-size chunking for speed.
- **Ignoring embedding model context window when chunking**: chunks longer than the embedding model's window get silently truncated by most serving frameworks, causing invisible information loss.
- **Picking chunking strategy by "vibes"**: strategy differences are often smaller than expected (per Qu, Bao & Tu, EMNLP 2024 — fixed-size vs. semantic chunking showed no measurable difference on BEIR/RAGBench); validate empirically on your own data and don't assume semantic chunking wins by default.
- **Judging chunking quality solely via generation-stage evaluation**: the "lost in the middle" effect confounds generation results with chunk *ranking*, not just chunk *content* — evaluate retrieval quality (precision/recall/nDCG/MRR) separately.

## Code Examples

```python
# PDF: extracting unstructured text while excluding table content (PyMuPDF)
def extract_unstructured_text(pdf_path):
    unstructured_text = []
    with pymupdf.open(pdf_path) as doc:
        for page in doc:
            page_text = page.get_text()
            tables = page.find_tables()
            table_text_blocks = []
            if tables.tables:
                for table in tables:
                    bbox = table.bbox
                    table_text = page.get_text(clip=bbox)
                    table_text_blocks.append(table_text.strip())
            lines = page_text.split('\n')
            filtered_lines = []
            for line in lines:
                line = line.strip()
                if line:
                    is_table_content = any(line in t for t in table_text_blocks)
                    if not is_table_content:
                        filtered_lines.append(line)
            page_unstructured = '\n'.join(filtered_lines)
            if page_unstructured.strip():
                unstructured_text.append(page_unstructured)
    return '\n\n'.join(unstructured_text)
```
- **What it demonstrates**: PDFs store characters by coordinate, not structure — table content must be explicitly extracted and subtracted from raw page text to get clean prose.

```python
# Vector storage & retrieval with pgvector
cursor.execute("CREATE EXTENSION IF NOT EXISTS vector;")
cursor.execute("""
    CREATE TABLE IF NOT EXISTS sentence_embeddings (
        sentence TEXT,
        embedding VECTOR(384)
    )
""")
cursor.execute("""
    CREATE INDEX ON sentence_embeddings
    USING hnsw (embedding vector_l2_ops)
    WITH (m = 16, ef_construction = 64);
""")

def vector_search(query, model, top_k):
    query_embedding = model.encode([query])[0].tolist()
    cursor.execute(
        """
        SELECT text, 1 - (embedding <=> %s::vector) as similarity
        FROM sentence_embeddings
        WHERE embedding IS NOT NULL
        ORDER BY embedding <=> %s::vector
        LIMIT %s;
        """,
        (query_embedding, query_embedding, top_k)
    )
    return cursor.fetchall()
```
- **What it demonstrates**: HNSW index creation and cosine-similarity search in Postgres via pgvector; note `<=>` returns *dissimilarity*, so similarity = `1 - (a <=> b)`.

```python
# Generation with Anthropic Claude, grounded in retrieved context
import anthropic
response = anthropic.Anthropic().messages.create(
    model="claude-sonnet-4-5",
    messages=[
        {"role": "user",
         "content": prompt_template.format(query=query, context=context)}
    ]
)
print(response.content[0].text)
```
- **What it demonstrates**: minimal generation call using a query-then-context prompt template (query placed before context, per the chapter's tip on template ordering).

## Reference Tables

| Method | Pros | Cons |
|---|---|---|
| Fixed-size chunking | Simple, predictable size, efficient batching | Breaks semantic/syntactic structure; needs overlap (redundancy) |
| Sentence/paragraph (content-aware) | Preserves linguistic structure; coherent | Boundary detection error-prone; uneven chunk sizes |
| Recursive chunking | Flexible, adapts to structure levels | More tuning; still delimiter-dependent |
| Document-structure chunking | Aligns with headings/sections | Needs well-structured docs (Markdown/HTML) |
| Semantic chunking | Best coherence, aligns with embedding search | Computationally expensive; unpredictable size; hard to debug |

Embedding model context windows (at time of writing): Qwen3-Embedding (0.6B/4B/8B) 32k · OpenAI text-embedding-3-{small,large} 8k · Gemini gemini-embedding-002 8k · BAAI BGE-M3 1k.

LLM context windows: GPT-5.1 400k · Claude Sonnet 4.5 1M · Gemini 3 1M — but inference latency scales roughly quadratically with context length (Llama 3.3 70B on 2×H100: 200 tokens→31ms, 10,000 tokens→1833ms), so bigger windows don't remove the need to chunk.

## Worked Example
Sentiment-similarity demo with Sentence Transformers (`all-MiniLM-L6-v2`, 384-dim): embedding four sentences — "I am a happy person," "I am a joyful person," "I am a pessimistic person," "I am not an optimistic person" — and computing pairwise cosine similarity produces a 4×4 matrix where sentence 1↔2 similarity is 0.8151 (semantically aligned, different wording) and 1↔3 is only 0.3864. Querying this same table via pgvector with `"I am a smiling person"` returns "happy" (0.76) and "joyful" (0.69) far ahead of "not optimistic" (0.38) — concretely showing semantic search matching meaning, not surface tokens. Querying instead with `"I have a bad feeling about the future"` correctly flips the ranking to surface "pessimistic" (0.56) and "not optimistic" (0.51) first.

## Key Takeaways
1. Parsing errors are unrecoverable downstream — bad OCR or lost table structure at ingestion cannot be fixed by better retrieval or generation later.
2. Chunk size must respect the *embedding* model's context window, which is usually far smaller than the LLM's.
3. Don't assume one chunking strategy is empirically superior by default — validate on your own retrieval benchmark; published results (BEIR/RAGBench) found little difference between fixed-size and semantic chunking on short-passage benchmarks.
4. Reranking exists because embedding-based top-k ranking is necessary but insufficient — redundant chunks waste tokens, and cross-attention rerankers capture query-chunk relationships embeddings miss.
5. Vector DBs are more than ANN indexes — they add persistence, metadata filtering, updates/deletes, and integration; general-purpose DBs (Postgres+pgvector, SQLite+sqlite-vec, MongoDB Atlas) now compete with dedicated vector DBs (Pinecone, Milvus, Weaviate, Qdrant).
6. Pick top-k, embedding dimension, and LLM/prompt combination empirically via an eval set and LLM-as-judge (or a dedicated NLG evaluator like HHEM), not by intuition.
7. No RAG layer operates in isolation — chunking quality gates retrieval quality gates generation quality; debugging requires tracing the whole pipeline, not one stage.

## Connects To
- **Ch1**: this chapter is the detailed implementation of the ingestion/query flows introduced there.
- **Ch3**: extends this "base stack" with hybrid search, reranking depth, and guardrails.
- **Ch6**: the LLM-as-judge and NLG-evaluator concepts introduced here (relevance, faithfulness) are expanded into full RAG evaluation metrics.
- **Ch8**: multimodal parsing (tables, images) introduced here via PyMuPDF/python-docx is expanded for full multimodal RAG.
