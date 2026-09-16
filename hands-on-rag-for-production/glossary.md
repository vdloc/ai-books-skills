# Glossary

**A2A (Agent2Agent)** — open protocol (Google/Linux Foundation) for agents to discover each other's capabilities via "Agent Cards" and collaborate across vendors (Ch7).

**ANN (Approximate Nearest Neighbor)** — search algorithms (e.g. HNSW) that trade small accuracy loss for massive speed gains vs. exact/brute-force search (Ch2).

**AutoNuggetizer** — reference-free generation metric: decomposes relevant chunks into "nuggets" (Vital/OK), checks if the generated answer covers them (Supported/Partially/Not) (Ch6).

**BM25** — the dominant lexical/keyword search ranking function, successor to TF-IDF (Ch2, Ch3).

**Blur-on-ingest** — model-based redaction of faces/signatures/PII in images before they hit the vector store (Ch8).

**Candidate generation** — stage 1 of the two-stage retrieval pipeline, optimized for recall (Ch3).

**Chunk enrichment** — using a knowledge graph to add missing facts/context to a vector-search-retrieved chunk (Ch9).

**Chunking** — splitting parsed text into smaller pieces sized for LLM/embedding context windows (Ch2).

**Context engineering** — dynamically assembling the optimal prompt (facts + history + instructions) rather than just wording a static prompt (Ch10).

**Corpus** (Vectara) — an isolated data container for one RAG application's ingested content (Ch5).

**Cypher** — Neo4j's graph query language, pattern-matching syntax that mirrors the graph visually (Ch9).

**Entity linking (record linkage)** — merging different surface forms ("IBM," "I.B.M.") of the same real-world entity into one canonical KG node (Ch9).

**Faithfulness (factual consistency)** — whether every statement in a response is verifiable from retrieved chunks; core hallucination metric (Ch6).

**Federated retrieval** — querying enterprise data in its native systems (Snowflake, Elasticsearch) rather than centralizing it all into one vector store (Ch10).

**GraphRAG** (Microsoft) — LLM-built knowledge graph + community detection + hierarchical summarization, purpose-built for broad "sensemaking" queries (Ch9).

**Guardrails** — pipeline safeguards (bias filters, prompt-injection defense, hallucination detection) ensuring safe, policy-compliant RAG outputs (Ch1, Ch3).

**Hallucination** — LLM output unsupported by or contradicting retrieved context or facts (Ch1, Ch3, Ch6).

**HHEM (Hughes Hallucination Evaluation Model)** — dedicated classifier scoring factual consistency 0-1, faster/cheaper than LLM-as-judge (Ch3, Ch6).

**HNSW (Hierarchical Navigable Small World)** — the dominant ANN algorithm, multi-layer graph search (Ch2).

**Hybrid search** — combining semantic (vector) and lexical (BM25) search, fused via RRF or weighted scoring (Ch2, Ch3).

**Hybrid-graph retrieval** — LLM translates a query into Cypher/SPARQL, merges graph results with vector search chunks (Ch9).

**Idempotency** — a pipeline task can be safely retried without side effects (Ch3, Ch9).

**Instruction defense** — using delimiters (XML tags) + explicit prompt instructions to prevent prompt injection (Ch3).

**Knowledge graph (KG)** — a network of entities (nodes) and typed relationships (edges) encoding deterministic facts (Ch9).

**LLM-as-a-judge** — using an LLM to score another LLM's output against defined criteria (Ch3, Ch6).

**MCP (Model Context Protocol)** — open standard for how agents discover/call tools and access data via Tools, Resources, Prompts primitives (Ch7).

**MMR (Maximum Marginal Relevance)** — reranking technique balancing relevance with diversity to reduce redundant chunks (Ch3).

**Modality alignment** — maintaining traceable links between text, image, table, video components of a source document (Ch8).

**MRR / MAP / nDCG** — rank-aware retrieval metrics: mean reciprocal rank (first relevant hit), mean average precision (precision at each relevant hit), normalized discounted cumulative gain (graded relevance, position-weighted) (Ch6).

**Multi-agent system** — multiple specialized agents collaborating (supervisor or orchestrator-worker topology) vs. a single agent (Ch7).

**Ontology vs. schema** — ontology = abstract domain rules ("a Person can DIRECT a Movie"); schema = concrete DB implementation of those rules (Ch9).

**Orphan chunk** — a vector that has lost its link to spatial/temporal source metadata, unusable for citation (Ch8).

**ReAct (Reason + Act)** — LLM prompting paradigm interleaving thought → action (tool call) → observation in a loop (Ch7).

**Reciprocal Rank Fusion (RRF)** — combining semantic + lexical result rankings without needing score normalization (Ch3).

**Reranking** — second-pass relevance scoring (cross-encoder or business logic) applied to a retrieved candidate set (Ch2, Ch3).

**RAG (Retrieval-Augmented Generation)** — augmenting LLM generation with retrieved external facts rather than relying solely on parametric/training knowledge (Ch1).

**RAG platform (RAG-as-a-service)** — a managed, end-to-end service bundling ingestion, retrieval, generation, and hallucination detection behind one API (Ch5).

**RAG sprawl** — proliferation of independently managed, incompatible RAG stacks across an organization (Ch5).

**Semantic caching** — caching keyed on query-embedding similarity rather than exact string match (Ch4).

**Semantic vs. lexical search** — vector-embedding similarity (meaning) vs. keyword/token matching (exact form) (Ch2).

**SLM (Small Language Model)** — efficient models (often <32B params) enabling local, air-gapped, cost-predictable RAG inference (Ch10).

**SPARQL** — W3C standard query language for RDF triple stores (Ch9).

**Tool calling / function calling** — LLM capability to emit a structured request (name + typed args) for an orchestration layer to execute (Ch7).

**Tool dilution** — degraded tool-selection accuracy from too many tools in one agent's context (Ch7).

**Two-stage retrieval pipeline** — candidate generation (recall-optimized) followed by reranking (precision-optimized) (Ch3).

**UMBRELA** — reference-free retrieval evaluation: LLM judge scores each chunk 0-3 for relevance without needing golden chunks (Ch6).

**Vector database** — system for storing, indexing, and similarity-searching embeddings, with metadata filtering and persistence (Ch2).

**VLM (Vision-Language Model)** — a model that processes both visual and textual input, used for multimodal parsing, summarization, and judging (Ch2, Ch8).
