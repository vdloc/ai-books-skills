# Chapter 5: The RAG Platform

## Core Idea
A RAG platform (RAG-as-a-service / turnkey RAG) bundles document extraction, chunking, embedding, vector storage, retrieval, generation, and hallucination detection behind a single API — trading DIY's granular control and lower direct API costs for speed, reduced operational burden, centralized governance, and TCO predictability.

## Frameworks Introduced
- **DIY vs. Platform trade-off, component-by-component**: for each RAG component (embedding model, vector DB, advanced retrieval, prompt engineering, LLM support, hallucination detection), DIY gives control + responsibility; platform gives abstraction + vendor dependency. Evaluate each component separately rather than treating "DIY vs. platform" as one binary decision.
- **RAG sprawl / centralized governance model**: analogous to "shadow IT" → "shadow AI" — independently built DIY RAG apps across departments create policy drift, duplicated infrastructure spend, and fragmented security surfaces. A platform provides a single "golden path" (approved vector store, automatic PII redaction) enforced org-wide.
- **Two RAG platform pricing models**: developer-centric consumption (free tier + low subscription, e.g. $50–500/mo, variable pay-as-you-go overages — e.g. Ragie.ai, LlamaCloud) vs. all-in-one enterprise (annual subscription, bundled "credits" covering API/storage/compute/retrieval as one predictable unit — e.g. Vectara). Pick based on whether your priority is low entry cost or TCO predictability.
- **Three deployment models**: SaaS (vendor-managed, fastest, least control), VPC (isolated cloud segment, middle ground, more MLOps expertise needed), on-premises/air-gapped (maximum control and data isolation, requires local LLM inference + local parsing/embedding/reranking stack, highest operational overhead). Map to your data-sensitivity and compliance requirements, not just cost.

