# Patterns

## Pipeline Pattern
**When to use**: any AI system with a nontrivial data-ingestion stage (collection, cleaning, harmonization, transformation, feature extraction).
**How**: organize ingestion into a structured, sequential workflow with clear stage boundaries.
**Trade-offs**: adds structure/overhead but prevents ingestion-layer failures that are hard to debug downstream.

## Model Factory Pattern
**When to use**: need to produce and deploy multiple model versions consistently (e.g., per-language classifiers).
**How**: automate training, packaging, and deployment of model variants via CI/CD.
**Trade-offs**: requires upfront pipeline investment; pays off once you have 3+ variants to maintain.

## Microservice Architecture Pattern
**When to use**: pipeline stages have different scaling needs (e.g., inference vs. ingestion).
**How**: decompose into independently deployable services communicating via APIs.
**Trade-offs**: gains independent scaling/fault isolation; costs orchestration complexity and inter-service communication overhead.

## A/B Testing vs. Shadow Testing
**When to use**: A/B for UX-facing model changes; Shadow for compliance-sensitive or high-risk upgrades.
**How**: A/B splits live traffic across versions; Shadow runs the new model silently alongside production, comparing outputs without user exposure.
**Trade-offs**: A/B needs load-balancing/routing infra; Shadow needs dual inference pipelines but zero user risk.

## Federated Learning Pattern
**When to use**: cross-region/cross-org data-privacy constraints (GDPR/CCPA) prohibit centralizing raw data.
**How**: train locally per region, share only model updates with a central/anchor node.
**Trade-offs**: strong privacy guarantee; requires synchronization/coordination overhead and careful update-merge policy.

## Differential Privacy Pattern
**When to use**: need to publish aggregate statistics without exposing individual records.
**How**: add calibrated mathematical noise to data/outputs so no individual's presence can be inferred.
**Trade-offs**: protects only information framed as "private" — general/aggregate facts still leak; noise reduces precision.

## Blue-Green / Canary Deployment
**When to use**: any production model update where downtime or bad-rollout risk must be minimized.
**How**: Blue-Green runs old+new in parallel for instant rollback; Canary exposes the new version to a small user subset first.
**Trade-offs**: Blue-Green needs double the running infra briefly; Canary needs traffic-splitting infra and slower full rollout.

## Zero Trust Pattern
**When to use**: any system handling regulated data (HIPAA/PCI-DSS/ISO 27001).
**How**: default-deny all access; grant permissions strictly per verified need, no implicit trust between services.
**Trade-offs**: stronger security posture at the cost of more access-control configuration and potential friction.

## Agent Registry & Discovery
**When to use**: systems with more than a handful of agents that need dynamic composition.
**How**: maintain a structured catalog (name, task, I/O schema, engine, capabilities, endpoint, data-residency) that orchestrators query to find the right agent.
**Trade-offs**: prevents duplicated development and enables compliance-aware selection; requires disciplined metadata upkeep.

## Self-Reflection Pattern
**When to use**: quality-critical generation tasks (code, compliance summaries) where prompt engineering alone isn't reliable enough.
**How**: pair a generator agent with a reviewer/critic step (self-critique or a separate agent) that scores output against a checklist and triggers regeneration until a threshold is met.
**Trade-offs**: meaningfully improves accuracy; adds working-memory load, latency, and LLM-call cost per reflection layer.

## Model Context Protocol (MCP)
**When to use**: an agent needs to pull from 3+ heterogeneous external systems (GitHub, Jira, Grafana, wikis, databases).
**How**: standardize resource/tool discovery and invocation via a client-server protocol (`resources/list`, `resources/read`, `tools/list`, `tools/call`) instead of bespoke integrations per source.
**Trade-offs**: large integration-maintenance savings at scale; adds an abstraction layer that must itself be secured (URI validation, RBAC, rate limiting).

## Conditional LLM Routing
**When to use**: high query volume where many queries don't actually need an LLM.
**How**: pre-filter via keyword/regex/metadata classifier; bypass the LLM when a cache hit, structured-DB lookup, or prewritten function answers the query.
**Trade-offs**: cuts cost/latency significantly; requires maintaining the routing/classification layer and cache invalidation logic.

## Multi-Index Retrieval / Hybrid Search
**When to use**: any RAG system with diverse document types or the need for both conceptual and exact-term matching.
**How**: separate indexes by topic/type/function; tag metadata at indexing time; combine semantic (embedding) and keyword (Lucene) search; chunk along semantic boundaries.
**Trade-offs**: much better retrieval precision/recall at scale; more indexing infrastructure and routing logic to maintain.

## Medallion Data Architecture
**When to use**: pipelines handling regulated/sensitive raw data that must remain reprocessable.
**How**: Raw (immutable source of truth) → Masked (PII removed/encrypted) → Segmented (chunked) → Analytical/Silver (cleaned, enriched) → Gold (query-optimized).
**Trade-offs**: strong auditability and reprocessing safety; more storage layers and retention-policy management.

## Risk Tiering (Inherent / Transient / Performance)
**When to use**: prioritizing governance investment across many GenAI use cases.
**How**: score each system on inherent risk (architecture/data, highest weight), transient risk (situational/context, medium weight), and performance risk (accuracy/reliability, lowest weight); combine into a final risk score.
**Trade-offs**: enables resource-efficient prioritization; requires periodic rescoring as models/data/context change.

## NIST AI RMF (Govern-Map-Measure-Manage)
**When to use**: building an enterprise AI governance program that needs to be both ethical and operationally actionable.
**How**: Govern (policies, culture, third-party oversight) → Map (define purpose/scope/allowed-disallowed uses) → Measure (metrics, red-teaming, benchmarks) → Manage (mitigate, prioritize, evaluate effectiveness) — iterate continuously.
**Trade-offs**: provides concrete operational structure that pure ethics-principle frameworks lack; needs organizational commitment to run the full loop repeatedly, not just once at launch.

## Active Learning Loop
**When to use**: any production GenAI system expected to improve post-launch from real usage.
**How**: capture user feedback (ratings/thumbs) → label failed/uncertain responses → remediate (fine-tune for pattern-based errors, re-index/re-chunk for retrieval-based errors) → align cadence to sprint cycles.
**Trade-offs**: continuous quality improvement without full retraining cycles; requires feedback-capture UI and disciplined labeling/triage process.
