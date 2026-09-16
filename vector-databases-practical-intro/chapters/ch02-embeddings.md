# Chapter 2: Embeddings

## Core Idea
Embeddings map unstructured data into dense vectors where geometric proximity reflects semantic similarity — this learned representation (not hand-crafted statistics) is the foundation modern LLMs, GenAI, and vector search are built on.

## Frameworks Introduced
- **Word2Vec (Mikolov et al., 2013)**: represents words as dense vectors via a shallow neural network trained to predict a word from context (CBOW) or context from a word (skip-gram); extracts the hidden-layer weights as embeddings. Preserve exact naming — CBOW / skip-gram.
  - When to use: historically foundational; conceptually still the model for "meaning = proximity in vector space."
  - How: train on large unlabeled text corpus → words with similar contexts converge to similar vectors → supports vector arithmetic (`king − man + woman ≈ queen`).
- **Transformer embedding families**: Encoder-only (BERT and variants — best for embeddings/classification), Decoder-only (GPT family — best for generation), Encoder-Decoder (T5, BART — best for seq2seq tasks like summarization/translation). Preserve this three-way split; it determines which architecture to reach for.
- **RAG pipeline (conceptual)**: embed document → store in vector DB → embed query with the *same* model → similarity search top-k → pass retrieved context + query to LLM.

## Key Concepts
- **Embedding (recap)**: verb = mapping process; noun = resulting vector(s).
- **Contrastive learning**: training embedding models on positive (similar) and negative (dissimilar) sentence pairs.
- **Knowledge distillation**: training a small model (e.g. all-MiniLM-L6-v2) to mimic a larger one, trading some accuracy for speed/size.
- **all-MiniLM-L6-v2**: 384-dim, ~80MB, ~2,000 sentences/sec on GPU — the book's default embedding model for enterprise RAG, chosen for speed/size/accuracy balance, not peak accuracy.
- **all-mpnet-base-v2**: 768-dim, ~420MB, ~1,000 sentences/sec — higher accuracy, use when accuracy matters more than latency/cost.
- **paraphrase-multilingual-mpnet-base-v2**: 768-dim, 50+ languages, ~970MB — use for cross-lingual similarity.
- **Zero-shot learning (via embeddings)**: embedding models generalize to unseen categories/tasks without task-specific fine-tuning, because semantic proximity itself does the classification work.

## Mental Models
- Think of embedding model choice as a **speed/size/accuracy/language trade-off**, not a single "best" model — pick per constraint (production latency → MiniLM; accuracy-critical → mpnet-base; multilingual → paraphrase-multilingual).
- Use the **same embedding model for both documents and queries** — embeddings from different models don't share a comparable vector space; mismatching them silently breaks similarity search.
- Treat embedding quality as **the hidden bottleneck of RAG**: "it's often the careful use of embedding models that determines whether an AI application is practical and effective in production" — more attention often goes to the LLM than to this earlier, equally decisive step.

## Anti-patterns
- **Using different embedding models for indexing vs. querying**: vectors won't be comparable; similarity search silently returns garbage.
- **Treating embeddings as generative models**: sentence-transformers models are not chatbots — they only produce vectors, not text generation.
- **Choosing the biggest/most accurate model by default**: for production systems, a smaller model (MiniLM) is often "good enough" and dramatically cheaper/faster; oversized models add latency and cost without proportional gains for many tasks.

## Reference Tables

| Model | Dimensions | Size | Speed (GPU) | Best for |
|---|---|---|---|---|
| all-MiniLM-L6-v2 | 384 | ~80 MB | ~2,000 sent/s | Production, speed/efficiency |
| all-mpnet-base-v2 | 768 | ~420 MB | ~1,000 sent/s | Accuracy-critical applications |
| paraphrase-multilingual-mpnet-base-v2 | 768 | ~970 MB | Slower | Multilingual (50+ languages) |

| Transformer type | Examples | Primary use |
|---|---|---|
| Encoder-only | BERT and variants | Embeddings, classification, semantic search |
| Decoder-only | GPT family | Text generation |
| Encoder-Decoder | T5, BART | Summarization, translation, seq2seq |

## Worked Example
Minimal RAG pipeline using `sentence-transformers` + ChromaDB:
```python
from sentence_transformers import SentenceTransformer
from chromadb import Client, Settings

model = SentenceTransformer('all-MiniLM-L6-v2')  # 384-dim, fast/small
chroma_client = Client(Settings(is_persistent=False))
collection = chroma_client.create_collection(name="climate_docs")

documents = [
    "Climate change is affecting global weather patterns, causing more extreme events.",
    "Rising sea levels threaten coastal communities worldwide.",
    "Greenhouse gas emissions continue to rise despite international agreements."
]
embeddings = model.encode(documents)
collection.add(
    embeddings=[e.tolist() for e in embeddings],
    documents=documents,
    ids=[f"doc_{i}" for i in range(len(documents))]
)

query = "How does climate change affect weather?"
query_embedding = model.encode(query)
results = collection.query(query_embeddings=[query_embedding.tolist()], n_results=2)
for doc in results['documents'][0]:
    print(f"Retrieved document: {doc}")
```
Key points the book stresses: same model for docs and query; cosine similarity used by default; production would add error handling, chunking, persistent storage, distance-threshold filtering, and metadata handling — all deferred to later chapters.

## Key Takeaways
1. Embeddings are learned representations (trained on corpora), not hand-crafted feature vectors — proximity = semantic similarity by construction.
2. Word2Vec's skip-gram/CBOW insight (predict word↔context) is the conceptual ancestor of every modern embedding model, including today's transformers.
3. Match transformer architecture to task: encoder-only for embeddings/search, decoder-only for generation, encoder-decoder for seq2seq.
4. Always embed queries and documents with the *same* model — this is the single most common integration bug.
5. Default to all-MiniLM-L6-v2 for production RAG unless you specifically need higher accuracy (mpnet-base) or multilingual support (paraphrase-multilingual-mpnet-base-v2).
6. Embedding model choice is as decisive for RAG quality as LLM choice — don't under-invest here.

## Connects To
- **Ch 1**: applies the vector/similarity-search concepts introduced there using a concrete embedding model.
- **Ch 3–8**: all downstream FAISS/SQLite/pgvector/RAG chapters use all-MiniLM-L6-v2 as the default embedding model established here.
- **Transformer architecture (external concept)**: BERT/GPT/T5 families referenced throughout as the origin of modern embedding techniques.
