# Chapter 9: Vector Query Language

## Core Idea
This chapter is explicitly a **speculative proposal**, not documented shipping technology: Vector Query Language (VQL) is a hypothetical SQL-inspired query language for vector databases, meant to seed community standardization — its value here is as a vocabulary/syntax vocabulary for vector-specific operations that current vendor-specific APIs don't standardize.

## Frameworks Introduced
- **VQL Data Model**: Collection (≈database) → Table (≈DB table, structured vectors + metadata) → Vector (fixed-dim numerical array) → Embedding (a vector representing an encoded entity) → Field (one dimension's value) → Metadata (non-vector associated data) → Distance metric (required, defined on the vector space).
  - When to use: as a conceptual checklist for evaluating or designing any vector database's abstraction layer — does it cleanly separate these six concerns?
- **VQL Query Template**: `SELECT subspace FROM collection.table VECTOR_OPERATION operation_parameters WHERE metadata_filters USING METRIC distance_metric LIMIT top_k` — SQL's SELECT/FROM/WHERE/LIMIT plus a vector-specific VECTOR_OPERATION clause.
- **Hybrid Search Score Fusion (VQL formalization)**: `final_score = w1 * norm(vector_score) + w2 * norm(text_score)`, weights nonnegative and summing to 1.0; default text scoring is BM25 (configurable). This is the same fusion idea as Ch 6's hybrid_search, expressed as declarative syntax instead of imperative code.
- **Centroid vs. Geometric Median (vector aggregation)**: centroid = `argmin_x Σᵢ‖x − vᵢ‖²` (closed-form, average coordinates); geometric median = `argmin_x Σᵢ‖x − vᵢ‖` (no closed form, iterative, robust to outliers). Preserve the distinction exactly — dropping the square is what changes the optimization landscape and the robustness property.

## Key Concepts
- **Similarity Search (VQL)**: `SIMILARITY SEARCH [vector] USING METRIC cosine TOP K n` — nearest-neighbor query with an explicit metric and result cap.
- **Range Search**: find all vectors within a distance `THRESHOLD` of a query vector, rather than a fixed top-K — useful when "all relevant items" matters more than "the K most similar."
- **Batch Similarity Search**: run many query vectors against one table in a single operation; output schema `(query_id, result_rank, matched_vector, similarity_score)`, execution may be parallelized but per-query ordering is preserved.
- **Vector functions**: `DIMENSION(v)`, `DISTANCE(v1, v2, metric)`, `CONCAT(v1, v2)`, `DOT(v1, v2)` — basic vector algebra as query-language primitives.
- **Vector arithmetic in queries**: `vector_data + influence_vector`, `vector_data * scalar_value` — element-wise operations directly in SELECT.
- **Medoid**: the actual data point (not a computed point) from the collection that minimizes the sum of distances to all others — related to, but distinct from, the geometric median.

## Mental Models
- Treat VQL as **"what a standard vector query interface could look like"** rather than something to actually implement against — its value is vocabulary and pattern recognition when reading vendor-specific vector DB APIs (FAISS, pgvector, sqlite-vss all implicitly implement fragments of this data model).
- Use the **centroid-vs-geometric-median choice as a robustness decision**: if outliers are expected/likely in a vector cluster, the geometric median (or medoid, for a "real" representative point) resists distortion; the centroid does not.
- Recognize that **the industry lacks the relational-database abstraction layer** SQL provides — every vector DB vendor currently exposes its own metaphors, which "leak into the application architecture." This is the actual production pain point VQL is a response to, independent of whether VQL itself ships.

## Anti-patterns
- **Treating this chapter's syntax as an existing, installable product**: the book is explicit — "this language doesn't currently exist as an implementation." Don't cite VQL syntax as something you can run today.
- **Using the centroid as "the average" when outliers are present**: centroid is dragged toward outliers (squared-distance optimization); if that's undesirable, use geometric median or medoid instead.
- **Assuming hybrid search weights are independent**: VQL and Ch 6 both require w1+w2=1.0 and nonnegative — arbitrary unnormalized weights break the "70% vector / 30% text" interpretability the fusion formula promises.

## Reference Tables

| Vector aggregation | Formula | Closed-form? | Outlier robustness |
|---|---|---|---|
| Centroid (mean) | argmin_x Σ‖x−vᵢ‖² | Yes (average coordinates) | Low — dragged toward outliers |
| Geometric median | argmin_x Σ‖x−vᵢ‖ | No — iterative | High — like scalar median |
| Medoid | (discrete) point minimizing Σ distance to others | No — search over actual points | High, and always a real data point |

| VQL operation | Purpose |
|---|---|
| `SIMILARITY SEARCH ... TOP K n` | Nearest-neighbor, fixed result count |
| `RANGE SEARCH ... THRESHOLD t` | All results within a distance bound |
| `HYBRID SEARCH (VECTOR ... WEIGHT w1, TEXT ... WEIGHT w2)` | Weighted vector+text fusion |
| `BATCH SIMILARITY SEARCH (...)` | Many queries against one table in one call |

## Worked Example
Metadata-filtered similarity search:
```sql
SELECT * FROM ecommerce.product_vectors
SIMILARITY SEARCH [1.2, 0.8, -0.2, 0.5]
THRESHOLD 0.8
USING METRIC euclidean
WHERE category = 'electronics' AND price < 1000
TOP K 5;
```
Hybrid search (70% vector / 30% text):
```sql
HYBRID SEARCH (
    VECTOR [1.2, 0.8, -0.2] WEIGHT 0.7,
    TEXT "machine learning" WEIGHT 0.3
)
FROM research.paper_vectors
WHERE publication_year > 2020;
```
Robust representative-vector query using geometric median instead of centroid:
```sql
SELECT GEOMETRIC_MEDIAN(feature_vector) AS representative
FROM analytics.product_features
GROUP BY product_category;
```

## Key Takeaways
1. VQL is a proposal/thought experiment, not shipping software — treat its syntax as illustrative, not installable.
2. The real problem VQL responds to is genuine: vector DB vendors currently lack a shared abstraction layer, unlike SQL for relational databases.
3. Hybrid search fusion generalizes cleanly to a declarative form: weighted, normalized combination of vector and text scores, weights summing to 1.0.
4. Choose centroid for speed/simplicity when data is clean; choose geometric median or medoid when outlier robustness matters — the "no squared term" difference is the whole story.
5. Range search (threshold-bounded) and batch search (many queries, one call) are useful primitives beyond simple top-K similarity search.

## Connects To
- **Ch 6**: VQL's hybrid search fusion formula is a declarative restatement of the imperative `hybrid_search` function there.
- **Ch 1, Ch 3**: VQL's data model (Collection/Table/Vector/Field/Metadata/Distance metric) formalizes concepts introduced informally in the earlier chapters.
- **SQL (external concept)**: VQL is explicitly modeled on SQL's DML syntax, deliberately deferring DDL/indexing concerns.
