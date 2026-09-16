# Cheatsheet: Build AI-Enhanced Web Apps

## Decision Rules

- **Choosing a prompt type**: single throwaway query → `prompt`. Anything with history, attachments, or tool calls → `messages`. Standing persona/behavior for the whole session → `system`. Default to `messages` — it's strictly more flexible.
- **`useChat` vs `useCompletion`**: conversation history matters → `useChat`. Single prompt-in/completion-out → `useCompletion`.
- **When to reach for LangChain agents over Vercel AI SDK tool calling**: task needs dynamic, iterative, multi-tool reasoning the model itself decides → LangChain ReAct agent. Task needs one fixed, developer-known tool call → plain Vercel `streamUI`/tool calling (simpler, less overhead).
- **Structured output**: consuming AI output programmatically (tables, forms, other services) → `generateObject`/`streamObject` + Zod schema. Never hand-roll regex parsing when this is available.
- **Summarization strategy**: doc fits context window → stuffing. Long doc, parallelizable, chunk independence OK → MapReduce. Long doc, cross-chunk coherence needed → refine (accept the latency/error-propagation cost).
- **Retry vs. fallback**: error is transient (429, temporary 5xx) → retry (`maxRetries`). Error is persistent (quota exhaustion, provider down) → model fallback (`ai-fallback`), not retry.
- **What to mock in tests**: input sanitization, response formatting, error handling, flow control → mock it. Sentiment/reasoning quality, model reliability, performance characteristics → do NOT mock; needs real model calls.
- **Rate limit vs. quota**: burst abuse protection → sliding-window rate limiter (per IP/user, short window). Total cost control → daily message quota (per user, Redis-backed) — always check quota *after* rate limiting, never before.
- **Vector store isolation**: default → shared store + strict namespacing/metadata filtering (cost-efficient). Strict regulatory/compliance requirement → dedicated store per tenant (physical isolation).
- **API route vs. server action**: need to stream binary/non-JSON data (e.g., audio) or want a stable REST-style contract → API route. Simple form-submission-triggered generation tied to a React component → server action.

## Decision Tree: Adding an AI Capability to a Web App

1. Does the response need to be a rendered component, not text? → `streamUI`/`createStreamableUI` (Ch4).
2. Does the model need current/external data? →
   - One-off, developer-known tool → Vercel AI SDK tool calling (Ch4).
   - Dynamic, multi-tool, model-decided → LangChain ReAct agent (Ch6) or MCP server (Ch12) if cross-framework reuse matters.
3. Does the answer need to be grounded in a document corpus? → RAG chain (Ch7): index → retrieve → augment → generate; add grounding verification if hallucination risk is high.
4. Does the output need to be consumed by other code? → `generateObject`/`streamObject` + Zod (Ch4).
5. Is this multi-tenant (multiple users' data)? → namespace/filter every vector-store operation by tenant ID (Ch11).

## Trade-off Matrix: Summarization Methods

| Method | Speed | Coherence | Max doc size | Error propagation risk |
|---|---|---|---|---|
| Stuffing | Fastest | N/A (single call) | Must fit context window | None (single pass) |
| MapReduce | Fast (parallel) | Lower (independent chunks) | Unlimited | Isolated per chunk |
| Refine | Slowest (sequential) | Highest | Unlimited | High (compounds) |

## Trade-off Matrix: Structured Output Techniques

| Technique | Reliability | Cost | Use when |
|---|---|---|---|
| Prompt engineering alone | Low | Low | Never sufficient alone |
| Output parsing (regex) | Low, fragile | Low | Avoid if alternatives exist |
| Provider function calling | Medium-high | Low | Provider supports it natively |
| Zod schema validation (`generateObject`) | High (auto-retry) | Low-medium | Default choice |
| Iterative reprompting | High | High (extra tokens) | Other methods failed |
| Postprocessing | Safety net | Low | Always, as a final pass |

## Thresholds & Defaults (from the book's examples)

- Chat response token cap: `maxTokens: 100–150` for short conversational replies.
- Rolling conversation context: keep last **10** messages.
- Retry attempts: `maxRetries: 3` for transient errors.
- Rate limit example: **5 requests / 10 seconds** (sliding window, per IP).
- Daily message quota example: **10 messages/user/day**.
- RAG retriever: top **k = 6** chunks per query (production example); `k = 1` for narrow single-fact lookup.
- Vector index dimensionality: must match the embedding model's output (e.g., **768** for Google AI embeddings).
- Text chunking: `chunkSize: 100`, `chunkOverlap: 20` as an illustrative starting point — tune per document type/model context window.
- Max file upload size for images (OpenAI vision): **20 MB**.

## Tells & Smells

- **"Missing value for input {X}" chain error** → argument mismatch between two piped Runnables; check the previous step's output shape against the next step's expected input.
- **Retrieval returns irrelevant chunks** → check embedding model mismatch between index-time and query-time, or `k` set too low/high, or missing standalone-question rewrite for ambiguous queries.
- **Redacted input in logs but LLM response implies it saw PII** → bug is in UI display/response handling, not the anonymization function itself — check each pipeline stage independently.
- **Rate limiter passes but Redis quota counter keeps climbing on rejected requests** → quota check placed before rate limiting in the pipeline; reorder so quota check comes after.
- **Tests pass against mocks but production behavior differs** → likely mocking something that needed real model behavior (reasoning/sentiment/quality) rather than app logic.
- **Agent hits context-window errors as more tools are added** → too many registered tools; each tool's name/description/params consumes prompt tokens — trim the toolset.
- **A skill/tool built for one framework won't work in another** → framework-specific tool-calling (LangChain/Vercel-native); consider MCP if genuine cross-framework reuse is needed.
- **Chunk summaries lose narrative thread across a long document** → used MapReduce where refine's sequential coherence was actually needed.
