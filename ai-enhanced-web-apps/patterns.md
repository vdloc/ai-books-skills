## Incremental SDK Integration
**When to use**: introducing any new framework/SDK into an existing production app (e.g., Vercel AI SDK, LangChain).
**How**: replace one API call at a time (route handler → streaming → multi-provider → multimodal), verifying the frontend contract stays stable after each step, before moving to the next capability.
**Trade-offs**: slower to reach full capability, but each step is independently testable and rollback-safe.

## Provider-Agnostic Model Calls (Abstract Factory)
**When to use**: any app that may need to swap or add LLM providers.
**How**: call `generateText`/`streamText` with a `model` argument from a provider package (`@ai-sdk/openai`, `@ai-sdk/google`, etc.) rather than calling a provider's raw client SDK directly.
**Trade-offs**: adds one layer of abstraction, but swapping providers becomes a one-line change instead of a rewrite.

## Custom Hook Construction (5 steps)
**When to use**: chat/form logic starting to clutter a component.
**How**: (1) name the hook `use...`, (2) decide what state stays internal vs. exposed, (3) inject helper functions as dependencies, (4) implement core logic, (5) return an object of state + handlers.
**Trade-offs**: more indirection for trivial state, but pays off once logic needs reuse or testing in isolation.

## RSC Generative UI (`streamUI`)
**When to use**: the AI response itself should be a rendered component (card, table), not text the client re-interprets.
**How**: `streamUI({ model, messages, text, tools })` — `text` renders plain responses, `tools` triggers richer component generation via an async-generator `generate` function that can `yield` interim states.
**Trade-offs**: tighter coupling between AI output and UI components; more powerful UX at the cost of more server-side rendering logic.

## Structured Output via Schema Validation
**When to use**: AI output your code will consume programmatically (tables, forms, other services).
**How**: `generateObject`/`streamObject` with a Zod schema — SDK validates and auto-retries on failure, rather than hand-parsing free text with regex.
**Trade-offs**: requires defining schemas up front; far more reliable than string parsing.

## Tool-Calling Loop (Model-Requested Side Effects)
**When to use**: the model needs current/external data (weather, DB lookups, APIs) it wasn't trained on.
**How**: register a Zod-typed `parameters` schema + handler function per tool; the SDK's tool manager executes it and feeds the result back to the model — execution stays under app control.
**Trade-offs**: context-window cost per registered tool; keep the toolset small and well-described.

## Few-Shot Prompting
**When to use**: adapting model tone/format/domain without fine-tuning.
**How**: set context → list 2-3 diverse representative example use cases with example interactions → close with an explicit directive ("Now respond to the following...").
**Trade-offs**: cheap and fast, but overly narrow examples cause the model to overfit to that narrow pattern.

## Chain-of-Thought Prompting
**When to use**: multi-step reasoning tasks (math, logic, planning) needing transparency/auditability.
**How**: explicitly instruct "solve step-by-step"; provide one worked example; test with varied inputs and refine wording.
**Trade-offs**: improves transparency and often accuracy, but never guarantees correct intermediate steps — validate for precision-critical tasks.

## Embeddings-Based Retrieval (Semantic Search)
**When to use**: FAQ/knowledge-base lookup, or as the retrieval half of RAG.
**How**: embed content offline, store in a vector DB; embed the query with the *same* model at query time; rank via cosine similarity/Euclidean distance; return top-k.
**Trade-offs**: requires consistent embedding model between index-time and query-time or comparisons become meaningless.

## LCEL Runnable Chain Composition
**When to use**: a workflow needs more than one processing step around the LLM call.
**How**: wrap functions as `RunnableLambda`, compose via `.pipe()` or `RunnableSequence.from([...])` — transform → transform → prompt → model → output parser.
**Trade-offs**: each step's output must match the next step's expected input shape; mismatches surface as opaque template errors.

## RAG Chain (Retrieval + Augmentation + Generation)
**When to use**: grounding LLM answers in a specific/current/private document corpus.
**How**: split docs → embed → vector store → `asRetriever()` → `RunnableSequence` merging `{ context: retriever.pipe(formatDocs), question: RunnablePassthrough() }` → prompt → model → parser; always return `sourceDocuments` for transparency.
**Trade-offs**: quality hinges on chunking strategy and `k` (retrieved doc count); consider rewriting ambiguous queries to standalone form first.

