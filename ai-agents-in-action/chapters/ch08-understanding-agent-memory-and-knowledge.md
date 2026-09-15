# Chapter 8: Understanding Agent Memory and Knowledge

## Core Idea
Retrieval augmented generation (RAG) — embed content into vectors, store in a vector database, retrieve by similarity, inject as context — is the single mechanism underlying both agent "knowledge" (static documents) and agent "memory" (dynamic conversational facts), differing mainly in how content enters the store; memory further subdivides into sensory/short-term/long-term and semantic/episodic/procedural forms, and both knowledge and memory benefit from periodic compression as they scale.

## Frameworks Introduced
- **RAG Two-Phase Pattern**: (1) load -> transform/split -> embed -> store; (2) query -> embed query -> similarity search -> inject as context -> generate.
  - When to use: any time an agent needs to answer questions grounded in documents too large to fit in a prompt, or needs to recall prior conversational facts.
  - How: choose a splitter/chunk size/overlap for phase 1; choose an embedding model and vector DB; at query time embed the query the same way and retrieve top-N by similarity/distance before calling the LLM.
- **Memory Taxonomy (cognitive-inspired)**: sensory / short-term (working) / long-term memory, further split into semantic / episodic / procedural at the long-term level.
  - When to use: to decide what kind of memory store an agent needs — a chat buffer (short-term/working) vs. durable facts about the user (semantic long-term) vs. event history (episodic) vs. how-to knowledge (procedural).
  - How: short-term = conversation history held in a thread/context window; long-term = extracted facts pushed through an LLM "memory function" into a vector store, tagged/categorized by memory type.
- **Semantic Memory Augmentation**: an extra LLM pass that extracts relevant questions/categorized statements from raw input before embedding, rather than embedding raw conversation text directly.
  - When to use: when plain memory retrieval isn't surfacing the most relevant facts — semantic augmentation improves recall precision at the cost of an extra LLM call per memory write.
  - How: run a "memory function" prompt (e.g., "Summarize the conversation... return JSON with categorized statements") over new input before embedding/storing it.
- **Memory/Knowledge Compression**: cluster (e.g., k-means) then summarize each cluster into a more succinct representation.
  - When to use: when a store accumulates redundant/repetitive/unbalanced content over time — verbose literary sources benefit more than terse code; memory benefits from periodic re-compression, knowledge usually only needs it once on load; multiple compression passes (multi-level compression) can further improve retrieval.

## Key Concepts
- **Knowledge vs. memory**: knowledge = augmentation from pre-existing static documents; memory = augmentation from the agent's own captured interactions/facts — both use the same retrieval mechanism, differing in how the store gets populated.
- **TF-IDF (Term Frequency-Inverse Document Frequency)**: a classic word-frequency-based vectorization; simple but blind to word relationships/context.
- **Cosine similarity/distance**: similarity = cos(angle) between vectors, range -1..1 (1 = identical); distance = 1 - similarity, range 0..2 (0 = identical, 2 = opposite).
- **Document embedding**: neural-network-based vectorization (e.g., OpenAI `text-embedding-ada-002`, 1536 dimensions) that captures semantic/syntactic relationships far better than TF-IDF.
- **Chunking/splitting (with overlap)**: breaking documents into pieces small enough to embed/retrieve meaningfully; overlap (e.g., 25 of 100 characters) prevents cutting off ideas mid-thought; token-based splitting (vs. character-based) generally yields more semantically coherent, better-performing chunks.
- **Vector database**: stores embeddings for similarity search at scale (e.g., ChromaDB for local/small-scale use); returns nearest neighbors by distance.
- **Nexus Knowledge Store / Memory Store**: Nexus's UI-configurable implementations of knowledge (static document RAG) and memory (LLM-mediated fact extraction into a vector store), each configurable for splitter type, chunk size, overlap, and (for memory) semantic/episodic/procedural type.

## Mental Models
- Use TF-IDF only for learning/simple exact-term-ish matching; use trained embeddings (OpenAI or similar) whenever semantic (meaning-based, not just word-based) matching matters — TF-IDF misses paraphrases entirely.
- Treat knowledge stores as "index once, query often" and memory stores as "continuously written and periodically compressed" — this difference should drive how often you invest in compression for each.
- When retrieval results look word-matchy but semantically wrong (or vice versa), diagnose by checking which vectorization method is in use (TF-IDF vs. embeddings) and which splitter (character vs. token-based) produced the chunks.

## Anti-patterns
- **Submitting whole documents in the prompt instead of chunking/retrieving**: even though modern LLMs technically fit larger contexts, doing so costs more tokens and often doesn't improve results — RAG-style retrieval of relevant chunks remains the better default.
- **Using TF-IDF for semantic-similarity-dependent applications**: it only captures word frequency, not meaning — queries that paraphrase (different words, same meaning) will fail to match well.
- **Leaving memory/knowledge stores uncompressed as they grow**: repetitive or duplicate content degrades retrieval relevance over time; the chapter explicitly flags large/unbalanced clusters as the trigger to compress.
- **Assuming one compression pass is always enough**: the chapter notes multiple compression passes (multi-level compression) have been shown to further improve retrieval performance in some cases — don't stop at the first pass without checking.

