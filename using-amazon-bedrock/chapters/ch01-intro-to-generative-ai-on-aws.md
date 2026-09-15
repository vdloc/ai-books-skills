# Chapter 1: Introduction to Generative AI on AWS

## Core Idea
Amazon Bedrock is a fully managed service that unifies access to multiple providers' foundational models (FMs) behind one API — `InvokeModel()` — abstracting infrastructure management while leaving data quality, governance, and compliance as the customer's responsibility.

## Frameworks Introduced
- **Life Cycle of a Generative AI Solution (7 phases)**: Problem Definition & Scope → Data Collection & Preparation → Model Selection & Tuning → Training & Evaluation → Integration & Deployment → Monitoring, Security & Compliance → Feedback & Improvement.
  - When to use: scoping any new Bedrock project, from a single prototype to an org-wide rollout.
  - How: treat phases as iterative, not strictly linear — expect back-and-forth especially between Problem Definition and Data Collection while you're still discovering what "good" looks like for your use case.
- **Two core technologies underlying Bedrock FMs**: Diffusion Models (progressive denoising — image generation, inpainting, image-to-image) and Transformers (self-attention — text/NLP, parallelizable vs. older recurrent architectures).
  - When to use: pick diffusion-based models (Stable Diffusion) for image synthesis tasks, transformer-based models (Claude, Titan, Llama) for language tasks.

## Key Concepts
- **Foundational Model (FM)**: large-scale neural net pretrained on broad data, adaptable to many downstream tasks without retraining from scratch.
- **Transfer learning**: FM pretrained knowledge lets it be fine-tuned on small task-specific datasets, or used zero-shot with no extra training.
- **Zero-shot learning**: model performs a task it never saw examples for, relying only on pretrained knowledge + prompt instructions.
- **Token**: basic text unit (word, subword, or punctuation) that Bedrock bills on; ~1 token ≈ 0.75 English words.
- **Embedding**: high-dimensional vector representation of a token/text used for semantic similarity (powers RAG retrieval).
- **Provisioned throughput vs. on-demand pricing**: on-demand = pay-per-token, only for out-of-the-box models; provisioned throughput = fixed hourly reserved capacity, *required* for fine-tuned/customized models.
- **Bedrock Guardrails**: configurable content filters (denied topics, sensitivity thresholds) applied before a response reaches the user.

## Mental Models
- Think of Bedrock as a **translation layer**, not a model: it doesn't replace the need for your own data governance, labeling, or domain expertise — it removes the burden of hosting/serving the models themselves.
- Use **temperature / top-k / top-p** as your creativity dials: top-k caps the candidate-token pool to the k most likely; top-p (nucleus) caps by cumulative probability mass — both trade coherence for diversity.
- Treat **RAG vs. fine-tuning** as different fixes for different problems: RAG grounds answers in an external knowledge base at inference time (no retraining); fine-tuning bakes domain behavior into model weights (requires provisioned throughput).

## Anti-patterns
- **Treating Bedrock as a data-quality solution**: it does not automate data collection/cleaning — bad input data still produces bad output regardless of which FM you call.
- **Skipping guardrail/compliance planning until after deployment**: EU GDPR, CCPA, and similar region-specific laws (e.g., Saudi PDPL) constrain where data can be processed — bake VPC isolation and regional deployment into the architecture from the start, not retrofitted.
- **Assuming a single "best" model**: model choice depends on context window, modality, cost, and task fit (e.g., Claude for long-context reasoning vs. Nova Micro for low-latency text).

## Code Examples
```python
bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    aws_access_key_id=os.getenv('aws_access_key_id'),
    aws_secret_access_key=os.getenv('aws_secret_access_key'),
    region_name='us-west-2'
)
context = ConversationBufferMemory()
context.chat_memory.add_user_message(
    "You are a food expert and can recommend the best food to eat based on the user's taste.")
context.chat_memory.add_ai_message(
    "I am a food expert and will recommend the best food to eat based on the user's taste.")
ai21_llm = Bedrock(model_id="ai21.j2-ultra", client=bedrock_runtime)
ai21_llm.model_kwargs = {"maxTokens": 2048, 'temperature': 1.0, 'topP': 0.9}
conversation = ConversationChain(llm=ai21_llm, verbose=True, memory=context)
```
- **What it demonstrates**: instantiating the Bedrock runtime client, wiring a LangChain `ConversationBufferMemory` for context retention, and configuring inference parameters (`maxTokens`, `temperature`, `topP`) before invoking a model through `ConversationChain`.

