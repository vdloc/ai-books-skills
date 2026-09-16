# Cheatsheet

## Decision Rules

**"Should this be an LLM?"** — If the task is structured extraction/lookup with a fixed schema, use a rule-based parser or classical ML (cheaper, no hallucination risk). Use an LLM only when the task genuinely requires language understanding, generation, reasoning, or multi-step synthesis. (Ch 8, Ch 4)

**"Should this agent use an LLM?"** — Use LLM utility "High" only where reasoning/creativity is central to the task (sentiment analysis, compliance review). Use "Medium/Low" LLM utility (traditional ML/rules) for well-defined statistical tasks (price prediction, defect detection via CV). Force-fitting LLMs onto simple tasks = overengineering. (Ch 4)

**"Fine-tune or prompt-tune?"** — Default to prompt tuning: fast, cheap, no training data needed. Fine-tune only when: (a) domain nuance genuinely can't be captured by prompting, (b) you have curated task-specific data, (c) compute budget allows. Prefer LoRA/QLoRA over full fine-tuning when tuning is justified. (Ch 5, Ch 6)

**"A/B test or Shadow test this model update?"** — Shadow test for compliance-sensitive or high-risk changes (no user exposure until validated). A/B test for UX-facing changes where live comparison is acceptable risk. (Ch 3)

**"Centralized or decentralized orchestration?"** — Centralized: when predictability, ease of monitoring, and control matter most (regulated/structured workflows). Decentralized: when resilience and flexibility matter more than central control, and you can tolerate more design complexity. (Ch 4)

**"Cloud, on-prem, hybrid, or edge deployment?"** — Cloud: elastic scale, GPU/TPU access, non-regulated data. On-prem: regulated/sensitive data, full control needed, capex available. Hybrid: mixed-sensitivity workloads. Edge: real-time/offline/privacy-critical, accept compute constraints (quantize/prune models). (Ch 6)

**"Which deployment topology?"** — PoC only → single-node. High-traffic + fault tolerance → multinode/clustered. Independent component scaling → microservices. Bursty/event-driven, no infra mgmt → serverless. Global low-latency → distributed. Real-time edge + heavy cloud training → cloud-edge hybrid. (Ch 6)

**"When does a RAI principle conflict need resolving?"** — Transparency vs. Privacy → mask + differential privacy. Explainability vs. Performance → SHAP/LIME post-hoc or surrogate model. Inclusiveness vs. Fairness → bias audit + data augmentation. Never just pick one principle and ignore the conflicting one. (Ch 7)

## Decision Tree: Prompt Engineering Pattern Choice
- Multiple interchangeable prompt strategies, switch by context? → **Strategy Pattern**
- Same structure, minor tweaks, repeated? → **Template Pattern**
- Prompt composed of multiple sections built incrementally? → **Builder Pattern**

## Decision Tree: LLMOps Issue Triage
- Hallucination / wrong facts → check grounding first → integrate RAG with verified sources
- Outdated responses → check training cutoff → add external tool/API lookup
- High cost/compute → check for redundant large-model calls → downsize model + cache + autoscale
- Poor enterprise jargon understanding → check training corpus scope → fine-tune or embed proprietary docs
- Data leakage in logs/output → check masking → anonymize PII before persisting
- Missing audit trail → check logging config → log prompts/responses/user IDs to compliance-grade storage

## Trade-off Matrix: Latency vs. Accuracy vs. Resilience Optimization
| Priority | Techniques |
|---|---|
| Latency | Smaller/quantized models, minimize token input, parallelize retrievals, async multi-index calls |
| Accuracy | Fine-tune on domain data, multipass reasoning + self-verification, high-quality RAG with metadata filtering |
| Resilience | Failure-triggered retrain/reprompt pipelines, automated regression per release, confidence-threshold fallback routing |

## Thresholds & Defaults

- **Context window as model-selection gate**: pick a model whose context window fits your largest single-pass artifact (book example: GPT-4-32k forced multi-pass segmentation; GPT-4o's 128K enabled single-pass — always check this before other criteria).
- **Embedding dimensionality for code/syntax retrieval**: compress from default (3072) to ~256 dims — improves speed/memory with acceptable precision loss for syntax-heavy content; keep higher dims (768-3072) for nuanced natural-language semantic tasks.
- **Chunk size ceiling**: ≤2MB per object/chunk for code/document segmentation (book's IDFE convention) — exceed only when a single object can't be split without breaking semantic coherence (then store whole).
- **Accuracy expectation by complexity tier**: expect 90-95% accuracy on low/medium-complexity extraction tasks, 80-85% on high-complexity (deeply nested, dynamic-SQL-style) tasks — plan human-in-the-loop review effort accordingly, don't expect uniform accuracy across complexity tiers.
- **Latency benchmark (text generation)**: target <500ms; >1.5s should auto-trigger a cached-output fallback.
- **Availability target**: 99.9% monthly uptime (≈43 min max downtime/month); <99.5% should flag incident review.
- **Error rate ceiling**: <1% invalid/hallucinated output per 10k interactions; >3% should trigger a retraining/fine-tuning evaluation.
- **Adversarial robustness bar**: >90% attack detection rate in red-team simulations; <80% should trigger defense-layer tuning.
- **Prompt injection tolerance**: <1% injection success in simulated tests; any detected live attempt should auto-disable the session.
- **User satisfaction bar**: NPS >60, SUS >80; a >10-point NPS drop within two weeks should alert the product team.
- **Risk-score weighting**: weight inherent risk highest, transient risk second, performance risk lowest when computing an overall system risk score — inherent risk is hardest/costliest to fix after deployment.

## Tells & Smells

- **"No baseline to compare post-fine-tune quality"** → you skipped telemetry/eval-pipeline setup before fine-tuning; you can't tell if the tune helped.
- **Generic, soulless GenAI output** ("Introducing the new Smart Water Bottle!...") → your prompt lacks role, brand voice, target audience, concrete features, and format constraints — spend the extra 60-120 seconds to fix the prompt.
- **Model consistently misses the same class of case** (e.g., same SQL pattern, same phrasing) → pattern-based error → fix via fine-tuning, not re-chunking.
- **Model surfaces wrong/irrelevant documents but reasoning is otherwise sound** → retrieval-based error → fix via re-indexing/re-chunking/better metadata, not fine-tuning.
- **Chatbot gives good answers on paper but stakeholders don't trust it in production** → likely a Transparency/Explainability gap, not an accuracy gap — add traceability (chunk IDs, source logging) before touching the model.
- **"It works great on our test set but breaks on edge cases in production"** → check if evaluation only used low/medium-complexity examples; re-evaluate against a stratified complexity-tier sample.
- **Security review finds hardcoded credentials/URIs in agent tool code** → common demo-code shortcut that ships to production; always audit for hardcoded secrets before launch.
- **A GenAI feature has no documented "disallowed use cases"** → missing NIST RMF "Map" step — define allowed/disallowed uses explicitly before wider rollout.
- **Two RAI principles seem to be in tension and no one has a resolution plan** → don't pick a side by default; consult the conflict-resolution table (Ch 7) for the matched mitigation technique.
