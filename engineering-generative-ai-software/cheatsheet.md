# Cheatsheet — Engineering Generative AI-Based Software

## Decision Rules

- **When choosing a prompting strategy**: if the task needs integrating multiple weak signals → use Chain-of-Thought. If one clear example suffices → one-shot. If retraining is affordable and the task is stable → fine-tune instead of stacking few-shot examples every call. (Ch2)
- **When picking an evaluation metric**: generated text → BLEU (clipped); generated code → CodeBLEU (AST-based, never raw BLEU); summarization → ROUGE. Wrong metric choice actively rewards degenerate output (e.g., BLEU rewards verbatim code copying). (Ch3)
- **When a model answer scores well on BLEU but you suspect it's factually wrong** (e.g., near-identical 100-word responses swapping one fact) → switch to metamorphic testing: define a non-equivalence relation (perturb the fact, expect the answer to change) or equivalence relation (perturb irrelevant text, expect the answer to stay the same). (Ch6)
- **When choosing an architecture**: ask "where does the model live?" Same process → monolith (prototypes only). Same machine, separate process → MVC (best maintainability-to-complexity ratio). Remote machine → microservice (best scale/interoperability, worst latency/security surface). Split device+cloud → embedded/edge (use for latency- or privacy-sensitive sub-tasks only). (Ch5)
- **When you need to reduce hallucination** → ground the model with RAG (embed → vector search → summarize), but always apply a relevance/where-clause filter, because nearest-neighbor search always returns *something* even when nothing relevant exists. (Ch5, Ch7)
- **When picking an implementation language**: need IP protection or direct hardware access → C/C++/C#. Need fastest prototyping with the largest ecosystem → Python. Need cross-vendor GPU/NPU inference without CUDA lock-in → DirectML. Need cross-language/cross-platform portability → export to ONNX. (Ch6)
- **When designing a multi-agent workflow**: always set a hard iteration cap and an explicit success condition — agents converge in ~10-20 iterations or never, and never self-terminate sensibly on their own. Mix model families across roles (reasoning model as critic/designer, plain LLM as generator) rather than reusing one model twice. (Ch7)
- **When a code-generation agent produces broken output** → add a compiler/validator-in-the-loop with a bounded retry count before reaching for a bigger/more expensive model; tool access often substitutes for model size. (Ch7)
- **When choosing a cloud deployment level**: selling a finished experience → app-level. Selling model access to developers → capability-level. Selling data for others to train on → service-level. Selling raw scalable compute → infrastructure-level. (Ch8)
- **When exposing any GenAI capability as an API**: version the path (`/v1/`) from day one, add heartbeat/capabilities/restart endpoints, use 200/400/500 status families deliberately, and gate every endpoint with token auth + TLS before anything else. (Ch9)
- **When deciding how autonomous an agentic feature should be**: workflow steps are enumerable → classical software in the driving seat (AI called for bounded sub-tasks, validated by code). Workflow is genuinely open-ended → AI in the driving seat, but treat "when does it stop" as an unsolved problem requiring your own engineered stop-condition. (Ch10)

## Trade-off Matrix: Architectural Styles (Ch5, Table 5.1)

| Architecture | Scalability | Maintainability | Performance | Latency | Security |
|---|---|---|---|---|---|
| Monolithic | Low | Low | High (if small) | Low | High (few attack vectors) |
| MVC | Medium | Medium | Medium | Medium | Medium |
| Microservices | High | High | Variable | High | Medium (complex setup) |
| RAG | High | High | Variable | Medium | Medium |
| Embedded AI | Medium | Medium | High | Low | High |

## Trade-off Matrix: Cloud Deployment Levels (Ch8)

| Level | You provide | Customer controls | Best for |
|---|---|---|---|
| App | Finished web app/container | Nothing (uses your UI) | End-user products (chatbots, image generators) |
| Capability | REST-exposed trained model | Their own app on top | Model-access businesses (OpenAI-API-style) |
| Service | Exposed datasets | Their own model training | Data-collection businesses (BigQuery-style) |
| Infrastructure | Pre-configured compute/OS | Their entire stack | Power users needing scalable compute, not software |

## Thresholds & Defaults

- **Latency target for interactive response**: below 1 second is the general acceptance bar; below 0.7s feels fully responsive. (Ch1, Ch4)
- **Datacenter-connection-lost detection**: no response within 30 seconds → treat connection as lost and trigger fallback. (Ch4)
- **Throughput example requirement**: "at least 1,000 prompts per minute" is cited as a representative performance non-functional requirement. (Ch4)
- **Quantization cost/benefit**: 32-bit → 8-bit reduces memory ~4x, 32-bit → 4-bit reduces memory ~8x, for roughly a 10% performance (quality) drop. (Ch6, Ch8)
- **McCabe complexity ceiling** (safety-critical modules, general SE practice cited): keep below 20. (Ch4)
- **Multi-agent convergence window**: agents that will converge to a solution typically do so within 10-20 iterations; beyond that, expect drift, mutual praise loops, or degeneration — cap iterations accordingly. (Ch7)
- **BLEU/ROUGE model-size finding**: average scores comparing same-family models of different sizes were surprisingly low (~0.35 BLEU, ~0.2 ROUGE for SmolLM family; best case 0.74 BLEU for Qwen3 family) — don't assume larger same-family models converge toward similar answers. (Ch8)
- **Fine-tuning accuracy example**: a k-NN classifier fine-tuned on embeddings from a matched-domain pre-trained model achieved 99.7% accuracy — domain match between pre-training and fine-tuning data is the biggest lever for this kind of result. (Ch1)

## Tells & Smells

- **"The BLEU score is nearly perfect but the answer feels off"** → you're probably comparing near-identical-length responses that differ in one key fact (e.g., wrong capital city); switch to metamorphic or exact-fact testing. (Ch6)
- **"Our multi-agent demo works great in the first 10 turns but degrades after that"** → expected behavior, not a bug — agents either converge quickly or drift into off-topic/praise loops; add a stop condition, don't chase "why did it wander." (Ch7)
- **"Vector search always finds something, even for irrelevant queries"** → expected behavior of nearest-neighbor embedding search; add a distance/where-clause relevance filter rather than trusting top-1 results blindly. (Ch7)
- **"Our exposed model server is getting hammered with traffic we didn't generate"** → you likely have no authentication on the endpoint; the book reports 100+ attacks/hour on an unauthenticated exposed Ollama server within a short window. (Ch5, Ch9)
- **"Model behaves inconsistently after a mid-conversation correction ('forget the previous answer')"** → expected; conversation history persists and is still sent to the model even after a "forget" instruction — start a fresh context if you need a clean slate. (Ch2)
- **"Fine-tuned model works great on our C++ demo dataset but fails badly on other domains"** → check for a pre-training/fine-tuning domain mismatch (high OOV token rate is the diagnostic). (Ch1)
- **"Removing an existing public API endpoint keeps breaking clients"** → expected; per "public APIs are forever," add before you need to remove, and version (`/v2/`) instead of mutating `/v1/`. (Ch9)
