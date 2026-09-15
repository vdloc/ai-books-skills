## Decision Rules

- **Choosing a sampling strategy** (Table 3.7): need determinism/structured output → greedy decoding. Need global creativity control → temperature sampling. Need a randomness ceiling → top-k. Default for open-ended NL generation → top-p (nucleus).
- **Solving a capability gap**: missing/outdated knowledge → RAG. Wrong style/format the model can't be prompted into → fine-tuning (PEFT before full). Neither → better prompt design first (cheapest, try this before either).
- **Choosing invocation pattern**: interactive, low-latency → `InvokeModel`/`Converse`. UI needs tokens as they arrive → `InvokeModelWithResponseStream`/`ConverseStream`. Large non-interactive volume → Batch Inference (`CreateModelInvocationJob`).
- **Choosing compute for scaling**: bursty/event-driven/short → AWS Lambda. Sustained/heavy/long-running → Amazon ECS with Fargate.
- **Handling a failing Bedrock call**: transient (throttling) → exponential backoff + jitter via Boto3 retry config. Persistent (service down) → Circuit Breaker (fail fast, don't retry). Preferred model unavailable → Model Fallback chain. Regional outage risk → Cross-Region Inference.
- **IAM scoping**: learning/prototype phase → broad managed policy (`AmazonBedrockFullAccess`) is acceptable. Production → narrow to specific action prefixes (`bedrock:InvokeModel`, etc.) once real permission surface is known.
- **Single vs. multi-agent**: task fits one coherent instruction/tool set → single agent. Task decomposes into specialized roles (e.g. plot + dialogue + continuity) → multi-agent collaboration.
- **Tool integration for agents**: one-off internal tool → Action Group + Lambda. Multiple external systems, want standardized integration → MCP server.

## Model Selection Matrix (by workload)

| Need | Model class | Why |
|---|---|---|
| Complex reasoning, long-form | Claude (Sonnet/Opus) | Highest capability, higher cost |
| Cost-effective at scale | Amazon Nova (Micro/Lite) | Cheap, fast, good for routing/simple tasks |
| Vision-language at scale | Amazon Nova (Pro/Premier) | Multimodal, tuned for throughput |
| Embeddings / semantic search | Amazon Titan Embeddings | Purpose-built for retrieval, not generation |
| Open-weight experimentation | Llama (2/3 variants) | Open weights, customizable |

## Adaptation Method Ladder (cost-increasing)

| Method | Data needed | Compute cost | When |
|---|---|---|---|
| Zero-shot | None | None | Model already generalizes well to the task |
| Few-shot | A handful of examples | None | Need to pin output format/style |
| PEFT (LoRA/QLoRA) | Moderate | Low-moderate | Prompting plateaus, need consistent behavior shift |
| Full fine-tuning | Large | High + ongoing maintenance | Need deep domain/style adaptation nothing else achieves |

## Alignment Technique Comparison

| Technique | Reward model needed? | Label type | Robustness to noise |
|---|---|---|---|
| RLHF | Yes (separate RM + PPO) | Pairwise rankings | Lower — expensive to scale |
| DPO | No | Preference pairs | Moderate |
| ORPO | No (single-step SFT+alignment) | Preference pairs | Moderate |
| KTO | No | Binary good/bad | Highest — most robust to noisy labels |

## Circuit Breaker States

| State | Behavior | Next |
|---|---|---|
| Closed | Requests pass, failures counted | → Open if failures exceed threshold |
| Open | All calls rejected immediately | → Half-Open after timeout |
| Half-Open | Limited trial requests | → Closed (success) / Open (failure) |

## Thresholds & Defaults

- Default sampling strategy for general text generation: **top-p (nucleus)**.
- Embedding model chunk size ceiling cited in the book: some models cap around **512 tokens** — size chunking strategy accordingly.
- Custom (fine-tuned) Bedrock models require **Provisioned Throughput** — no on-demand serving option; factor this fixed cost into fine-tune-vs-RAG decisions.
- Bedrock has **no free tier** — cost monitoring (AWS Budgets, Cost Explorer) should be set up at initial environment setup, not deferred.
- Prompt versioning: **MAJOR** = behavior/output-contract change, **MINOR** = backward-compatible enhancement, **PATCH** = wording/formatting only.

## Tells & Smells

- Seeing `ThrottlingException` handled with manual `time.sleep()` retry loops → thundering-herd risk; switch to Boto3's built-in exponential-backoff-with-jitter retry config.
- Prompts hardcoded as literal strings inside Lambda functions → couples prompt iteration to full redeploys; decouple into external storage.
- A single agent's instructions/tools growing unwieldy trying to cover multiple distinct sub-tasks → signal to decompose into a multi-agent architecture (Ch7's Narrative Agent pattern).
- Logging only final model output, no `FinalPrompt`/`InputVariables`/`GuardrailEvaluation` → audit trail is incomplete; can't reconstruct what actually happened for compliance or debugging.
- Defaulting to the largest/most capable model for every call in a high-volume application → cost inefficiency; apply the model-bucket mental model (cheap model for routing/simple tasks, premium model only where reasoning quality drives value).
- Writing bespoke integration code for each new external tool an agent needs → signal to move that tool behind an MCP server instead.