## Reference Tables
| Provider | Model(s) | Strength |
|---|---|---|
| Amazon | Titan (text, image), Nova (Micro/Lite/Pro/Canvas/Reel) | unified pricing, up to 75% cheaper/token (Nova) |
| Anthropic | Claude family | safety-focused (constitutional + harmlessness training), strong reasoning, large context |
| Stability AI | Stable Diffusion | text-to-image, photorealism |
| Meta | Llama | efficient text generation |
| Cohere | Command (chat/knowledge), Embed (semantic search) | enterprise privacy controls |
| Mistral AI | Mistral 7B / 8x7B | open-source, fast inference, low memory |
| DeepSeek | DeepSeek R1 | open-weight (MIT), 75-90% cheaper reasoning/code tasks |

| Pricing model | When required | Billing basis |
|---|---|---|
| On-demand (consumption-based) | out-of-the-box models only | per input/output token |
| Provisioned throughput | fine-tuned / customized models (mandatory) | fixed hourly reserved capacity |

## Worked Example
**Scenario**: build a virtual assistant that summarizes news articles, walked through across all 7 life-cycle phases.
1. **Problem Definition**: scope = summarize long articles into concise, accurate output; decide up front whether to use a pretrained model as-is, fine-tune, or train new (author picks "use/adapt pretrained").
2. **Data Collection**: gather diverse news articles across domains; have human experts write reference summaries as ground truth for evaluation.
3. **Model Selection**: choose Anthropic Claude for its large context window and strong text generation/understanding — suited to long-input summarization.
4. **Training & Evaluation**: set criteria — relevance, conciseness, clarity — and iteratively test against the human-written ground truth summaries.
5. **Integration & Deployment**: Bedrock's serverless architecture deploys the assistant without infrastructure management; expose via web interface or embed in an existing platform.
6. **Monitoring, Security & Compliance**: track response accuracy, latency, and resource usage; enforce encryption and access control; run periodic compliance checks (GDPR/CCPA if applicable).
7. **Feedback & Improvement**: analyze user queries and summary accuracy over time; retrain/fine-tune as needed, potentially using A/B testing across model configurations.

**Why it works**: grounding the life-cycle framework in one running example makes each phase's deliverable concrete — e.g., "ground truth dataset" isn't abstract, it's "expert-written summaries for the same articles the model will summarize."

## Key Takeaways
1. Bedrock unifies many providers' FMs behind one API (`InvokeModel()`) but does not remove your responsibility for data quality, governance, or compliance.
2. The 7-phase life cycle (Problem Definition → Feedback & Improvement) is iterative, not linear — expect rework early on.
3. Fine-tuned/customized models require provisioned throughput pricing; on-demand pricing only covers out-of-the-box models.
4. Token consumption (input + output) drives cost — ~1 token ≈ 0.75 words; budget accordingly for high-volume workloads.
5. Guardrails and regional compliance (GDPR, CCPA, PDPL) should be architected in from Problem Definition, not bolted on after deployment.
6. Model choice is a fit exercise: context window, modality, cost, and safety profile all vary by provider (Claude vs. Nova vs. Mistral vs. DeepSeek).

## Connects To
- **Ch2**: prompt engineering is the practice referenced here for shaping `InvokeModel()` inputs effectively.
- **Ch4**: data collection/preparation for multimodal use cases is deferred to this chapter.
- **Ch6**: embeddings and knowledge bases introduced here are the foundation for RAG.
- **Ch7**: Model Evaluation and HITL capabilities mentioned here are covered in depth for performance optimization.
