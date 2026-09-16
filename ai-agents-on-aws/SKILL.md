---
name: ai-agents-on-aws
description: "Knowledge base from \"AI Agents on AWS: Beginner's Guide to Building AI Agents on AWS\" by Bunny Kaushik and Mona M. Use when applying Strands Agents SDK patterns, Bedrock AgentCore (Runtime, Memory, Gateway, Identity, Policy), multi-agent architectures (Supervisor-Worker, Swarm, Graph), MCP/A2A protocols, or agent evaluation/observability/governance, studying the book, or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# AI Agents on AWS: Beginner's Guide to Building AI Agents on AWS
**Author**: Bunny Kaushik, Mona M (Packt Publishing) | **Pages**: ~272 (7 content chapters) | **Chapters**: 7 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `agent memory`, `MCP`, `multi-agent patterns`, `AgentCore Runtime`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

**The core loop: RAG retrieves and answers; an agent plans, acts, observes, and re-plans.** If a user wants something *done* rather than *answered*, a passive RAG pipeline will disappoint — that gap is exactly why agents exist. Every agent decomposes into four parts: Tools, Memory, Planning, and an LLM "brain" — use this as a design checklist.

**Diagnose maturity before building**: LLM → LLM+RAG → Augmented LLM (tools+memory) → Agentic workflow (plan/execute/synthesize) → Autonomous agent (persistent, self-correcting) → Multi-agent collaboration. Build only as far up this ladder as the problem requires — don't reach for multi-agent collaboration when a single Augmented LLM with tools solves it.

