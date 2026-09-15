# Chapter 1: Getting Started with Amazon Bedrock

## Core Idea
Amazon Bedrock is a fully managed, serverless service that gives unified API access to multiple foundation models (Anthropic Claude, Amazon Nova/Titan, Meta Llama, Mistral, AI21, Cohere, Stability AI) without managing infrastructure — the entry point for turning Generative AI's probabilistic "predict-the-next-token" capability into production applications.

## Frameworks Introduced
- **Generative AI vs Traditional AI comparison table**: the author's baseline mental model for deciding when GenAI applies.
  - When to use: framing any "should we use an LLM here" conversation with stakeholders.
  - How: contrast on 7 axes — Primary Objective (classify/predict vs generate/reason), Input Data (structured vs unstructured), Output (labels vs novel content), Training Paradigm (supervised, task-specific vs self-supervised pretrain + fine-tune, general-purpose), Flexibility (rigid vs zero/few-shot adaptable), Interpretability (higher vs "black box"), Hardware (CPU/moderate GPU vs massive GPU/TPU clusters).
- **Foundation Model Selection Matrix**: match model family to workload rather than defaulting to one provider.
  - When to use: at solution design time, before writing any integration code.
  - How (Model Family → Recommended Use, from the book's table): Claude (Sonnet/Haiku) → complex reasoning and long-form responses; Amazon Nova (Micro/Lite) → cost-effective AI at scale; Amazon Nova (Pro/Premier) → vision-language tasks at scale; Amazon Titan Embeddings → embeddings and semantic search / RAG workloads; Llama (2/3 variants) → open-weight model experimentation; NVIDIA Nemotron (Nano/VL) → specialized multimodal/edge scenarios. Selection rationale should weigh: task complexity, latency/cost budget, need for open weights, and multimodal requirements.

## Key Concepts
- **Foundation Model**: a large-scale model (billions of parameters) pretrained on massive datasets that serves as a general-purpose engine adaptable to many downstream tasks via prompting or fine-tuning, rather than one task-specific model per use case.
- **RAG (Retrieval-Augmented Generation)**: grounding a foundation model's output in retrieved enterprise data at inference time instead of relying solely on parametric knowledge.
- **Provisioned Throughput**: a Bedrock capacity model that reserves dedicated model throughput for predictable, high-volume workloads (vs on-demand pay-per-token).
- **Bedrock Guardrails**: a configurable content-filtering/safety layer applied consistently across models to block harmful content, PII, and off-topic responses.
- **Amazon Bedrock Marketplace**: the mechanism for accessing additional third-party and specialized foundation models beyond Bedrock's core curated set.
- **Agentic AI**: the 2025+ industry shift from models that only respond ("talk") to models that autonomously execute multi-step workflows ("act") — introduced here as the trajectory the rest of the book (esp. Ch8) builds toward.

## Mental Models
- Think of Bedrock as a **unified API layer**, not a model: the value is provider-agnostic access + built-in enterprise controls (guardrails, IAM, audit logging), not any single model's capability.
- Use the **Traditional AI vs Generative AI table** whenever a stakeholder asks "why not just use a normal ML model" — most traditional AI problems (classification, churn scoring) are still better served by discriminative models; GenAI earns its cost when the task requires generating novel, unstructured output.
- Treat **model selection as a per-workload decision**, not a one-time platform choice — a single Bedrock application commonly mixes a small/cheap model (Nova Micro) for classification-like routing with a larger model (Claude Sonnet) for complex reasoning steps.

## Anti-patterns
- **Defaulting to the biggest/most capable model for every task**: wastes cost and latency budget on tasks a smaller model (Nova Micro/Lite) handles adequately — the book's model-selection table exists specifically to prevent this.
- **Treating Bedrock adoption as "just another AWS API call"**: skipping guardrails, audit logging, and access control configuration at the prototype stage creates rework later, since production GenAI has compliance/security requirements traditional AI projects often didn't (finance, healthcare, HR domains named explicitly).

## Reference Tables
| Feature | Traditional AI (Predictive/Discriminative) | Generative AI (Foundation Models) |
|---|---|---|
| Primary Objective | Classify, Predict, Cluster, Optimize | Generate, Reason, Synthesize, Transform |
| Input Data | Structured (tabular, numerical, categorical) | Unstructured (text, images, audio, video, code) |
| Output | Discrete labels, probabilities, forecasts | Novel content (text, code, media) |
| Training Paradigm | Supervised learning (specific task) | Self-supervised pretraining + fine-tuning (general purpose) |
| Flexibility | Rigid; retraining required for new tasks | Highly adaptable; zero/few-shot via prompting |
| Interpretability | Often higher (e.g. decision trees) | Lower ("black box" neural networks) |
| Hardware | CPU or moderate GPU | Massive GPU/TPU clusters |
| Example | Predicting churn probability (0.75) | Drafting a personalized email to prevent churn |

Generative AI timeline the book grounds Bedrock in: 1950s-80s rule-based/symbolic AI → 1980s Boltzmann machines → 2014 GANs (Goodfellow) → 2017 Transformers (Vaswani et al.) → 2018-2020 GPT-1/2/3 → 2020 diffusion models (Ho et al.) → 2022 ChatGPT/Stable Diffusion ("Netscape Moment") → 2023 Amazon Bedrock launch → 2025 Agentic AI era begins.

## Worked Example
The book's worked mental model for word prediction: given the incomplete sentence "Children like to", a generative model assigns probabilities to candidate next words (e.g. "play" = 40%, highest) and selects the highest-probability completion — illustrating that Bedrock's foundation models are fundamentally probabilistic next-token predictors, not rule engines, which is why prompting technique (Ch3+) and guardrails matter more than for traditional software.

## Key Takeaways
1. Bedrock's core value is a unified, serverless API across multiple foundation-model providers plus built-in enterprise guardrails/security — not raw model capability alone.
2. Pick the foundation model per workload using the selection matrix (task complexity, cost, latency, need for open weights, multimodality) rather than standardizing on one model for an entire application.
3. Reach for the Traditional-vs-Generative-AI table before proposing an LLM solution — many "AI problems" are still better and cheaper solved with discriminative ML.
4. RAG and embeddings (Titan Embeddings) are introduced here as the standard grounding mechanism — expect deep coverage in later chapters on Knowledge Bases.
5. The industry (and the book's own structure) is moving from single-turn generation toward Agentic AI — multi-step, tool-using workflows — culminating in Ch8's coverage of agents.

## Connects To
- **Ch3**: expands "Foundation Models in Amazon Bedrock" — deep dive on each model family's parameters, context windows, and selection criteria referenced only briefly here.
- **Ch4**: builds the first hands-on Generative AI solution using the models introduced here.
- **RAG / Knowledge Bases**: the RAG concept introduced here recurs as a named Bedrock feature in Ch4-Ch5.
- **Agentic AI**: the "models that act" trend flagged here is the explicit subject of Ch8.
