# Chapter 9: Knowledge-Enhanced RAG

## Core Idea
Vector/hybrid search excels at conceptual similarity but fails on time-bound facts, multi-constraint intersections, and multi-hop reasoning; knowledge graphs (KGs) fix this by making relationships deterministic and traversable — at a steep, often-underestimated cost in data engineering, entity linking, and infrastructure that must be justified against actual business ROI.

## Frameworks Introduced
- **Three failure classes semantic search can't solve**: time-bound facts (embeddings blur past/present — "who was CEO of Twitter in October 2022?"), intersection of multiple constraints (retrieval surfaces chunks about each constraint separately, rarely one chunk covering both — "drugs interacting with both warfarin AND grapefruit juice"), and chained/multi-hop reasoning (vector search finds topical similarity but can't follow a relationship chain — "lead actor in the movie directed by the Inception director"). Recognize these query shapes as your signal that a KG (or an agent, Ch7) may be needed.
- **Chunk enrichment vs. hybrid-graph retrieval** (two KG integration patterns): chunk enrichment = standard vector search finds a "thin" chunk (missing name/context), then a graph lookup on entities found in that text adds missing facts (metadata-first, low latency, near-zero risk — the "best bang for your buck" starting point). Hybrid-graph retrieval = an LLM translates the user's natural-language question directly into Cypher/SPARQL, executes it against the graph, and merges those results with vector-search chunks (discovery-first, handles true relational/multi-hop queries, but carries text-to-query hallucination risk and higher latency).
- **Ontology vs. schema**: an ontology is the abstract domain model ("a Person can DIRECT a Movie," the "rules of reality"); a schema is the database-level implementation (concrete node labels, property types) built from that ontology. The ontology helps the LLM reason about traversal logic; the schema is what you feed the LLM (as text) so it can generate correct Cypher/SPARQL.
- **Automated KG construction (3 steps)**: entity identification (find/categorize key nouns → nodes) → relation extraction (find connections between them → edges/triples) → entity linking (merge different surface forms of the same real-world thing into one canonical node). An LLM can do steps 1-2 in one prompt, but entity linking remains the hard, unsolved bottleneck.
- **GraphRAG (Microsoft's specific approach)**: build a KG from source text with an LLM → run community detection to find clusters of densely connected entities → summarize each community hierarchically (fine-grained to broad) → at query time, generate partial answers from relevant community summaries, then synthesize a final answer from those partial answers. Purpose-built for "sensemaking" queries (query-focused summarization, QFS) that need a *global* view of the dataset, not a few retrieved snippets.
- **KG adoption decision checklist** (6 questions — need ≥4 "yes" to justify the investment): factual failure (do multi-hop/time-bound/constrained queries actually fail in your evals?), grounding necessity (does the domain require deterministic, not probabilistic, answers — legal/medical?), available assets (is there a licensable ontology/KG like FIBO/DrugBank to leverage instead of building from scratch?), data connectivity (does the value live in relationships between documents, not document content?), long-term ownership (is there a team to own schema evolution and ETL indefinitely?), business ROI (are accuracy gains tied to a measurable business outcome?).

## Key Concepts
- **Nodes and edges**: nodes = entities (Person, Movie, Company); edges = typed relationships between them (ACTED_IN, DIRECTED, FOUNDED_BY). Querying a KG means traversing known paths, not computing similarity.
- **Cypher**: Neo4j's pattern-matching query language — `(p:Person)-[:DIRECTED]->(m:Movie)` reads almost like the graph itself.
- **SPARQL**: the W3C standard query language for RDF triple stores (subject-predicate-object statements) — more declarative, less visually intuitive than Cypher, but a formal open standard.
- **Entity explosion**: when a single real-world entity ("James Bond," "007," "Bond") gets split across multiple graph nodes instead of merged into one — the direct symptom of unsolved entity linking.
- **Entity linking (record linkage)**: matching different surface forms of the same entity (via normalization + candidate generation, e.g. Jaro-Winkler string distance) to a single canonical node — the hardest part of automated KG construction, especially at enterprise scale with messy data.
- **Local search vs. global search** (GraphRAG query modes): local search = "bottom-up," starts from seed nodes matching the query and walks outward through nearby edges (good for specific, localized questions); global search = "top-down," scans the whole graph/community structure for relevant themes even in disconnected parts (good for broad, synthesizing questions).
- **Standard/licensed ontologies**: FIBO (finance), DrugBank (pharma) — adopting an existing ontology avoids the common failure mode of trying to model "everything" in a domain from scratch (>50% of from-scratch KG projects reportedly fail).
- **CDC vs. event-driven KG updates**: CDC (e.g. Debezium reading a Postgres transaction log) for structured source data; event-driven (S3 upload triggers a Lambda extraction pipeline) for unstructured documents — both feed incremental `MERGE`/`DELETE` operations rather than full graph rebuilds.
- **Tombstoning**: marking a fact as inactive/expired (`status: "inactive"` on a `:WORKS_AT` edge) rather than deleting it outright, to preserve historical context while letting queries filter to current-state facts.
- **Idempotent graph ingestion**: using `MERGE` instead of `CREATE` so a failed-and-restarted ingest job produces the same graph state as a successful one — necessary because wrapping a full graph build in one ACID transaction is often infeasible at scale.

## Mental Models
- Treat KG adoption as a classic accuracy-vs-cost engineering trade-off, not a default upgrade — "don't underestimate vanilla RAG," which is often sufficient and dramatically cheaper for general Q&A and summarization.
- Use your evaluation framework (Ch6) as the *evidence generator* for this decision: analyze which query types fail under vanilla/hybrid RAG, and only pilot KG-enhanced RAG for those specific failure classes — don't build a KG speculatively.
- Distinguish "GraphRAG" as a generic term (any graph-aided RAG) from GraphRAG as Microsoft's specific community-detection-based system — the latter is a heavy, expensive, sensemaking-specific tool, not a synonym for "use a graph."
- Think of chunk enrichment as a targeted, low-risk patch (annotate what vector search already found) and hybrid-graph retrieval as a structural upgrade (let the graph discover connections vector search would never find) — start with the patch, escalate to the upgrade only when query patterns demand true relational discovery.

## Anti-patterns
- **Building a from-scratch, general-purpose ontology trying to model "everything" in a domain**: the leading cause of KG project failure (>50% by one practitioner's estimate) — scope the ontology to your specific use case, or better, adopt a standard one (FIBO, DrugBank).
- **Relying purely on LLM-driven triple extraction without validation**: noisy and domain-dependent on its own — production systems need a hybrid strategy layering deterministic heuristics (regex for IDs/dates), classical NLP validation, and human-in-the-loop review for critical entities.
- **Wrapping a full graph ingestion build in a single ACID transaction**: locks the database for hours and risks memory exhaustion at scale — use idempotent `MERGE` operations with batched commits instead.
- **Deleting expired facts outright**: destroys historical context needed for audit/compliance — tombstone with a status flag and filter at query time instead.
- **Deploying Microsoft's GraphRAG for rapidly changing corpora, latency-sensitive queries, or use cases needing verbatim citations**: its expensive multi-stage LLM preprocessing (indexing 20 movies took ~45 minutes and $6.32; scaling to 1,800 movies estimated at $560+) makes it unsuitable for frequently updated data, subsecond-response needs, or legal/compliance contexts requiring exact source text.
- **Underestimating ongoing KG maintenance cost**: schema evolution (no `ALTER TABLE` equivalent), ETL data-integrity risk, and RBAC at the node/relationship level (e.g. traverse a Company node but block the connected Employee's Salary property) are persistent operational burdens, not one-time setup costs.

## Code Examples

```cypher
// Cypher: find who directed Oppenheimer
MATCH (p:Person)-[:DIRECTED]->(m:Movie)
WHERE m.title = 'Oppenheimer'
RETURN p.name
```
- **What it demonstrates**: Cypher's pattern syntax visually mirrors the graph — nodes in parens, relationships in brackets, arrow shows direction.

```python
# Chunk enrichment: vector search finds a "thin" chunk, KG lookup adds missing facts
# Chunk found via vector search mentions "Vincent" saying "Royale with Cheese" but
# neither the movie title nor the actor's real name appear in the raw text.
# Graph lookup on the Character entity found in the chunk:
#   Character: Vincent -> IS_REAL_NAME: Vincent Vega
#   Character: Vincent -> PLAYED_BY: John Travolta
#   Character: Vincent -> APPEARS_IN: Pulp Fiction
# The enriched context (chunk text + these facts) lets the LLM answer with certainty,
# where the raw chunk alone left the movie/actor unknown.
```
- **What it demonstrates**: the core value of chunk enrichment — transforming a semantically-correct-but-underspecified retrieval into a fully answerable one via a cheap, indexed graph lookup.

```cypher
// Hybrid-graph retrieval: LLM-generated Cypher for a relational query
// "What are all the characters in GoldenEye? Which interacted with Bond?"
MATCH (m:Movie) WHERE toLower(m.title) CONTAINS 'goldeneye'
WITH m
MATCH (c:Character)-[:APPEARS_IN]->(m)
WITH m, collect(DISTINCT c.name) AS all_characters, m.title AS movie_title
MATCH (ch:Chunk)-[:MENTIONS]->(bond_char:Character)
WHERE toLower(bond_char.name) CONTAINS 'bond'
MATCH (ch)-[:MENTIONS]->(other_char:Character) WHERE other_char <> bond_char
WITH movie_title, all_characters,
     collect(DISTINCT other_char.name) AS interacted_with_bond,
     collect(DISTINCT ch.text)[0..10] AS chunks
RETURN movie_title, all_characters, interacted_with_bond, chunks
```
- **What it demonstrates**: an LLM-generated multi-hop query that vector search alone cannot answer — combining these graph-retrieved chunks with vector-search chunks lets the RAG pipeline produce a complete, correct character list, where vector search alone returned "I cannot answer this question."

## Reference Tables

| Approach | Best for | Key limitation | Cost & complexity |
|---|---|---|---|
| Standard RAG (vector search) | General Q&A, similarity retrieval | Struggles with multi-hop, time-bound, exact-match queries | Low |
| Hybrid (vector + BM25) | Balancing semantic + exact match | Still struggles with multi-hop/deep relationships | Low-Medium |
| KG-hybrid RAG | Precise multi-constraint factual queries | Requires building/maintaining a live KG | High |
| Microsoft GraphRAG | Broad "sensemaking" queries over large corpora | Very high upfront cost, not real-time, lossy for verbatim citation | Very High |
| Agentic RAG (Ch7) | Multi-hop logic, iterative research | High latency, can loop/hallucinate tool paths | Medium-High |

| Feature | Chunk enrichment | Hybrid-graph retrieval |
|---|---|---|
| Primary goal | Contextualize a specific snippet | Find relationships/aggregations |
| Ideal for | Factual grounding (names, dates, IDs) | Multi-hop reasoning ("who met whom?") |
| Runtime risk | Low (indexed lookup) | Higher (query hallucination, slow joins) |
| Latency | Minimal (sub-ms) | Variable |
| Implementation | Straightforward, good MVP | Complex, needs text-to-Cypher tuning |

| Graph DB deployment | Best when |
|---|---|
| Traditional server/cluster (Neo4j, TigerGraph) | Strict on-prem/regulatory needs, deep engine customization required |
| Managed cloud (Neo4j Aura, Amazon Neptune) | Need HA/multitenancy but lack in-house graph DB expertise |
| Embedded library (Kuzu, DuckDB) | KG is small (<10-20GB), mostly read-only, refreshable via CI/CD |

## Worked Example
Building a movie knowledge graph from two sources: IMDb (clean, pre-structured — Movie/Person/Character/Genre nodes with DIRECTED/ACTED_IN/HAS_GENRE edges) and MovieSum movie scripts (messy — requires regex-based character-name extraction from all-caps script conventions, then filtering false positives like "THE" or "AND" that appear ≥3 times). Chunks from scripts get `(Chunk)-[:MENTIONS]->(Character)` and `(Character)-[:APPEARS_IN]->(Movie)` edges layered on top of the IMDb-derived graph. This dual-source construction — one clean, one messy — is explicitly framed as the *best-case* scenario for KG building, since movie scripts have predictable all-caps character-name formatting; the book notes that real enterprise data (contracts, tickets, logs) lacks this convenient structure entirely, making entity extraction dramatically harder in production than in this demo.

## Key Takeaways
1. Reach for a KG only when your evaluation results (Ch6) show recurring, specific failures on time-bound, multi-constraint, or multi-hop queries — not as a default upgrade to "better" RAG.
2. Chunk enrichment is the low-risk, high-value entry point; hybrid-graph retrieval and Microsoft's GraphRAG are progressively more powerful but progressively more expensive and operationally demanding.
3. Entity linking (merging "IBM," "I.B.M.," "International Business Machines Corp." into one node) is the hardest, most underestimated part of KG construction — LLMs help with extraction but don't solve this alone.
4. Prefer adopting a standard ontology (FIBO, DrugBank) or licensed pre-resolved KG over building an ontology from scratch — most from-scratch KG projects fail from over-scoping, not lack of tooling.
5. GraphRAG's community-detection approach is purpose-built for broad sensemaking queries over large, relatively static corpora — it's the wrong tool for real-time data, low-latency needs, or verbatim-citation requirements.
6. KG maintenance (idempotent ETL, schema evolution without `ALTER TABLE`, node/relationship-level RBAC, tombstoning expired facts) is an ongoing operational cost comparable to running any other stateful production database — budget for it as such.
7. Use the 6-question adoption checklist before committing — needing fewer than 4 "yes" answers signals the KG's operational tax likely outweighs its accuracy benefit at your current stage.

## Connects To
- **Ch2/Ch3**: KG integration is explicitly framed as "the same base RAG stack, with a graph DB layered alongside the vector DB" — either co-locating embeddings in graph nodes or keeping them separate and synchronized.
- **Ch6**: the evaluation framework is the primary tool for deciding *whether* to invest in KG-enhanced RAG — identify failing query patterns first, then pilot a KG solution.
- **Ch7**: agentic RAG is presented as an alternative way to solve multi-hop/multi-constraint queries (via iterative tool use) without building a KG — the two approaches are complementary options, not mutually exclusive.
- **Ch4**: KG infrastructure (idempotent ETL, RBAC, backup/DR) requires the same production-deployment discipline covered there, applied to a stateful graph database instead of (or alongside) a vector database.
