# Chapter 7: Document Summarization and RAG with LangChain.js

## Core Idea
Two production project patterns built on LangChain.js: (1) document summarization for long texts using MapReduce/stuffing/refine strategies to fit within context windows, and (2) a full RAG system (offline document indexing → online query embedding → retrieval → context augmentation → grounded generation) with a persistent vector store (HNSWLib) and Google grounding to reduce hallucination.

## Frameworks Introduced
- **MapReduce summarization**: split document → summarize each chunk independently (map) → combine chunk summaries into a final summary (reduce). Enables parallelization for large documents.
  - When to use: long documents that exceed the model's context window.
  - How: `loadSummarizationChain` with the `map_reduce` type; each chunk summarized in isolation, then aggregated.
- **Stuffing vs. refine summarization**: stuffing passes the whole document in one prompt call (best for short docs that fit context); refine summarizes the first chunk, then iteratively concatenates+re-summarizes with each subsequent chunk (best for long docs needing cross-chunk continuity, at the cost of error propagation from early bad summaries).
  - When to use: stuffing when document fits context window; refine when it doesn't but sequential coherence matters more than parallelism (MapReduce).
- **The RAG architecture** (offline/online split): Offline — index documents into a vector DB. Online — embed user query → retrieval mechanism (similarity search) → context augmentation (merge retrieved chunks + query into a prompt) → generation engine (LLM) → response.
  - When to use: any application needing to ground LLM answers in a specific, current, or private document corpus rather than the model's frozen training knowledge.
  - How: keep the four RAG components (document indexing, retrieval mechanism, augmentation layer, generation engine) as separately reasoned-about pieces — each has its own quality levers (chunking strategy, `k` retrieved docs, prompt template, model choice).