## Document Summarization Strategy Selection
**When to use**: summarizing documents that may exceed the model's context window.
**How**: use stuffing for short docs (fits context); MapReduce for long docs needing parallelism; refine for long docs needing sequential coherence.
**Trade-offs**: stuffing fails on long docs; MapReduce loses cross-chunk coherence; refine is slower and propagates early errors.

## ReAct Agent (Dynamic Multi-Tool Reasoning)
**When to use**: multistep tasks requiring iterative reasoning across many tools chosen dynamically by the model (vs. a single fixed tool call).
**How**: `createReactAgent({ llm, tools, prompt })`; the agent cycles decision → tool call → observe until it satisfies end-goal criteria.
**Trade-offs**: more context-token overhead per registered tool; more latency from propagation through the reasoning loop.

## Layered Error/Rate-Limit Defense
**When to use**: any production app calling a rate-limited external LLM provider.
**How**: proactive (`maxTokens`, rolling context window) + reactive (`maxRetries` for transient 429/5xx, `ai-fallback` model fallback for persistent outages) + user-facing error classification (`AIErrorTracker`).
**Trade-offs**: more moving parts to configure and monitor, but prevents silent failures and wasted retry cost.

## Mock-Only-What-You-Own Testing
**When to use**: writing tests for any LLM-integrated feature.
**How**: mock simple/predictable logic (sanitization, formatting, error handling) with minimal response shapes (`{ text, isComplete }`); never mock evolving model capability (reasoning quality, sentiment accuracy) — use `MockLanguageModelV1`/`simulateReadableStream` for SDK-level mocks.
**Trade-offs**: some tests still require real (costly, nondeterministic) API calls to validate actual model behavior.

## Security Middleware Pipeline
**When to use**: any Next.js app needing more than one cross-cutting security concern.
**How**: `composeMiddleware([handleCORS, rateLimit, authenticate, securityHeaders])`, positioned at the very front of the request pipeline (edge, on Vercel), each step able to short-circuit or halt.
**Trade-offs**: ordering matters — cheap rejects (CORS, rate limit) should run before expensive ones (auth, LLM calls).

## Defense-in-Depth Abuse Control
**When to use**: any app exposing paid LLM calls to end users.
**How**: rate limiting (burst, sliding window) → message quota (daily cap per user, Redis-backed, checked *after* rate limiting) → registration friction (invite-only, CAPTCHA) if quota evasion via disposable accounts occurs.
**Trade-offs**: layered complexity, but a rate limiter alone doesn't stop determined abuse.

## PII Redaction at the Input Boundary
**When to use**: any user input that might contain sensitive data before it reaches an LLM or gets logged.
**How**: run `redact-pii` (or `@google-cloud/dlp` for production) on validated input before sending to the model or storing/logging it.
**Trade-offs**: adds a processing step; essential for compliance (GDPR/CCPA) and provider data-sharing risk.

## Redis-as-Session-Store (Lightweight State)
**When to use**: intermediate data needs (session lists, per-user sets) where a full relational DB is unnecessary.
**How**: design keys around access patterns — a Redis *set* for `user:sessions:{userId}`, a Redis *hash* per `session:{sessionId}` — fetch via `SMEMBERS` then parallel `HGETALL`.
**Trade-offs**: no native transactions/joins; migrate to SQL once data becomes genuinely relational.

## Namespaced Shared Vector Store (Multi-Tenant RAG)
**When to use**: multi-tenant RAG apps where per-user dedicated vector DBs are cost/ops-prohibitive.
**How**: one shared vector store with strict namespacing/metadata filtering (user ID + knowledge-base ID) enforced on every query and write.
**Trade-offs**: logical (not physical) isolation — insufficient for strict regulatory/compliance requirements demanding physical separation.

## MCP Tool Exposure (Cross-Framework Reuse)
**When to use**: a tool integration needs to be reusable across frameworks/assistants, not hardwired to one app's tool-calling code.
**How**: build an MCP server exposing named, schema-typed tools; any MCP client (regardless of framework) can consume them; use stdio transport for local dev, HTTP for remote/production.
**Trade-offs**: adds a protocol/process layer, but centralizes external API access as a security boundary and makes the tool portable.
