---
name: building-gen-ai-applications-with-amazon-bedrock
description: "Knowledge base from \"Building Gen AI Applications with Amazon Bedrock: Architect Foundation Models into Secure, Scalable Generative AI Solutions\" by Syed Kadar Ansari Syed Ahamed. Use when applying Bedrock architecture patterns for prompt engineering, RAG/Knowledge Bases, fine-tuning (PEFT/LoRA), model selection, resilience (circuit breakers, fallback), scaling/cost, or Bedrock Agents/MCP-based agentic AI, studying the book, or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Building Gen AI Applications with Amazon Bedrock
**Author**: Syed Kadar Ansari Syed Ahamed | **Pages**: ~23 (EPUB, dense) | **Chapters**: 8 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `RAG`, `fine-tuning`, `circuit breaker`, `Bedrock Agents`, or another indexed topic; I find and read the relevant chapter
- **With a chapter** — ask for `ch05`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read the relevant chapter file before answering.

Note: this repo also has an unrelated skill `using-amazon-bedrock` (from a different book, by Renaldi Gondosubroto). Don't conflate the two — this skill is specifically Syed Kadar Ansari Syed Ahamed's book.

---

## Core Frameworks & Mental Models

**Escalation ladder for any AI-engineering requirement**: prompt engineering (zero-shot → few-shot → chain-of-thought) → RAG → fine-tuning (PEFT/LoRA → full fine-tuning). Try the cheapest rung first; escalate only when it plateaus. Missing/outdated knowledge → RAG. Wrong style/format prompting can't fix → fine-tuning.

