---
name: using-amazon-bedrock
description: "Knowledge base from \"Using Amazon Bedrock: Learn to Architect, Secure, and Optimize Generative AI Applications on AWS\" by Renaldi Gondosubroto. Use when applying Bedrock's architecture patterns for prompt engineering, RAG, fine-tuning, multimodal models, security/guardrails, performance optimization, or building end-to-end generative AI applications on AWS."
---

# Using Amazon Bedrock: Learn to Architect, Secure, and Optimize Generative AI Applications on AWS

**Author**: Renaldi Gondosubroto | **Pages**: ~475 | **Chapters**: 10 | **Generated**: 2026-09-15

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference across any Bedrock task.
- **With a topic** — ask about `RAG`, `fine-tuning`, `guardrails`, `prompt engineering`, `text-to-SQL`, or another indexed topic; the relevant chapter is read on demand.
- **With a chapter** — ask for `ch06` (or by name) to load that specific chapter in full.
- **Browse** — ask "what chapters do you have?" to see the full index below.

When a question touches a topic not covered in Core Frameworks, read the relevant chapter file before answering — chapters carry the code examples, worked examples, and reference tables that make answers concrete rather than generic.

---

## Core Frameworks & Mental Models

**Bedrock's core value**: unifies many providers' foundational models (Claude, Titan, Nova, Llama, Mistral, DeepSeek, Stable Diffusion, Cohere) behind one API (`InvokeModel`/`InvokeModelWithResponseStream`), abstracting infrastructure while leaving data quality, governance, and compliance as the customer's responsibility.

**The 7-Phase Generative AI Solution Life Cycle** (Ch1): Problem Definition & Scope → Data Collection & Preparation → Model Selection & Tuning → Training & Evaluation → Integration & Deployment → Monitoring, Security & Compliance → Feedback & Improvement. Iterative, not linear — expect rework early on. Scales up to the 5-phase organizational project lifecycle (Ch9): Requirement Analysis & Design → Model Dev & Customization → Testing & Validation → Deployment & Scaling → Continuous Monitoring & Improvement.

**Customization decision (the book's central recurring framework, refined across Ch2/5/6/10)**: use prompt engineering when pretrained knowledge suffices and iteration speed matters; use RAG when you need current/external/proprietary facts injected at inference time without retraining; use fine-tuning when you need rigid, exactly-consistent output format or deep domain-jargon adaptation baked into weights; train from scratch only when nothing else fits. This same ladder is re-framed in Ch10 as a **sustainability ordering** — each step down costs more energy/carbon, so always try the lighter option first.

**RACCCA prompt evaluation framework** (Ch2): score any generated content on Relevance, Accuracy, Completeness, Clarity, Coherence, Appropriateness.

**Zero/One/Few-Shot Inference** (Ch2): examples provided *at inference time*, not training. Zero-shot for general tasks with resource constraints; one-shot for quick structural adaptation from a single example; few-shot (10-30%, up to 2x accuracy gains) when pattern reinforcement across a handful of examples helps — but costs the most tokens.

**Advanced prompting techniques** (Ch2): Chain of Thought (step-by-step reasoning, slower/costlier but more accurate and debuggable), Meta Prompting (ask the model to design the best prompt/strategy first — related to the Plan-and-Act paradigm), Templating (fixed structural scaffold for consistent format).

**RAG three-phase pipeline** (Ch6): Query (embed input) → Retrieval (semantic search top-N chunks via cosine similarity in a vector database) → Generation (augment prompt with retrieved chunks). Uniquely combines retrieval WITH true synthesis — unlike search engines (links) or semantic search (passages, no generation). Reduces hallucination but never eliminates it; pair with fallback disclaimers and vector-database write access controls.

**Fine-tuning decision (Ch5)**: fine-tune for Specialized Domains (legal/medical jargon), Repeated/Consistent Outputs (fixed schema), or Inference Cost Savings (shorter prompts at scale). Fine-tuned models require **provisioned throughput** (no on-demand option) — this is the single most important cost-risk fact in the book; always tear down provisioned throughput after testing.

**AWS deployment complexity/control spectrum** (Ch5): Bedrock (low complexity, low control, managed) → SageMaker (moderate, prebuilt algorithms + custom code) → EKS (high complexity, full infrastructure control). Pick based on how much infrastructure management your team wants to own.

**Multimodal prompting** (Ch4): multimodal prompts need significantly MORE detail than text-only prompts — vague prompts ("Beach picture") produce wildly inconsistent output; specify scene, style, composition explicitly. Seven best practices: Well-Crafted Prompts, Balanced Detail, Cultural/Societal Sensitivity, Syntactic Consistency, Iterative Testing and Refinement, Leveraging Model Strengths, Encouraging Creativity and Diversity.

**Performance optimization layers** (Ch7): Memory (pruning/quantization/mixed-precision), Compute (Chinchilla scaling laws — ~20:1 token:parameter ratio), Distributed Computing (data parallelism for large datasets, model parallelism for models exceeding node memory, hybrid for both), Evaluation (automatic first for objective metrics — accuracy/robustness/toxicity — human evaluation only for subjective nuance), Orchestration (Step Functions can invoke Bedrock directly or chain Lambda-mediated multi-stage prompts).

**Security: Shared Responsibility Model** (Ch8): AWS secures the cloud infrastructure; you secure your data, access configuration, and application logic — nothing security-related is automatic. **Generative AI Security Scoping Matrix** (5 scopes: Consumer App → Enterprise App → Pretrained Models → Fine-tuned Models → Self-Trained Models) determines how much governance/legal/risk/controls/resilience work is required. **STRIDE threat modeling** (Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege) is the recommended framework — apply during architecture design, not after deployment. **Guardrails** (denied topics, content filters, contextual grounding — grounding score + relevance score) constrain what the model can say; RAG improves what it does say — use both together for high-stakes domains. Never attach `AmazonBedrockFullAccess` to an agent — always scope IAM to least privilege.

**End-to-end pipelines** (Ch9): compose managed services, each doing one job — IoT Core (ingest) → DynamoDB (store) → Lambda (orchestrate) → Bedrock (interpret/generate) → API Gateway (expose) → S3 (present). Metadata (Glue Data Catalog, Bedrock Knowledge Base schema descriptions) is what makes AI-generated SQL/queries accurate — invest in it, don't treat it as optional. The retry-with-error-feedback pattern (feed execution errors back into regeneration) improves structured-output reliability.

**Sustainability & scale** (Ch10): the customization ladder re-framed by energy cost; zero-ETL integration (e.g., Aurora→Redshift) eliminates custom pipeline maintenance; batch inference gets a 50% cost discount for non-urgent bulk work; watermark detection verifies AI-generated vs. human-created content provenance.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-intro-to-generative-ai-on-aws.md) | Introduction to Generative AI on AWS | 7-Phase Solution Life Cycle, core Bedrock technologies (diffusion + transformers) |
| [ch02](chapters/ch02-prompt-engineering.md) | Prompt Engineering with Foundational Models on AWS | RACCCA, Zero/One/Few-Shot, Chain of Thought, Meta Prompting, Templating |
| [ch03](chapters/ch03-building-applications-bedrock-api.md) | Building Applications with the Amazon Bedrock API | 4-endpoint decision model, streaming, prompt caching, exception handling |
| [ch04](chapters/ch04-multimodal-foundational-models.md) | Working with Multimodal Foundational Models | 7 multimodal prompting best practices, model-strength matching |
| [ch05](chapters/ch05-fine-tuning-foundational-models.md) | Fine-Tuning Foundational Models on AWS | Fine-tune vs. prompt-engineer decision, Bedrock/SageMaker/EKS spectrum, PEFT |
| [ch06](chapters/ch06-retrieval-augmented-generation.md) | Performing Retrieval-Augmented Generation on AWS | 3-phase RAG pipeline, RAG vs. alternatives, ReAct agents |
| [ch07](chapters/ch07-optimizing-performance.md) | Optimizing Performance for Foundational Models | Memory optimization, Chinchilla scaling, distributed computing spectrum, RL |
| [ch08](chapters/ch08-security-and-privacy.md) | Security and Privacy for Deploying Generative AI Architectures on AWS | Security Scoping Matrix, STRIDE, guardrails, contextual grounding, IAM |
| [ch09](chapters/ch09-building-end-to-end-applications.md) | Building End-to-End Applications with Generative AI | End-to-end lifecycle, 3-tier unstructured-data architecture, text-to-SQL |
| [ch10](chapters/ch10-sustainability-and-scalability.md) | Sustainability and Scalability with Amazon Bedrock | Sustainability-ordered customization ladder, zero-ETL, batch inference |

