# Chapter 3: Foundational Models in Amazon Bedrock

## Core Idea
Every foundation model in Bedrock is a Transformer built on self-attention, trained on massive curated corpora, then aligned via post-training (RLHF/DPO/ORPO/KTO) — understanding this pipeline (architecture → data → training → alignment → decoding strategy) is what lets you choose and configure a model correctly instead of treating it as a black box.

## Frameworks Introduced
- **Decision Table: Choosing a Sampling Strategy** (Table 3.7): the author's exact framework for setting inference-time decoding parameters.
  - When to use: whenever configuring `temperature`/`top_k`/`top_p` for a Bedrock `invoke_model` call.
  - How: Greedy decoding → deterministic, fast, use for structured extraction/templates/debugging. Temperature sampling → scales the probability distribution, higher = creative/lower = consistent, use for chatbots/ideation. Top-k sampling → restrict to k most probable tokens, use for controlled diversity with a fixed randomness ceiling. Top-p (nucleus) sampling → sample from the smallest token set whose cumulative probability ≥ p, adapts to model confidence, **default choice for high-quality open-ended natural language generation**.
- **LLM Scaling Tradeoffs table**: how architectural choices trade off cost, latency, and capability.
  - When to use: justifying a model-size or architecture choice against budget/latency constraints.
  - How: for each factor (parameter count, context window, attention head count, batch size), the book documents Impact on Capability, Impact on Cost (Training/Inference), Impact on Latency (TTFT/TPOT), and Impact on Throughput — increasing any factor generally raises capability and cost together; the practical skill is finding the minimum factor level that meets the Capability Threshold for your use case.
- **Adaptation Methods of Foundation Models spectrum**: from cheapest/fastest to most expensive/most tailored.
  - When to use: deciding how to customize a model for a domain task.
  - How, in increasing cost/effort order: Zero-Shot Learning (no examples, prompt only) → Few-Shot Learning (a handful of examples in the prompt) → Parameter-Efficient Fine-Tuning/PEFT (LoRA/QLoRA — modify a small parameter subset, low data/compute) → Full Fine-Tuning (adjust all model parameters, highest cost, highest task specificity). Default to the cheapest method that meets the accuracy bar; escalate only when prompting-based methods plateau.

## Key Concepts
- **Self-Attention (Intra-Attention)**: the mechanism letting each token in a sequence weigh every other token when building its representation — the core innovation from "Attention Is All You Need" (Vaswani et al.) that Bedrock's Transformer-based models are built on.
- **Multi-Head Attention (MHA) / Grouped-Query Attention (GQA) / Multi-Query Attention (MQA)**: variants trading off quality vs inference cost — MHA is the original full-quality form, GQA/MQA reduce the Key/Value head count to cut inference latency/memory at a small quality cost, used by newer Bedrock models for throughput.
- **Context Window (Context Length)**: the maximum number of tokens (input + output) a model can attend to in one call; directly bounds how much RAG-retrieved context or conversation history can be included.
- **Time To First Token (TTFT) / Time Per Output Token (TPOT)**: the book's two latency metrics — TTFT matters for perceived responsiveness (first output), TPOT for total generation time on long outputs; different model/architecture choices optimize differently for each.
- **RLHF (Reinforcement Learning from Human Feedback)**: a 3-stage alignment process — (1) collect human-ranked response pairs, (2) train a reward model (RM) on those rankings, (3) use PPO (Proximal Policy Optimization) to fine-tune the LLM against the reward model — used by ChatGPT, Claude, Llama 2-Chat, but expensive and hard to scale due to reliance on human ranking data.
- **DPO / ORPO / KTO**: RLHF alternatives that skip the separate reward-model stage. DPO directly optimizes the LLM policy using preference pairs (the policy itself implicitly encodes a reward signal). ORPO combines instruction-tuning and preference alignment in one step. KTO uses simpler binary "good/bad" signals rather than precise pairwise rankings, making it more robust to noisy labels.
- **PEFT (Parameter-Efficient Fine-Tuning) / LoRA / QLoRA**: fine-tuning techniques that update only a small injected parameter subset (low-rank adapter matrices) instead of the full model — QLoRA additionally quantizes the base model to reduce memory, making fine-tuning large models feasible on modest hardware.

