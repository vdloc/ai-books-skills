---
name: enterprise-generative-agentic-ai
description: "Knowledge base from \"Enterprise Guide for Implementing Generative AI and Agentic AI: A Practical Guide to Developing, Deploying, and Operationalizing AI-Driven Applications for Enterprise Use\" by Shakuntala Gupta Edward, Rahul Bhattacharya, and Vikas Sinha. Use when designing enterprise GenAI/agentic architectures, building multi-agent orchestration systems, implementing RAG pipelines, evaluating LLM outputs, deploying to Kubernetes/cloud, or applying Responsible AI and risk-management frameworks."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Enterprise Guide for Implementing Generative AI and Agentic AI

**Authors**: Shakuntala Gupta Edward, Rahul Bhattacharya, Vikas Sinha | **Pages**: ~414 | **Chapters**: 8 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference across any enterprise GenAI/agentic task.
- **With a topic** — ask about `agentic AI`, `RAG`, `evaluation metrics`, `responsible AI`, `deployment`, or another indexed topic; the relevant chapter is read on demand.
- **With a chapter** — ask for `ch04` (or by name) to load that specific chapter in full.
- **Browse** — ask "what chapters do you have?" to see the full index below.

When a question touches a topic not covered in Core Frameworks, read the relevant chapter file before answering — chapters carry the worked examples, real code (FastAPI agents, Azure pipelines, Helm charts), and reference tables that make answers concrete.

---

## Core Frameworks & Mental Models

**LLM Fit Assessment** (Ch 8, Ch 4): before building anything, ask "does this need an LLM?" Weigh inference cost, hallucination risk, latency, and maintenance overhead against benefit. Structured extraction/lookup tasks are usually better solved by rule-based parsers or classical ML. Use LLM utility "High" only where reasoning/creativity is genuinely central.

**Agentic Workflow Evolution** (Ch 4): Manual → RPA (rigid, breaks on unstructured data) → AI+RPA/"intelligent automation" (still largely static) → AI Agents (LLM-powered reasoning/planning) → Multi-agent systems (specialized, collaborating agents). Diagnose automation maturity by placing a system on this spectrum.

**Agent Registry & Discovery** (Ch 4): a structured catalog (name, task, I/O schema, engine type, capabilities, endpoint, data-residency) letting an orchestrator dynamically find and invoke agents as black boxes. Essential past a handful of agents — prevents duplicated development.

**Orchestrator Pattern** (Ch 4): the LLM-driven "brain" that plans which tools/agents to call and consolidates results, typically via two LLM calls (plan with `tools` param + `tool_choice="auto"`, then execute/consolidate). Agents are black boxes to it — only interface matters, never internals.

**Model Context Protocol (MCP)** (Ch 4): standardized client-server protocol for agent/LLM access to external resources and tools (`resources/list`, `resources/read`, `tools/list`, `tools/call`) — replaces N×M bespoke integrations once a system touches 3+ external systems. Think of it as a master key vs. a key per room.

**Self-Reflection Pattern** (Ch 4): pair a generator with a critic step that scores output against a checklist and triggers regeneration until threshold. Materially improves quality over prompt engineering alone, but costs working memory, latency, and $ per layer — add only where the accuracy gain justifies it.

**Design Patterns for Enterprise GenAI** (Ch 3): Pipeline (structured ingestion), Model Factory (multi-version deployment), Microservice Architecture (independent scaling), A/B Testing vs. Shadow Testing (live comparison vs. silent validation — prefer Shadow for compliance-sensitive changes), Federated Learning + Differential Privacy + Zero Trust (the standard enterprise privacy/security stack for GDPR/HIPAA/PCI-DSS), Blue-Green/Canary Deployment (zero-downtime rollout).

**Medallion Data Architecture** (Ch 5): Raw (immutable source of truth) → Masked (PII removed/encrypted) → Segmented (chunked, ≤2MB) → Analytical/Silver (cleaned, enriched) → Gold (query-optimized). Never skip the immutable raw layer even after masking — it's what makes reprocessing possible.

**Business → Technical Objective → KPI Chain** (Ch 5): every technical objective must trace back to a measurable business goal and KPI. A technical objective with no traceable business objective "risks becoming a standalone innovation without clear ROI." Use MoSCoW + RACI + RTM together during requirements gathering.

**Context Window as Primary Model-Selection Criterion** (Ch 5): pick context window size before secondary criteria like pricing — an undersized window forces costly multi-pass segmentation (book's own team hit this with GPT-4-32k before switching to GPT-4o's 128K).

**Prompt Tuning > Fine-Tuning by default** (Ch 5, Ch 8): prompt tuning is fast, cheap, and iterable; fine-tune only when domain nuance genuinely can't be captured by prompting. Prefer LoRA/QLoRA over full fine-tuning when tuning is justified.

**Four Pillars of GenAI Evaluation** (Ch 6): Quality & Relevance (BLEU/ROUGE/FID, factual accuracy), Safety & Robustness (jailbreak/adversarial resistance), Efficiency & Scalability (latency/throughput/compression), Alignment with Human Values & Ethics (fairness, transparency, bias audits). A system strong on Quality alone is not evaluation-complete.