## Topic Index

- **Agents (Bedrock)** → ch06, ch08
- **Athena (Amazon)** → ch09, ch10
- **Batch inference** → ch10
- **Bedrock API (InvokeModel)** → ch01, ch03
- **Chain of Thought reasoning** → ch02
- **Chinchilla scaling laws** → ch07
- **CloudFormation** → ch07
- **CloudWatch / CloudTrail** → ch08
- **Contextual grounding** → ch08
- **Cost optimization** → ch02, ch03, ch07, ch10 (see cheatsheet.md)
- **Embeddings (Titan)** → ch01, ch04, ch06
- **End-to-end pipelines** → ch09
- **Exception handling (Bedrock errors)** → ch03
- **Fine-tuning** → ch05
- **Guardrails** → ch08
- **IAM policies / least privilege** → ch08
- **Image generation (Stable Diffusion)** → ch03, ch04
- **Inpainting / outpainting** → ch04
- **IoT (AWS IoT Core)** → ch09
- **Knowledge bases** → ch06, ch09
- **LangChain** → ch01, ch03, ch06
- **Model evaluation (automatic/human)** → ch07
- **Multimodal models** → ch04
- **Parameters (temperature, top_p, top_k)** → ch02, ch03
- **PEFT / LoRA** → ch05
- **Playground (Bedrock/PartyRock)** → ch01, ch02
- **Prompt caching** → ch03, ch07
- **Prompt engineering** → ch02
- **Prompt injection / jailbreaks** → ch02
- **RAG (Retrieval-Augmented Generation)** → ch06
- **ReAct framework** → ch06
- **Reinforcement Learning (RL)** → ch07
- **Security Scoping Matrix** → ch08
- **Step Functions** → ch07, ch09
- **STRIDE threat modeling** → ch08
- **Sustainability** → ch10
- **Text-to-SQL** → ch09
- **Vector databases (FAISS, OpenSearch, Pinecone)** → ch06
- **Visual Question Answering (VQA)** → ch04
- **Watermark detection** → ch10
- **Zero-ETL** → ch10

## Supporting Files

- [patterns.md](patterns.md) — all concrete techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — decision rules, thresholds, quick-reference tables

---

## Scope & Limits

This skill covers the book's 10 main chapters only. Appendix A (Configuring Your AWS Account) and Appendix B (Installing Python, Jupyter Notebook, and LangChain) were excluded from this conversion per user request — they're setup instructions with no durable framework content. For hands-on implementation in your own codebase, combine with project-specific tools and current AWS documentation (the book's code snippets reference specific model IDs and API shapes that AWS updates over time — verify against current docs before production use).
