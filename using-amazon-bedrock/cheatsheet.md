# Cheatsheet — Using Amazon Bedrock

Decision rules, thresholds, and quick-reference tables. Not a glossary — every line helps you decide something.

## Customization Decision: Prompt Engineer vs. RAG vs. Fine-Tune vs. Train From Scratch

| Signal | Choice |
|---|---|
| Pretrained knowledge already covers the domain | Prompt engineering |
| Need current/external/proprietary facts injected at inference time | RAG |
| Need rigid, exactly-consistent output format (fixed JSON schema, classification tags) every time | Fine-tune |
| Need domain-specific jargon/pattern understanding baked into behavior | Fine-tune |
| Need to cut inference cost by shortening prompts (no more few-shot examples needed) | Fine-tune |
| Task is general, need fast iteration, can't retrain quickly | Prompt engineering |
| Insufficient domain data to fine-tune meaningfully | Prompt engineering or RAG |
| No existing model fits at all | Train from scratch (last resort — highest cost/carbon) |

**Sustainability-ordered default**: try prompt engineering first, then RAG, then parameter-efficient tuning (LoRA/PEFT), then full fine-tuning, then training-from-scratch — each step down is a deliberate cost/carbon trade, not a default.

## Pricing Model Decision

| Model type | Pricing |
|---|---|
| Out-of-the-box (base) model | On-demand (pay per input/output token) |
| Fine-tuned / customized model | Provisioned throughput ONLY (no on-demand option) — bills continuously by model-unit-hour, must be explicitly deleted |
| Bulk, non-urgent generation | Batch inference — 50% discount vs. on-demand |

**Tell**: if you fine-tuned it, you must provision throughput to serve it — budget for continuous billing until you tear it down.

## Model Selection Heuristics

| Need | Model direction |
|---|---|
| Long-context reasoning, safety-focused dialogue | Claude family (Anthropic) |
| Ultra-low-latency text | Amazon Nova Micro |
| Cost-effective multimodal (text/image/video) | Amazon Nova Lite/Pro |
| Studio-quality image generation | Nova Canvas / Stable Diffusion 3.5 Large |
| Open-weight, cheapest reasoning/code | DeepSeek R1 |
| Enterprise privacy-focused chat/search | Cohere Command/Embed |
| Open-source, fast inference, low memory | Mistral 7B/8x7B |
| Visual Q&A, detailed image analysis | Claude 4 Sonnet or LLaVA (via SageMaker) |
| Embeddings for similarity search / RAG | Titan Embeddings G1-Text (25+ languages) or V2 (100+ languages, long docs) |

## Inference Parameter Tuning — Symptom → Fix

| Symptom | Fix |
|---|---|
| Responses too generic/similar | Increase temperature; widen top_p/top_k |
| Responses too erratic/irrelevant | Decrease temperature; tighten top_p/top_k |
| Responses cut off abruptly | Increase max_tokens |
| Responses too verbose | Decrease max_tokens |
| Responses don't conclude naturally | Add/refine stop_sequences |

## Guardrail Threshold Selection

| Domain stakes | Content filter threshold |
|---|---|
| Finance, healthcare, legal (high liability) | HIGH — strict blocking, accept more false positives |
| General consumer chat | MEDIUM — balance safety and UX |
| Internal tools, trusted users | LOW — minimize false positives, allow more latitude |

**Trade-off rule**: stricter thresholds block more unwanted content but also block more borderline-legitimate content — no universal right answer, only the one matching your risk tolerance.

## Vector Database Selection

| Priority | Choice |
|---|---|
| Native AWS integration, hybrid keyword+vector search | Amazon OpenSearch |
| Fully managed, lowest latency, willing to pay more | Pinecone |
| Raw query throughput, self-hosting acceptable | Milvus |
| Prototyping / small-scale, in-memory | FAISS |
| Strong developer community, hybrid search | Weaviate |

## Batching Strategy

| Situation | Choice |
|---|---|
| High request volume, need max GPU utilization | Continuous batching |
| Have a cheap draft model that's often right | Speculative batching |
| Draft model frequently wrong | Avoid speculative batching (recompute overhead) |

## Security Tells & Smells

| Observation | Likely issue |
|---|---|
| Agent has `AmazonBedrockFullAccess` | Overprivileged — scope to specific actions immediately |
| Blocked-message text explains exactly why | Information leak — use generic messaging instead |
| SSL verification disabled anywhere outside a demo | Critical security anti-pattern — never ship to production |
| No metadata/knowledge base backing a text-to-SQL or RAG system | Expect inaccurate/hallucinated structured output |
| Model repeatedly queried with similar prompts about one entity | Possible model inversion attack — rate-limit and monitor |
| Provisioned throughput still running after testing | Active cost leak — thousands/month if forgotten |

## Compliance Quick-Map

| Handling... | Must address |
|---|---|
| EU personal data | GDPR — data minimization, right to erasure |
| US healthcare data (ePHI) | HIPAA — KMS encryption, strict IAM, full audit |
| Payment card data | PCI DSS — tokenization, network segmentation |
| US federal/public sector | FedRAMP — authorized regions, accredited data flows |
| California consumer data | CCPA — transparency, opt-out, retention limits |

## Memory Sizing Rule of Thumb

- ~2 bytes per parameter at 16-bit precision.
- 1B params ≈ 2GB; 175B params ≈ 350GB (parameters only — excludes activations/gradients during training/inference).
- Use this before requesting SageMaker instance quota increases.

## Cost Optimization Checklist (apply in order)

1. Trim prompt verbosity; drop unnecessary few-shot examples.
2. Lower `max_tokens` to the minimum sufficient for the task.
3. Cache static prompt prefixes (system instructions, few-shot blocks).
4. Route simple requests to a smaller model (LLM cascading).
5. Move non-urgent bulk work to batch inference (50% discount).
6. Use on-demand event-driven invocation (Lambda/EventBridge) instead of always-on compute.
7. Set up AWS Budgets + Cost Anomaly Detection alarms before scaling to production volume.