**Nondeterministic Evaluation Problem** (Ch 6): the same GenAI input can produce different valid outputs — evaluation must consider a spectrum of acceptable responses via a blend of statistical, qualitative, and quantitative methods, not a single ground truth.

**Deployment Strategy Selection** (Ch 6): choose Cloud/On-Prem/Hybrid/Edge from data sensitivity + latency + cost + regulatory constraints, not from what's trendiest. Six topologies (single-node, multinode/clustered, microservices, serverless, distributed, cloud-edge hybrid) map to specific scale/latency/management-complexity trade-offs.

**Responsible AI — 9 Principles** (Ch 7): Fairness, Transparency, Reliability, Robustness, Safety, Privacy & Security, Accountability, Inclusiveness, Sustainability — threaded through every life-cycle stage, not a final review gate. Principles frequently conflict (e.g., Transparency vs. Privacy) — use the matched mitigation technique (masking/differential privacy) rather than picking a side.

**Risk Classification (3 axes)** (Ch 7): by Source (data/model/human/process/third-party), by Nature (technical/ethical/legal/security/operational/psychological/reputational/economic/environmental), by Intent (unintentional/intentional/accidental/systemic). Risk score = weighted(inherent [highest], transient, performance [lowest]).

**NIST AI RMF (Govern-Map-Measure-Manage)** (Ch 7): the operational "how" complementing RAI's ethical "what." Govern (policies/culture), Map (define allowed/disallowed use, impact), Measure (metrics, red-teaming), Manage (mitigate, prioritize, evaluate) — iterative, not one-shot.

**Total-Cost-of-Ownership over Per-Call Cost** (Ch 8): a higher-latency, higher-accuracy approach (e.g., multipass reasoning) is often net-cheaper once human rework/escalation cost is included — the book's own numbers show a $5,500/month net saving despite 50% higher inference cost. Always compute TCO before rejecting the more expensive-looking option.

**Security-by-Design Checklist** (Ch 5, Ch 8): RBAC at every layer, encryption at rest/in transit, never expose inference endpoints publicly, field-level PII/PCI/PHI classification and masking, secrets in a vault (never hardcoded), full audit logging into SIEM, principle of least privilege everywhere.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-evolution-of-ai-and-llms.md) | Evolution of AI and Large Language Models | GOFAI vs. connectionism, Turing test, Transformer architecture |
| [ch02](chapters/ch02-genai-in-business.md) | Generative AI in Business Unlocking Value | Five GenAI application categories, value-driver mapping |
| [ch03](chapters/ch03-design-patterns-enterprise-genai.md) | Design Patterns for Developing Enterprise GenAI Applications | Pipeline/Model Factory/Microservice patterns, A/B vs. Shadow testing, Federated Learning, Zero Trust |
| [ch04](chapters/ch04-introduction-to-agentic-ai.md) | Introduction to Agentic AI | Agent registry, orchestrator, memory, self-reflection, MCP |
| [ch05](chapters/ch05-end-to-end-practical-use-case.md) | End-to-End Implementation of a Practical Use Case | MoSCoW/RACI/RTM, medallion architecture, model selection, prompt tuning lifecycle |
| [ch06](chapters/ch06-evaluation-and-deployment.md) | Evaluation and Deployment | Four evaluation pillars, HELM, LLMOps, deployment strategies/topologies |
| [ch07](chapters/ch07-responsible-ai-and-risk-framework.md) | Responsible AI and Risk Framework | RAI principles, risk taxonomy/tiering, NIST AI RMF |
| [ch08](chapters/ch08-best-practices.md) | Best Practices | LLM fit assessment, modular design, security-by-design, active learning |

## Topic Index

- **Agent registry / discovery** → ch04
- **Agentic AI, agentic workflow** → ch01, ch02, ch03, ch04
- **A/B testing vs. shadow testing** → ch03, ch06
- **Best practices (design/implementation/evaluation)** → ch08
- **Business/technical objectives, KPIs** → ch05
- **Cloud/on-prem/hybrid/edge deployment** → ch06
- **Data pipeline / medallion architecture** → ch05
- **Deployment topologies** → ch06
- **Design patterns (general)** → ch03
- **Evaluation metrics (text/code/image)** → ch06
- **Federated learning, differential privacy, zero trust** → ch03, ch07
- **Fine-tuning vs. prompt tuning** → ch05, ch08
- **Hallucination** → ch01, ch06, ch07
- **HELM benchmark** → ch06
- **LLMOps** → ch06
- **Memory (working/long-term)** → ch04
- **Model Context Protocol (MCP)** → ch04
- **Model selection / benchmarking** → ch05
- **NIST AI Risk Management Framework** → ch07
- **Orchestrator, orchestration** → ch04
- **Prompt engineering / management / compression** → ch05
- **RAG (retrieval-augmented generation)** → ch04, ch05, ch08
- **Responsible AI principles** → ch07
- **Risk classification, risk tiering** → ch07
- **Security-by-design** → ch05, ch08
- **Self-reflection pattern** → ch04
- **Transformer architecture, LLM history** → ch01

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills (e.g., `hands-on-rag-for-production` for deeper RAG-specific coverage) or ask the agent directly.