## Mental Models
- Treat **model choice as a bucket, not a single winner**: the book frames this as "the Bedrock model bucket" — pick a small/cheap model for routing and simple tasks, a mid-tier model for typical generation, and a top-tier model only for the reasoning-heavy subset of requests.
- Use **Top-p (nucleus) sampling as your default** and only reach for greedy decoding or explicit top-k when you specifically need determinism or a hard randomness ceiling.
- Think of the **adaptation-method spectrum as an escalation ladder**: zero-shot → few-shot → PEFT → full fine-tuning. Skipping straight to full fine-tuning without exhausting cheaper rungs is the single most common cost mistake the chapter warns against.

## Anti-patterns
- **Jumping to full fine-tuning before trying prompting/PEFT**: full fine-tuning requires the most data, compute, and ongoing maintenance (re-tuning on model updates); the book positions it as a last resort once prompt engineering, few-shot examples, and PEFT/LoRA have been exhausted.
- **Using greedy decoding for creative/open-ended generation, or high-temperature sampling for structured extraction**: mismatching sampling strategy to task type produces either repetitive/robotic output or unreliable structured output — Table 3.7 exists specifically to prevent this.
- **Ignoring context-window limits when designing RAG pipelines**: retrieving more context than the model's window supports silently truncates input; the book flags this as a design-time constraint, not a runtime surprise to debug later.

## Reference Tables
**Table 3.7 — Decision Table: Choosing a Sampling Strategy**
| Strategy | What it controls | Pros | When to use it |
|---|---|---|---|
| Greedy decoding | Always picks the single most probable token | Deterministic, fast, predictable | Debugging, deterministic pipelines, structured extraction |
| Temperature sampling | Scales the probability distribution before sampling | Simple, flexible creativity control | Global creativity/coherence tuning (chatbots, ideation) |
| Top-k sampling | Limits choices to top-k most probable tokens | Prevents unlikely tokens, more coherent than pure temperature | Controlled diversity with a fixed randomness ceiling |
| Top-p (nucleus) sampling | Samples from smallest token set with cumulative probability ≥ p | Adapts to model confidence; best fluency/diversity balance | Default for high-quality open-ended generation |

**Alignment technique comparison** (RLHF / DPO / ORPO / KTO): RLHF = highest quality alignment, highest cost/complexity (needs separate reward model + PPO); DPO = simpler, directly optimizes on preference pairs, no reward model needed; ORPO = combines SFT + preference alignment in a single training step; KTO = works with simple binary good/bad signals, most robust to noisy/inconsistent labeling.

## Worked Example
The book's worked example for tokenization/attention effects uses the sentence pair "The cat sat on the mat" vs "The mat sat on the cat" — identical bag-of-words but opposite meaning — to demonstrate why self-attention (which encodes token *position and relationship*, not just presence) is necessary for language understanding, motivating why positional encoding is a required Transformer component covered alongside attention.

## Key Takeaways
1. Default to Top-p (nucleus) sampling for general text generation; use greedy decoding only for deterministic/structured tasks.
2. Escalate model customization through the adaptation-method ladder (zero-shot → few-shot → PEFT/LoRA → full fine-tuning) — don't skip to full fine-tuning.
3. GQA/MQA-based models trade a small quality loss for meaningfully lower inference latency/cost — prefer them for high-throughput production workloads.
4. RLHF produces the highest-quality alignment but is expensive; DPO/ORPO/KTO are viable lower-cost alternatives, with KTO the most noise-tolerant.
5. Context window size is a hard design constraint for RAG — size your retrieval/chunking strategy (Ch5) against the target model's actual context length.

## Connects To
- **Ch1**: the Foundation Model Selection Matrix from Ch1 is grounded here with the underlying architectural reasons (attention variant, context window, parameter count) behind each recommendation.
- **Ch4**: applies these sampling-strategy and adaptation-method choices when building the first working solution.
- **Ch5**: context-window constraints here directly bound RAG chunking/retrieval design.