## Code Examples
```python
def cosine_similarity_search(query, database, vectorizer, top_n=5):
    query_vec = vectorizer.transform([query]).toarray()
    similarities = cosine_similarity(query_vec, database)[0]
    top_indices = np.argsort(-similarities)[:top_n]
    return [(idx, similarities[idx]) for idx in top_indices]
```
```python
chroma_client = chromadb.Client()
collection = chroma_client.create_collection(name="documents")
collection.add(embeddings=embeddings, documents=documents, ids=ids)

def query_chromadb(query, top_n=2):
    query_embedding = get_embedding(query)
    results = collection.query(query_embeddings=[query_embedding], n_results=top_n)
    return [(id, score, text) for id, score, text in
            zip(results['ids'][0], results['distances'][0], results['documents'][0])]
```
- **What it demonstrates**: the progression from a hand-rolled TF-IDF + cosine-similarity search (first snippet) to a real embedding-backed vector database query (second snippet, ChromaDB + OpenAI embeddings) — the same retrieval *shape* (embed query, compare, rank, return top-N) underlies both, but embeddings capture meaning where TF-IDF captures only word overlap.

## Reference Tables
| Vectorization method | Captures meaning? | Notes |
|---|---|---|
| TF-IDF | No (word-frequency only) | Simple, fast, no training needed, blind to synonyms/paraphrase |
| OpenAI embeddings (e.g., ada-002) | Yes | 1536 dimensions; needs PCA or similar to visualize; standard for most memory/retrieval apps |

| Memory type | Maps to | Nexus/agent mechanism |
|---|---|---|
| Sensory | Immediate multi-modal input (text/image/audio) | RAG-like but non-text forms |
| Short-term/working | Recent conversation buffer | Held directly in thread/context |
| Long-term (semantic) | Durable facts/concepts | Vector store + LLM memory function extraction |
| Long-term (episodic) | Event history | Vector store, event-focused extraction |
| Long-term (procedural) | Steps/processes | Vector store, process-focused extraction |

| Compression guidance | Recommendation |
|---|---|
| Verbose/literary sources | Benefit more from compression |
| Repetitive code | Can still benefit if highly repetitive |
| Memory stores | Benefit from periodic/repeated compression |
| Knowledge stores | Usually only need compression once, at load |
| Multiple passes | Can further improve retrieval performance |

## Worked Example
The chapter builds a movie-script knowledge store in Nexus end-to-end: upload `back_to_the_future.txt` to a new Knowledge Store -> Nexus chunks/embeds it into ChromaDB (configurable splitter type, chunk size, overlap) -> inspect the resulting embeddings in a 3D plot and run test queries -> enable the `time_travel` knowledge store on a knowledge-capable agent engine in the Chat page -> ask questions grounded in the script. A parallel worked example builds a "my_memory" memory store: add personal facts/preferences in the Memory page -> Nexus runs each entry through a conversational memory function ("Summarize the conversation and create a set of statements... Return a JSON object...") to extract structured statements -> enable `my_memory` in Chat with a different agent engine and observe the agent recalling those facts, demonstrating memory and knowledge share the retrieval mechanism but differ in how they're populated (upload vs. LLM-mediated extraction).

## Key Takeaways
1. RAG is one mechanism (embed -> store -> retrieve -> augment) powering both agent knowledge (static documents) and agent memory (dynamic facts) — the difference is population, not retrieval.
2. Use trained embeddings, not TF-IDF, whenever true semantic (paraphrase-tolerant) matching is required.
3. Chunk with overlap, and prefer token-based over character-based splitting for better-quality, more coherent chunks.
4. Map an agent's memory needs onto sensory/short-term/long-term (and semantic/episodic/procedural within long-term) rather than treating "memory" as one undifferentiated bucket.
5. Semantic memory augmentation (an extra LLM extraction pass before embedding) improves recall relevance over embedding raw conversational text directly.
6. Compress memory and knowledge stores (cluster + summarize) once redundancy/imbalance appears — memory benefits from periodic compression, knowledge typically only once; multiple passes can help further.
7. A single knowledge/memory store can serve multiple agents, and advanced systems may need multiple stores (e.g., per-user memory, shared vs. private) — plan store architecture accordingly as systems scale.

## Connects To
- **Ch1**: implements the "knowledge/memory" component of the five-component agent model.
- **Ch5**: the native "load seen movies" function in chapter 5 was an informal preview of the memory concepts formalized here.
- **Ch7**: extends Nexus (introduced in ch7) with its Knowledge Store Manager and Memory pages.
- **Ch9**: LangChain's retrieval abstractions here parallel Microsoft Prompt Flow's evaluation-oriented tooling covered next.
