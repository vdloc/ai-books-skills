# Cheatsheet: Hands-On RAG for Production

## Decision Rules

- **If retrieval recall < ~60-70%, don't tune the prompt.** Fix retrieval (hybrid search, reranking) first — no prompt engineering compensates for missing context.
- **If a query needs time-bound facts, multi-constraint intersection, or multi-hop chains** → vector/hybrid search will underperform. Consider a knowledge graph (chunk enrichment first, hybrid-graph retrieval second) or agentic RAG.
- **If you need a single agent vs. multi-agent** → default to single agent. Escalate to multi-agent only for: distinct security domains, vast tool surfaces (tool dilution), or organizational ownership boundaries.
- **If choosing image strategy** → reasoning over precise data (charts, tables-in-images) → summarization. Concept/aesthetic visual search → shared embedding (CLIP/SigLIP).
- **If a leading question could bias a VLM** (e.g. "where's the damage?") → use blind verification (neutral prompt first) before trusting the answer.
- **If deciding DIY vs. RAG platform** → evaluate per-component (embedding model, vector DB, retrieval, prompt, LLM, hallucination detection), not as one binary choice. More than one RAG use case in your org → sprawl risk tilts toward a platform.
- **If deciding whether to invest in a knowledge graph** → need ≥4/6 "yes" on: recurring factual failures in evals, deterministic-grounding requirement, existing licensable ontology, relationship-centric data value, long-term team ownership, clear business ROI.
- **If compliance requires air-gapped/on-prem** → SaaS RAG platforms are a nonstarter regardless of feature quality — check deployment model fit before features.
- **If PII must be removed from ingested text** → use entity-aware/typed redaction (`[PERSON]`), never blanket masking/nulling (destroys context needed for the answer).
- **If a new component (embedding model, reranker, prompt) is about to ship** → it must pass a CI/CD evaluation gate (no faithfulness/relevance regression, P95 latency +≤5%) against a versioned benchmark.

## Thresholds & Defaults

| Parameter | Typical value | Source |
|---|---|---|
| Retrieval latency target (semantic/hybrid/rerank) | ≤300ms average | Ch4 |
| Generation latency (non-reasoning models) | 2-5s | Ch4 |
| Uptime target | ≥99.9-99.99% | Ch4, Ch6 |
| Semantic cache similarity threshold | ~0.85 cosine | Ch4 |
| Online eval sampling rate | 5-10% of traffic | Ch6 |
| CI/CD latency regression budget | ≤5% P95 increase | Ch6 |
| Chunk size (general text) | 500-1000 chars, ~10-20% overlap | Ch2 |
| Embedding model context window (typical) | 8k tokens (OpenAI, Gemini) | Ch2 |
| DIY RAG TCO overrun (actual vs. projected) | 3-5x | Ch4 |
| KG project failure rate (from-scratch, no standard ontology) | >50% (practitioner estimate) | Ch9 |

## Trade-off Matrices

**Retrieval metric choice**
| Need | Metric |
|---|---|
| Find one good answer fast | MRR |
| Rank multiple relevant chunks well | MAP |
| Graded relevance, position-sensitive | nDCG |
| No golden chunks available | UMBRELA (reference-free) |

**KG integration pattern**
| Query shape | Pattern |
|---|---|
| Semantic match, missing metadata (names, dates) | Chunk enrichment (low risk, low latency) |
| True relational/multi-hop discovery | Hybrid-graph retrieval (higher risk, more complex) |
| Broad sensemaking over full corpus | Microsoft GraphRAG (very high cost, not real-time) |

**RAG platform deployment**
| Priority | Model |
|---|---|
| Speed, ease, low ops burden | SaaS |
| Cloud-native compliance + some control | VPC |
| Max control, air-gapped, highest sensitivity | On-premises |

## Tells & Smells

- **Answer admits "I don't know" but pads with irrelevant detail** → likely a retrieval recall failure (missed a key chunk), not a hallucination — check recall before blaming generation.
- **Response is faithful to context but still "wrong" for the user** → likely stale/outdated ingested data (fix ingestion/versioning), not an LLM fault.
- **Judge scores drift over time with no system change** → evaluation drift — your judge prompt/model version probably changed, not your RAG system.
- **An agent makes an unexpectedly high number of tool/LLM calls for a simple task** → likely stuck in a loop or following a convoluted reasoning path.
- **Multiple entity nodes for what's clearly one real-world thing** ("007," "Bond," "James Bond") → entity linking failure ("entity explosion") in your KG pipeline.
- **A table's data cells retrieved without their column headers** → naive text-chunking of a table; switch to structured JSON/dataframe context.
- **VLM confidently confirms whatever the user's leading question implies** → visual sycophancy; use blind/neutral-prompt verification.
- **Query volume spikes then decays over 2-3 weeks post-launch** → canary for poor answer quality or latency frustration; investigate via logs + thumbs-up/down before assuming natural adoption plateau.
- **Costs balloon unpredictably in production vs. POC estimate** → expected (3-5x common); should have budgeted with granular cost monitoring + alerts from day one.