**AWS Agentic Stack has 3 tiers**: Amazon Q (zero-config) → Bedrock + AgentCore (managed, moderate effort — this book's focus) → SageMaker AI (full control, DIY). Pick the tier matching your required control/effort trade-off; don't over- or under-provision.

**Tool design**: build single-purpose tools (`get_sales_data`, not a mega-tool with modes) using the `@tool` decorator — type hints + docstring are the full contract the agent reasons over. Use class-based tools with a shared `__init__` when tools share an expensive resource (DB connection) to avoid connection exhaustion under concurrent load. Use `async def` tools only for slow I/O called multiple times with independent inputs.

**Memory follows CoALA**: split long-term memory into Episodic (what happened), Semantic (what the agent knows), Procedural (how it gets things done), plus short-term/working memory for the active session. On AWS, Bedrock AgentCore Memory implements this with 4 namespace levels (Global → Strategy → Actor → Session) — always pick the *narrowest* namespace that correctly scopes the data. Manage context actively (sliding window, compaction, structured note-taking, sub-agent delegation) before "context rot" degrades reasoning.

**Multi-agent architecture: pick by structure, not preference.** Supervisor-Worker (centralized, easy to debug, default choice — but the supervisor is a bottleneck/SPOF and workers can't talk directly). Agent-as-Tool (a reusable specialist wrapped with a `description` so other agents can call it like a tool). Swarm (agents hand off peer-to-peer via `handoff_to_agent` when the path can't be predicted upfront — watch for ping-pong loops). Graph/DAG (explicit nodes+edges when you must prove a specific order of checks happened — compliance/auditability workloads).

**Communication protocols are complementary, not competing.** MCP standardizes agent↔tool/data (collapses N×M integrations to N+M); A2A standardizes agent↔agent (opaque agents, discovered via Agent Cards, for cross-team/cross-org delegation). Never collapse an agent into an MCP tool to avoid using A2A — tools are for stateless predefined actions, agents solve open-ended iterative problems.

**Deployment target follows workload shape.** Lambda: short-lived, stateless, unpredictable traffic, scales to zero (15-min ceiling, cold starts). ECS/Fargate: multi-turn, warm state, persistent connections, always-on. Bedrock AgentCore Runtime: multi-turn, per-user-isolated (dedicated microVM per session, sanitized on termination), managed auth/policy/observability built in.

**Production trustworthiness needs three connected layers**, not one: Evaluation (did the agent do the right thing — attach metrics at session/trace/span level; built-in evaluators cover general quality, custom evaluators are required for domain-specific rules a generic metric can't see). Observability (what did the agent actually do — 4 layers, with Layer 3 "agent execution traces" being the layer generic APM tools don't give you; a 200 OK can still hide hallucination or a 3x cost blowup). Governance (what is the agent allowed to do — Bedrock Guardrails control *what it says*, Cedar policy in AgentCore Policy controls *what it does*, enforced deterministically outside the model's reasoning; never rely on a system-prompt instruction as a compliance control for anything with real consequences).

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-understanding-ai-agents-on-aws.md) | Understanding AI Agents on AWS | RAG vs. Agent, Agent components (Tools/Memory/Planning/LLM), AWS Agentic Stack, Maturity ladder, MCP, A2A |
| [ch02](chapters/ch02-building-agents-with-tools.md) | Building Agents with Tools | `@tool` decorator, Single-purpose tool design, Class-based tools, Async tools, Strands Agent Builder |
| [ch03](chapters/ch03-agent-memory.md) | Agent Memory | CoALA, Long-term memory types, Context management, Memory optimization strategies, AgentCore Memory, Namespaces |
| [ch04](chapters/ch04-advanced-agent-architecture-patterns.md) | Advanced Agent Architecture Patterns | Supervisor-Worker, Agent-as-Tool, Swarm, Graph (DAG/cyclic) |
| [ch05](chapters/ch05-agent-communication.md) | Agent Communication | MCP (transports, lifecycle, server/client features), FastMCP vs FastAPI, A2A (styles, implementation layers) |
| [ch06](chapters/ch06-production-deployment-and-enterprise-integration.md) | Production Deployment and Enterprise Integration | Production-readiness checklist, Lambda, ECS, AgentCore Runtime, AgentCore Gateway, AgentCore Identity |
| [ch07](chapters/ch07-evaluation-observability-and-ai-governance.md) | Evaluation, Observability, and AI Governance | Session/Trace/Span hierarchy, Evaluation process & levels, Observability layers, AI Governance (Guardrails/Policy/Org) |

## Topic Index

- **A2A (Agent-to-Agent Protocol)** → ch01, ch05
- **AgentCore Gateway** → ch06
- **AgentCore Identity** → ch06
- **AgentCore Memory** → ch03
- **AgentCore Namespaces** → ch03
- **AgentCore Runtime** → ch06
- **Agent-as-Tool pattern** → ch04
- **Agent evaluation** → ch07
- **Agentic RAG** → ch01
- **Async tools** → ch02
- **AWS Agentic Stack** → ch01
- **Cedar policy language** → ch07
- **CoALA cognitive architecture** → ch03
- **Context management** → ch03
- **Deployment (Lambda/ECS/AgentCore)** → ch06
- **Governance** → ch07
- **Graph pattern (DAG/cyclic)** → ch04
- **Guardrails vs Policy** → ch07
- **Maturity ladder** → ch01
- **MCP (Model Context Protocol)** → ch01, ch05
- **Memory optimization strategies** → ch03
- **Multi-agent patterns** → ch04
- **Observability (4 layers)** → ch07
- **OpenTelemetry (OTel)** → ch07
- **Production-readiness checklist** → ch06
- **Single-purpose tool design** → ch02
- **Strands Agents SDK** → ch01, ch02
- **Supervisor-Worker pattern** → ch04
- **Swarm pattern** → ch04
- **Tool decorator (`@tool`)** → ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond this book, check related skills
or ask the agent directly.

127 source images (figures, diagrams, screenshots) were not read during extraction — where a chapter references "Figure N.N," the prose around it was used, but the image itself was not.

The book's own "Chapter 8: Unlock Your Exclusive Benefits" is Packt promotional/back-matter about a companion web platform, not technical content — it was excluded from this skill.

This repo also has two other AWS/Bedrock-related skills — `using-amazon-bedrock` (a different, broader Bedrock book) and `building-gen-ai-applications-with-amazon-bedrock` (Bedrock architecture end-to-end). This skill is specifically about the *agents* layer (Strands SDK, AgentCore, MCP/A2A, multi-agent patterns) — prefer it for agent-building questions, and the others for general Bedrock service/architecture questions.