## Key Concepts
- **Grounding**: a verification step cross-checking generated responses against a corpus of reliable information to reduce hallucination (here, via Google Gemini's log-probability/grounding support).
- **HNSWLib**: a filesystem-persistent vector store (vs. the in-memory `MemoryVectorStore` from Ch6) suitable for production indexing that must survive process restarts.
- **`asRetriever({ k: 6 })`**: configures how many top-similar chunks a retriever returns per query.
- **MultiHop-RAG**: a RAG variant that recursively retrieves additional context based on initial results, useful when an answer requires reasoning across multiple supporting documents.
- **HyDE (hypothetical document embedding)**: generates synthetic "hypothetical" document text to improve retrieval matching, rather than relying only on real indexed documents.

## Mental Models
- Treat summarization strategy choice as a document-length/coherence trade-off: stuffing (simple, short docs) → MapReduce (parallel, but chunk summaries may lack cross-chunk continuity) → refine (sequential, coherent, but early errors compound).
- Split RAG indexing into offline (batch, cost-tolerant, run once per corpus update) vs. online (per-query, latency-sensitive) phases — this framing clarifies where to optimize for throughput (indexing) vs. speed (retrieval+generation).
- Grounding is a distinct concern from retrieval: retrieval finds relevant context; grounding verifies the *generated* answer is actually supported by that context — don't conflate the two.

## Anti-patterns
- **Using the stuffing method on long/complex documents**: the entire document must fit in one prompt — fails or truncates on anything beyond context window size.
- **Using MapReduce when cross-chunk narrative coherence matters**: independent chunk summaries can lose connective context that a sequential refine pass would preserve.
- **Treating retrieval as sufficient grounding on its own**: retrieving relevant chunks doesn't guarantee the LLM's generated answer is faithful to them — add an explicit grounding/verification layer for hallucination-sensitive use cases.
- **Re-indexing documents synchronously per user request**: indexing is explicitly recommended as an offline, batch process — doing it online blocks the query path and wastes cost on repeat runs.

## Code Examples
```js
// Document indexing into a persistent HNSWLib vector store
import { HNSWLib } from '@langchain/community/vectorstores/hnswlib';

async indexDocumentsFromDirectory(documentDirectory) {
  const documents = [];
  const pdfFiles = fs.readdirSync(documentDirectory)
    .filter((f) => path.extname(f).toLowerCase() === '.pdf')
    .map((f) => path.join(documentDirectory, f));
  for (const filePath of pdfFiles) {
    documents.push(...await this.processDocument(filePath));
  }
  const vectorStore = await HNSWLib.fromDocuments(documents, this.embeddings);
  return vectorStore;
}
```
```js
// Loading a persisted index and creating a top-6 retriever
async loadIndex(path) {
  this.vectorStore = await HNSWLib.load(path, this.embeddings);
  this.retriever = this.vectorStore.asRetriever({ k: 6 });
}
```
```js
// Full RAG chain: retrieve, augment, generate, return sources
async performRAG(query) {
  const prompt = ChatPromptTemplate.fromTemplate(`
    Answer the question based only on the context provided.
    Context: {context}
    Question: {question}`);
  const chain = RunnableSequence.from([
    { context: this.retriever.pipe(formatDocs), question: new RunnablePassthrough() },
    prompt, this.llm, new StringOutputParser(),
  ]);
  const response = await chain.invoke(query);
  const sourceDocuments = await this.retriever.invoke(query);
  return { answer: response, sourceDocuments };
}
```

## Reference Tables
| Summarization method | Best for | Trade-off |
|---|---|---|
| Stuffing | Short docs fitting context window | Fails on long/complex docs |
| MapReduce | Long docs, parallelizable | Chunk summaries may lose cross-chunk coherence |
| Refine | Long docs needing coherence | Sequential, slower; early errors propagate |

| RAG component | Role | Failure mode if weak |
|---|---|---|
| Document indexing | Converts docs → embeddings → vector store | Poor chunking → irrelevant retrieval |
| Retrieval mechanism | Finds top-k similar chunks | Too few/many `k` hurts precision/recall |
| Augmentation layer | Merges context + query into prompt | Context overflow if not budgeted |
| Generation engine | Produces final answer | Hallucination if ungrounded |

## Worked Example
Building the RAG web app end-to-end: offline, `node scripts/indexDocuments.google.js -d corpus` walks a directory of aviation-accident-report PDFs (from the KG-RAG-datasets), chunks each with the Ch6 text splitter, embeds with Google Gemini embeddings, and persists to an HNSWLib index on disk. Online, `loadIndex(path)` reloads that index and exposes a `k: 6` retriever; `performRAG(query)` retrieves the 6 most relevant chunks, formats them into a `{context}` slot in a strict "answer only from context" prompt template, generates the answer via the LLM, and separately returns `sourceDocuments` so the UI can cite exactly which report(s) the answer drew from — a concrete transparency/grounding mechanism.

## Key Takeaways
1. Choose a summarization strategy (stuffing/MapReduce/refine) based on document length and whether cross-chunk coherence matters more than parallel speed.
2. Structure any RAG system around the four canonical components (indexing, retrieval, augmentation, generation) so each can be tuned/debugged independently.
3. Run document indexing offline as a batch process — never synchronously per user query.
4. Persist vector indexes (HNSWLib or similar) rather than relying on `MemoryVectorStore` once the app needs to survive restarts.
5. Always return `sourceDocuments` alongside the generated answer — surfacing provenance is a cheap, high-value trust/grounding mechanism.
6. Retrieval alone does not guarantee a faithful answer — pair it with an explicit grounding/verification step (e.g., Gemini's grounding support) when hallucination risk is high.

## Connects To
- **Ch6**: directly extends the retrieval chain, text splitters, and `asRetriever` pattern introduced there from prototyping (`MemoryVectorStore`) to production (`HNSWLib`).
- **Ch5**: reuses embeddings concepts; grounding here addresses the hallucination risk flagged back in Ch1.
- **Ch11**: revisits RAG with a more advanced, realistic real-world project.
- **Ch8**: addresses testing/latency concerns relevant to a production RAG pipeline's multi-step chain.
