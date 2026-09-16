---
name: ai-enhanced-web-apps
description: "Knowledge base from \"Build AI-Enhanced Web Apps: How to get reliable results with React, Next.js, and Vercel\" by Theo Despoudis. Use when applying the Vercel AI SDK, LangChain.js, RAG, prompt engineering, agentic tool calling, MCP integration, or production security/deployment patterns for generative AI web apps built with React/Next.js."
---

# Build AI-Enhanced Web Apps: How to Get Reliable Results with React, Next.js, and Vercel
**Author**: Theo Despoudis | **Pages**: ~394 | **Chapters**: 12 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference across any generative-AI-in-React/Next.js task.
- **With a topic** — ask about `streaming`, `RAG`, `tool calling`, `prompt engineering`, `MCP`, `rate limiting`, or another indexed topic; the relevant chapter is read on demand.
- **With a chapter** — ask for `ch07` (or by name) to load that specific chapter in full.
- **Browse** — ask "what chapters do you have?" to see the full index below.

When a question touches a topic not covered in Core Frameworks, read the relevant chapter file before answering — chapters carry the code examples, worked examples, and reference tables that make answers concrete rather than generic.

---

## Core Frameworks & Mental Models

**The stack**: React (UI) + Next.js (frontend/backend, file-based routing) + Vercel AI SDK (LLM connection, streaming, generative UI) + LangChain.js (chains, retrieval, agents) + MCP (cross-framework tool portability). Default models: Google Gemini, with OpenAI as an alternative.

**Language model specification (abstract factory pattern)** — `generateText`/`streamText`/`generateObject`/`streamObject` are provider-agnostic; each provider package (`@ai-sdk/openai`, `@ai-sdk/google`, `@ai-sdk/anthropic`, ...) is a concrete factory. Swapping providers is a one-line `model` change. Never call a provider's raw client SDK directly once more than one provider is in play — this is the single most load-bearing idiom in the book, reused all the way through `ai-fallback` model fallback (Ch8) and MCP tool exposure (Ch12).