**Sampling strategy** (Table 3.7, the book's most-reused decision table): default to **top-p (nucleus) sampling** for open-ended natural language generation. Use greedy decoding for deterministic/structured extraction. Use temperature sampling for global creativity tuning. Use top-k for a fixed randomness ceiling.

**RAG Pipeline (5 stages)**: Document Ingestion & Preprocessing (load/clean/chunk) → Embedding (Titan Embeddings) → Vector Storage (OpenSearch Service or Aurora+pgvector) → Retrieval (semantic search) → Augmented Generation (inject retrieved chunks into the prompt). Match chunking strategy (fixed-size → recursive → structural → semantic → agentic) to document nature; RAG quality problems are almost always chunking/retrieval problems, debug there first.

**Model selection is per-workload, not platform-wide**: use the "Bedrock model bucket" — cheap/fast models (Amazon Nova Micro/Lite) for routing and simple tasks, mid-tier for typical generation, premium (Claude Sonnet/Opus) only for complex reasoning. Re-evaluate this at scale using the performance/price matrix (Ch6), not just capability (Ch1/Ch3).

**Resilience is layered, not singular**: exponential backoff with jitter (transient errors, via Boto3's built-in retry config — never hand-rolled `time.sleep()`) → Circuit Breaker (Closed/Open/Half-Open state machine, for persistent failures) → Model Fallback chain (graceful degradation) → Cross-Region Inference (regional-outage-level resilience). Each targets a different failure severity.

**IAM is a dial, progressively narrowed**: start broad (`AmazonBedrockFullAccess`) to unblock learning, narrow to specific action prefixes (`bedrock:InvokeModel`, etc.) before production — never ship broad policies unchanged.

**Bedrock's two client surfaces**: `boto3.client('bedrock')` = control plane (model/guardrail/customization management); `boto3.client('bedrock-runtime')` = data plane (actual inference — `InvokeModel`, `Converse`). Conflating them is a common source of confusing permission errors.

**Agentic AI = reasoning + secure execution, not just a better model**: an agent needs AgentCore-style secure/observable execution infrastructure to safely take real-world actions (Action Groups/Lambda, MCP tools) — model capability alone is necessary but not sufficient. The ReAct loop (Plan → Action → Observation) is both the design pattern and the debugging lens for agent behavior.

**Prompts and invocations are auditable artifacts in production**: version prompts with semantic versioning (MAJOR.MINOR.PATCH); log the full Audit Log Schema (InvocationID, ModelID, PromptTemplateID/Version, FinalPrompt, InferenceParameters, RawModelResponse, GuardrailEvaluation, TraceID, etc.) for any regulated/business-critical call — this is what makes an AI decision defensible after the fact.

**Experience Engineering, beyond functional prompting**: for customer-facing/brand-representing agents, deliberately engineer persistent persona (system prompts), brand-specific voice (fine-tuning where needed), and per-interaction-type temperature (low for factual/validating, higher for creative/brand responses) — functional correctness alone doesn't guarantee brand alignment.

**MCP (Model Context Protocol)**: the "USB-C port for AI" — a standardized interface for models to reach external tools/data without bespoke per-tool integration code; use it instead of hand-rolled integrations when an agent needs multiple external systems.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started-with-bedrock.md) | Getting Started with Amazon Bedrock | GenAI vs Traditional AI table, Foundation Model Selection Matrix |
| [ch02](chapters/ch02-setting-up-your-environment.md) | Setting Up Your Environment | Essential IAM Policies, Playground-first validation, bedrock vs bedrock-runtime |
| [ch03](chapters/ch03-foundational-models-in-bedrock.md) | Foundational Models in Amazon Bedrock | Sampling Strategy Decision Table, LLM Scaling Tradeoffs, Adaptation Method spectrum, RLHF/DPO/ORPO/KTO |
| [ch04](chapters/ch04-developing-generative-ai-solutions.md) | Developing Generative AI Solutions | Prompt Design Patterns, RAG Pipeline, Chunking Strategy Selection |
| [ch05](chapters/ch05-integrating-bedrock-with-workflows.md) | Integrating Bedrock with Existing Workflow | Exponential Backoff+Jitter, Batch vs Real-Time, Decouple Prompt Logic |
| [ch06](chapters/ch06-scaling-generative-ai-applications.md) | Scaling Generative AI Applications | Circuit Breaker, Model Fallback, Cross-Region Inference, Cost governance |
| [ch07](chapters/ch07-advanced-use-cases-industry-applications.md) | Advanced Use Cases and Industry Applications | Prompt Semantic Versioning, Audit Log Schema, Multi-Agent Collaboration |
| [ch08](chapters/ch08-future-of-genai-with-bedrock.md) | Future of Generative AI with Bedrock | ReAct Loop, Agentic AI vs Generative AI, MCP, Experience Engineering |

## Topic Index

- **Action Groups / Bedrock Agents** → ch08
- **Adaptation methods (zero-shot → fine-tuning ladder)** → ch03, ch04
- **AgentCore / MCP** → ch08
- **Audit logging / reproducibility** → ch07
- **Batch vs real-time inference** → ch05, ch06
- **Chunking strategies** → ch04
- **Circuit Breaker / resilience patterns** → ch06
- **Cost management (Budgets, Cost Explorer)** → ch06
- **Fine-tuning (PEFT/LoRA/QLoRA, full)** → ch03, ch04
- **IAM policies / least privilege** → ch02, ch05
- **Model selection (capability)** → ch01, ch03
- **Model selection (cost/performance)** → ch06
- **Prompt engineering patterns (zero/few-shot, CoT)** → ch04
- **Prompt versioning / audit schema** → ch07
- **RAG / Knowledge Bases** → ch04, ch07
- **RLHF / DPO / ORPO / KTO alignment** → ch03
- **Sampling strategies (temperature/top-k/top-p)** → ch03
- **Self-attention / Transformer architecture** → ch03
- **Setup (AWS account, IAM, SDK, Playground)** → ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. 65 source images (diagrams, screenshots) were dropped during extraction and not read — figure/screenshot content referenced in chapters is described from surrounding text, not the image itself. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills (this repo also has `using-amazon-bedrock`, from a different author/book, and `ai-agents-in-action`) or ask the agent directly.
