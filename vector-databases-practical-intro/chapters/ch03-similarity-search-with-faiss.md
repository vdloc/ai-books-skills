# Chapter 3: Similarity Search with FAISS

## Core Idea
FAISS (Facebook AI Similarity Search) implements the core index structures behind all practical vector search — flat (exact), IVF (inverted file / clustering), LSH, PQ (product quantization), and HNSW (graph-based) — each trading off accuracy, speed, and memory differently; choosing the right index is a deliberate engineering decision, not a default.

## Frameworks Introduced
- **FAISS Index Selection Heuristic**: pick by dataset size, dimensionality, desired accuracy, memory constraints, speed requirements, and hardware (GPU availability). Preserve as a checklist, not a single rule.
  - When to use: any time provisioning or choosing a vector index — this precedes almost every other design decision in a vector DB.
  - How: small dataset / exact results required → flat index. Large-scale / approximate OK → IVF or HNSW. High-dimensional data → HNSW tends to perform best. Tight memory → PQ-based index. Max speed → HNSW, then IVF. GPU available → consider GPU-accelerated variants.
- **Quantization**: reduce a vector's representational resolution (bits/dimensions) to trade some accuracy for large space/speed gains — defining codebooks, encoding, decoding as the three-step subprocess.

## Key Concepts
- **Distance metrics**: L2 (Euclidean) distance, inner-product distance, cosine similarity — pick per embedding model's training objective (many sentence-transformer models are optimized for cosine).
- **Flat index (IndexFlatL2 / IndexFlatIP)**: brute-force exact search; exhaustive; simplest and most accurate but doesn't scale.
- **IVF (Inverted File) index**: partitions vector space into `nlist` clusters (via a quantizer, e.g. IndexFlatL2), searches only `nprobe` clusters at query time — approximate, much faster than flat at scale.
- **LSH (Locality-Sensitive Hashing)**: hashes similar vectors into the same buckets using random projections/rotations; exhaustive within buckets.
- **HNSW (Hierarchical Navigable Small World)**: graph-based index; generally the fastest search speeds and best for high-dimensional data, at the cost of more memory and no exact guarantees.
- **PQ (Product Quantization)**: splits vectors into subspaces, each quantized independently via a learned codebook — dramatic memory savings (e.g. IndexIVFPQ combines coarse quantization + PQ on residuals).
- **`nprobe`**: IVF search-time parameter — number of clusters searched; higher = more accurate but slower. The single most important IVF tuning knob.

## Mental Models
- Think of the flat-vs-IVF-vs-HNSW choice as **exactness vs. scale**: flat indexes are ideal for small datasets needing exact results; IVF/HNSW excel at large-scale, approximate-is-fine applications.
- Treat **`nprobe` (IVF) and similar search-time parameters as a live accuracy/speed dial** — you don't have to choose one index config at index-build time only; retune per query load.
- Use **composite indexes (IndexPreTransform, IndexIDMap, IndexRefineFlat, IndexShards)** when a single index type doesn't satisfy the constraint set — combine dimensionality reduction, ID mapping, re-ranking, and sharding as needed rather than forcing one index to do everything.

## Anti-patterns
- **Defaulting to a flat index at scale**: exact search cost grows linearly with dataset size — fine for prototypes, unworkable for large production corpora.
- **Ignoring `nprobe` tuning**: leaving IVF at default `nprobe` either wastes speed (too high) or silently tanks recall (too low) without an explicit trade-off decision.
- **Choosing HNSW under tight memory constraints**: HNSW's graph structure costs more memory per vector than PQ-based approaches — mismatched to the actual constraint.

## Reference Tables

| Method | Class | Exhaustive | Bytes/vector | Best for |
|---|---|---|---|---|
| Exact L2 | IndexFlatL2 | Yes | 4×d | Small datasets, exact results |
| Exact inner product / cosine | IndexFlatIP | Yes | 4×d | Same, cosine (normalize vectors first) |
| HNSW graph | IndexHNSWFlat | No | 4×d + M-dependent | Fastest search, high-dimensional data |
| Inverted file | IndexIVFFlat | No | 4×d + 8 | Large-scale, approximate OK |
| LSH | IndexLSH | Yes (within buckets) | ceil(nbits/8) | Memory-light approximate search |
| Scalar quantization | IndexScalarQuantizer | Yes | d (SQ8) | Moderate memory savings, exact-ish |
| Product quantization | IndexPQ | Yes | ceil(M×nbits/8) | Strong memory savings |
| IVF + PQ (IVFADC) | IndexIVFPQ | No | ceil(M×nbits/8)+8 | Large-scale + memory-constrained |
| IVF + PQ + re-ranking | IndexIVFPQR | No | M+M_refine+8 | IVFADC accuracy boost via re-ranking |

**Decision heuristic** — Dataset size (small→flat, large→IVF/HNSW) · Dimensionality (high→HNSW) · Accuracy (exact→flat, approximate OK→IVF/HNSW) · Memory (tight→PQ) · Speed (max→HNSW, then IVF) · Hardware (GPU available→GPU variants).

## Worked Example
Building and querying an `IndexIVFFlat` on 100K synthetic 64-dim vectors:
```python
import faiss
import numpy as np

d = 64          # vector dimension
nb = 100000     # dataset size
nq = 1000       # query count

np.random.seed(1234)
xb = np.random.random((nb, d)).astype('float32')
xq = np.random.random((nq, d)).astype('float32')

nlist = 100     # number of clusters
k = 4           # neighbors to retrieve
quantizer = faiss.IndexFlatL2(d)          # clustering index (can itself be an ANN index)
index = faiss.IndexIVFFlat(quantizer, d, nlist)

index.train(xb)   # IVF indexes require a training step to learn cluster centroids
index.add(xb)

index.nprobe = 10  # clusters searched at query time — accuracy/speed trade-off knob
D, I = index.search(xq, k)  # D = distances, I = neighbor indices
```
Contrast with `IndexIVFPQ` for memory-constrained large-scale search:
```python
quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFPQ(quantizer, d, nlist, m, k)  # m = PQ subspaces, k = bits/subquantizer
```

## Key Takeaways
1. There is no single "best" FAISS index — selection is a multi-factor trade-off (size, dimension, accuracy, memory, speed, hardware).
2. Flat indexes are exact but don't scale; IVF/HNSW/PQ trade exactness for speed and/or memory at production scale.
3. `nprobe` is the primary live accuracy/speed tuning knob for IVF indexes — always set and tune it deliberately.
4. HNSW is generally fastest and best for high-dimensional data, but costs more memory than PQ-based approaches.
5. Composite indexes (IDMap, PreTransform, RefineFlat, Shards) let you combine techniques rather than forcing one index type to satisfy every constraint.
6. Match the distance metric (L2, inner product, cosine) to what the embedding model was actually trained/optimized for.

## Connects To
- **Ch 2**: FAISS indexes the embeddings produced there.
- **Ch 4, Ch 5, Ch 7**: SQLite and pgvector chapters implement these same index concepts (HNSW, IVF-equivalents) inside a relational database rather than a standalone FAISS index.
- **ANN (external concept)**: this chapter is a deep dive into Approximate Nearest Neighbor search, introduced conceptually in Ch 1.