## Key Concepts
- **Corpus** (Vectara terminology): an isolated virtual container of ingested, pre-processed data — separate corpora for separate applications/datasets (e.g. internal docs vs. support tickets vs. reviews).
- **BYO (bring-your-own) model support**: a platform's willingness to let you swap in your own embedding model or fine-tuned LLM instead of the platform default — check this if you anticipate needing a specialized model later, since switching embedding models later requires re-encoding your entire dataset.
- **RAG sprawl**: proliferation of independently managed, incompatible RAG stacks across an organization, causing policy drift and duplicated cost.
- **Factual consistency score**: a per-response hallucination score returned directly by the query API (e.g. Vectara's `factual_consistency_score`), letting the caller flag/filter low-confidence responses without a separate evaluation call.
- **Hallucination correction API**: a dedicated endpoint that takes a (possibly hallucinated) generated response plus the source chunks and returns a corrected response with per-span explanations of what was wrong and why.
- **"Platform of services" middle ground**: cloud-native offerings like Amazon Bedrock Knowledge Bases, Vertex AI Search, or Azure AI Search that simplify DIY (managed components) but still require you to select, configure, and orchestrate them yourself — distinct from a true end-to-end platform (Vectara, Nuclia) that hides this behind one API.

## Mental Models
- Draw the database-vendor analogy: almost nobody builds their own database engine anymore — they pay Oracle/Snowflake/Databricks and focus on the application layer. RAG platforms represent the same specialization emerging for retrieval/generation infrastructure.
- When evaluating a platform's data connectors, don't just check "do they support X source" — check *how* (e.g. does the Gmail connector cover Outlook too? does the Jira connector import attachments, or just tickets?). Coverage claims are often shallower than they look.
- Treat "RAG platform vs. DIY" as a build-vs-buy decision scoped per organization, not per project: if you already have >1 RAG use case, sprawl risk and duplicated infra cost tilt the calculus toward a platform even if a single DIY project would have been cheaper standalone.

## Anti-patterns
- **Choosing an embedding model/vector DB in isolation from downstream cost**: higher-dimensional embeddings can modestly improve accuracy but significantly raise storage, indexing, and query compute cost — especially painful to discover *after* committing to a DIY infrastructure choice.
- **Assuming DIY is cheaper because "direct API costs are lower"**: ignores setup/maintenance labor, security patching, model re-engineering, and the compounding TCO of RAG sprawl once more than one team builds RAG independently.
- **Deploying to SaaS when compliance requires air-gapped/on-prem**: if your policy forbids any data leaving your environment, SaaS RAG platforms are a nonstarter regardless of their compliance certifications (HIPAA/GDPR/SOC-2) — verify deployment model fit before evaluating features.
- **Not verifying connector granularity before committing to a platform**: a claimed "600+ connectors" catalog can still miss the specific data you need to refresh incrementally or the specific sub-resource (e.g. Jira ticket attachments) your use case requires.

## Reference Tables

| Open source data connector project | Approx. total connectors | Data refresh support |
|---|---|---|
| LangChain | 130+ | External schedulers + vector store ops |
| LlamaIndex | 160+ | LlamaCloud supports incremental updates |
| Airbyte | 600+ | Built-in incremental sync (cursor, CDC), scheduling |
| Meltano | 600+ | Yes, if a Singer tap implements it |
| Datavolo (Snowflake) | 300+ | NiFi-based true incremental fetching |

| Deployment model | Control | Compliance fit | Overhead |
|---|---|---|---|
| SaaS | Lowest | Vendor-certified (HIPAA/GDPR/SOC-2) but data leaves perimeter | Lowest |
| VPC | Medium | Good for cloud-native compliance needs | Medium (cloud/MLOps expertise needed) |
| On-premises/air-gapped | Highest | Required for air-gapped/highly regulated environments | Highest (local LLM inference, local parsing/embedding/reranking) |

## Code Examples

```python
# Vectara: query with hybrid search, reranker, and factual consistency scoring
url = f"https://api.vectara.io/v2/corpora/RAGBOOK/query"
payload = json.dumps({
    "query": "Are pets allowed in the office?",
    "search": {
        "lexical_interpolation": 0.025,
        "limit": 50,
        "context_configuration": {"sentences_before": 2, "sentences_after": 2},
        "reranker": {"type": "customer_reranker", "reranker_name": "Rerank_Multilingual_v1"},
    },
    "generation": {
        "max_used_search_results": 7,
        "response_language": "eng",
        "prompt_name": "vectara-summary-ext-24-05-med-omni",
        "enable_factual_consistency_score": True,
    },
})
response = requests.post(url, headers=headers, data=payload)
res = response.json()
print(res['summary'])
print(f"Factual Consistency Score: {res['factual_consistency_score']}")
```
- **What it demonstrates**: a single API call performs hybrid search (`lexical_interpolation`), reranking, generation, and hallucination scoring — the multi-service pipeline from Chapters 2–3 collapsed into one request.

```python
# Vectara: hallucination correction with per-span explanations
url = "https://api.vectara.io/v2/hallucination_correctors/correct_hallucinations"
payload = json.dumps({
    "generated_text": hallucinated_response,
    "documents": [{"text": r["text"]} for r in search_results],
    "model": "vhc-large-1.0",
})
res = requests.post(url, headers=headers, data=payload).json()
print(res['corrected_text'])
print(res['corrections'])
# [{'original_text': '...no rules to follow...', 'corrected_text': '...specific guidelines to follow...',
#   'explanation': 'The response incorrectly claims there are no rules...'}, ...]
```
- **What it demonstrates**: correction returns not just a fixed response but a structured diff of exactly which spans were wrong and why — auditable, unlike a black-box rewrite.

## Worked Example
Setting up Vectara end-to-end: create a corpus (`RAGBOOK`) with the built-in Boomerang embedding model → upload `pet_policy.pdf` via `upload_file` (Vectara auto-extracts text, applies sentence chunking, embeds, and stores vectors+text+metadata) → separately ingest structured text (Shakespeare's *King Lear*, nested into sections with per-section metadata like `stage-instructions`) via the direct-ingestion `/documents` endpoint → query `"Are pets allowed in the office?"` and get back a grounded summary plus a factual consistency score (0.777) → deliberately test hallucination correction on a response that wrongly claims "snakes" and "no rules," which the `correct_hallucinations` endpoint fixes with explanations pointing to exactly which claims were unsupported. This walkthrough shows the entire DIY pipeline from Chapters 2–4 (parse → chunk → embed → store → hybrid-search → rerank → generate → detect/correct hallucination) compressed into roughly five API calls.

## Key Takeaways
1. Evaluate DIY vs. platform per RAG component, not as one monolithic decision — some components (embedding models) are low-differentiation and safe to outsource; others (custom retrieval logic for a niche domain) may justify DIY.
2. RAG sprawl is a real organizational cost once more than one team builds RAG independently — duplicated infra spend and fragmented security postures compound quickly, similar to shadow IT.
3. Platform pricing splits into developer-consumption (cheap to start, variable cost) vs. enterprise-bundle (predictable TCO) — pick based on whether cost predictability or low entry cost matters more.
4. Deployment model (SaaS/VPC/on-prem) must be chosen from compliance/data-sensitivity requirements first, features second — a great platform is a nonstarter if it can't meet your air-gap requirement.
5. Don't take a connector catalog's headline count at face value — verify format coverage, refresh/incremental-sync support, RBAC/permission syncing, and error/partial-failure handling for your *specific* sources.
6. A platform's per-response factual consistency score and structured hallucination-correction output (with explanations) can replace a chunk of the custom evaluation infrastructure you'd otherwise build in-house (see Chapter 6).
7. The database-vendor analogy is apt: RAG platform adoption reflects the same specialization curve that made "build your own database engine" rare — most teams gain more by focusing on their RAG application's data and business logic than by re-deriving retrieval infrastructure.

## Connects To
- **Ch2/Ch3**: every DIY component discussed here (embedding models, vector DBs, hybrid search, reranking, hallucination detection) maps directly back to the base and advanced RAG stack chapters.
- **Ch4**: the TCO, team-expertise, and vendor-chaos problems raised there are the direct motivation for considering a platform here.
- **Ch6**: the factual consistency score and correction API introduced here are a preview of the full RAG evaluation framework covered next.