**Streaming is a UX necessity, not a nicety** — real-world LLM throughput is slow (~21 tok/s for GPT-4 in the book's benchmark). Use `streamText` + `useChat` for any user-facing conversational UI; `generateText` only for one-shot, non-interactive generation.

**Generative UI, not just generative text** — `streamUI`/`createStreamableUI` let the server stream whole rendered React components (cards, tables) instead of text the client has to interpret. Reach for this whenever the AI's output should *be* a UI element.

**Structured output over string parsing** — `generateObject`/`streamObject` + a Zod schema gives typed, auto-retried output. Never hand-parse LLM text with regex when this is available.

**Tool calling: three escalating levels** — (1) Vercel AI SDK tool calling (Ch4): one fixed, developer-registered tool, model decides when to call it. (2) LangChain ReAct agents (Ch6): dynamic, iterative, multi-tool reasoning loop (decide → act → observe, repeat). (3) MCP (Ch12): tools become portable across frameworks entirely, via a standardized client-server protocol — the LLM never touches external APIs directly, only tool definitions.

**RAG's four components, always separable**: document indexing (chunk + embed + store) → retrieval mechanism (similarity search, top-k) → augmentation layer (merge context + query into a prompt) → generation engine (LLM). Always embed queries with the *same* model used to embed stored content. Always return `sourceDocuments` alongside the answer for transparency/trust. Add a grounding/verification layer when hallucination risk is high — retrieval alone doesn't guarantee a faithful answer.

**Summarization strategy is a length/coherence trade-off**: stuffing (short docs, single call) → MapReduce (long docs, parallel, less coherent) → refine (long docs, sequential, coherent, but errors compound).

**Prompt engineering techniques, from cheap to expensive**: few-shot learning (inline examples) and chain-of-thought (explicit step-by-step instruction) are training-free and cheap; embeddings + retrieval grounds answers in real data; Tree of Thoughts / Self-Refine / LLM-as-a-judge are heavier techniques for reasoning depth and automated evaluation.

**Production reliability is layered, not single-point**: proactive limits (`maxTokens`, rolling context window) + reactive recovery (`maxRetries` for transient 429/5xx, `ai-fallback` for persistent provider outages) + user-facing error classification (`AIErrorTracker`). Distinguish retryable (429, temporary 5xx) from non-retryable (quota exhaustion) errors explicitly.

**Security is a pipeline, ordered by cost**: threat model → server-side validation (Zod) → composable middleware (CORS → rate limit → auth → security headers, in that order, cheapest rejects first) → per-user daily quota (checked *after* rate limiting) → PII redaction at the input boundary (before the LLM or logs ever see raw input) → secrets never in `NEXT_PUBLIC_` vars.

**Testing LLM apps: mock only your own logic** — mock input sanitization, response formatting, error handling, flow control; never mock evolving model capability (reasoning quality, sentiment accuracy) — that requires real model calls. Use the SDK's own `MockLanguageModelV1`/`simulateReadableStream` test doubles.

**Multi-tenant RAG**: default to one shared vector store with strict namespacing/metadata filtering by tenant ID on every query and write; reserve dedicated per-tenant stores for genuine regulatory/compliance requirements.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-using-generative-ai-in-web-apps.md) | Using Generative AI in Web Apps | Component stack, 4-stage interaction flow, model type selection |
| [ch02](chapters/ch02-building-your-first-app.md) | Building Your First Generative AI Web App | 10-step chat lifecycle, custom hook construction, Next.js file-based routing |
| [ch03](chapters/ch03-connecting-with-vercel-ai-sdk.md) | Connecting AI Models with the Vercel AI SDK | Language model specification, streaming, `useChat`/`useCompletion`, vision |
| [ch04](chapters/ch04-managing-conversation-and-state.md) | Managing Conversation and State | RSC generative UI, `streamUI`, tool calling, `generateObject`/Zod |
| [ch05](chapters/ch05-prompt-engineering.md) | Prompt Engineering in Web Applications | Prompt types, few-shot, chain-of-thought, embeddings |
| [ch06](chapters/ch06-building-ai-workflows-with-langchain.md) | Building AI Workflows with LangChain.js | LCEL runnables, text splitters, vector stores, ReAct agents, memory |
| [ch07](chapters/ch07-document-summarization-and-rag.md) | Document Summarization and RAG with LangChain.js | MapReduce/stuffing/refine, RAG architecture, HNSWLib, grounding |
| [ch08](chapters/ch08-testing-and-debugging.md) | Testing and Debugging Techniques | Error tracking, token/rate limits, `ai-fallback`, mocking strategy |
| [ch09](chapters/ch09-deployment-and-security.md) | Deployment and Security | Threat modeling, middleware pipeline, rate limiting, PII redaction, deployment tiers |
| [ch10](chapters/ch10-ai-interview-assistant-walkthrough.md) | AI Interview Assistant (Project) | Redis session store, feature flags, cached LLM output |
| [ch11](chapters/ch11-ai-rag-agent-walkthrough.md) | AI RAG Agent (Project) | Multi-tenant vector store, upload security, API/URL design |
| [ch12](chapters/ch12-integrating-web-apps-with-mcp.md) | Integrating Web Apps with MCP | MCP client-server architecture, stdio transport, gateways |

## Topic Index

- **Agents (ReAct)** → ch06
- **Chain-of-thought prompting** → ch05
- **Embeddings** → ch05, ch06, ch07
- **Error handling / fallback** → ch08
- **Feature flags** → ch10
- **Few-shot learning** → ch05, ch06
- **File-based routing (Next.js)** → ch02
- **Fine-tuning vs. few-shot vs. zero-shot** → ch05
- **Grounding** → ch07
- **Knowledge base (RAG app pattern)** → ch11
- **LangChain chains (LCEL)** → ch06
- **MCP (Model Context Protocol)** → ch12
- **Memory (conversation)** → ch06, ch10
- **Middleware / security pipeline** → ch09
- **Mocking LLM responses** → ch08
- **Multimodal / vision** → ch03
- **PII redaction** → ch09
- **Prompt types (basic/messages/system)** → ch05
- **Rate limiting** → ch08, ch09, ch10
- **RAG (retrieval-augmented generation)** → ch05, ch07, ch11
- **React server components (RSC)** → ch04
- **Redis (session/state store)** → ch09, ch10, ch11
- **Streaming** → ch03, ch04
- **Structured data generation (Zod)** → ch04
- **Summarization (MapReduce/stuffing/refine)** → ch07
- **Testing (Vercel AI SDK / LangChain)** → ch08
- **Text splitters** → ch06
- **Tool/function calling** → ch04, ch06, ch12
- **Vector stores** → ch06, ch07, ch11
- **Vercel AI SDK core (`generateText`/`streamText`)** → ch03

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond this book, check related skills
or ask the agent directly.
