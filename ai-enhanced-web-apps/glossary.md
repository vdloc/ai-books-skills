**AI state vs. UI state** — AI state is the source-of-truth conversation history sent to the model; UI state is what's rendered; RSC-based apps separate these explicitly (Ch4).

**`asRetriever()`** — LangChain method wrapping a vector store as a composable retriever chain component (Ch6, Ch7).

**Async iterable / async generator** — JS primitive (`async function*` + `yield`, consumed via `for await...of`) underlying streaming text (Ch3).

**Chain-of-thought (CoT) prompting** — instructing a model to explain its reasoning step-by-step; improves transparency, not guaranteed correctness (Ch5).

**`ChatPromptTemplate` / `FewShotPromptTemplate`** — LangChain factory-method classes encapsulating prompt construction with placeholders and few-shot example lists (Ch6).

**Compound message** — a LangChain/Vercel message with attachments (e.g., an image) beyond plain text (Ch5).

**Cosine similarity / Euclidean distance** — two standard vector-similarity metrics for comparing embeddings (Ch5).

**Embedding** — a numerical vector representation of text capturing semantic meaning, used for similarity search/RAG (Ch5).

**Feature extraction** — ML technique of identifying and extracting meaningful patterns from raw data (Ch1).

**Few-shot learning** — providing a model with a small number of representative input/output examples to guide its output format/tone without fine-tuning (Ch5, Ch6).

**File-based routing (Next.js)** — folder structure under `src/app` determines URL routes; `page.js` per folder = public route; `[slug]` = dynamic segment (Ch2).

**Fine-tuning** — retraining a pretrained model's weights on a task-specific dataset; most accurate, most resource-intensive learning technique (Ch5).

**Generative AI web app component stack** — User → UI/conversational components → Backend infrastructure ↔ LLMs/AI models (Ch1).

**Grounding** — verifying a generated response's factual accuracy against a reliable corpus to reduce hallucination (Ch7).

**Hallucination** — AI generating false/nonexistent content presented as fact (Ch1).

**HNSWLib** — a filesystem-persistent LangChain vector store, used for production RAG indexing (Ch7).

**HyDE (hypothetical document embedding)** — generating synthetic document text to improve retrieval matching (Ch7).

**Knowledge base (app concept)** — a user-created container grouping uploaded documents that a RAG chat session is scoped to (Ch11).

**Language model specification (abstract factory pattern)** — Vercel AI SDK's provider-agnostic interface (`generateText`/`streamText`) where each provider package is a concrete factory (Ch3, Ch8).

**LangChain agent (ReAct)** — an LLM-driven reasoning loop that dynamically selects and calls tools until a final response is ready (Ch6).

**LLM-as-a-judge** — using an LLM to evaluate another LLM's output quality, as an alternative to costly human evaluation (Ch5).

**MapReduce summarization** — split document → summarize chunks independently (map) → combine into final summary (reduce) (Ch7).

**MCP (Model Context Protocol)** — a standardized client-server protocol for AI apps to discover/invoke external tools/data sources, portable across frameworks (Ch12).

**MCP gateway** — a single AI-aware proxy centralizing routing/auth/context across many MCP servers (Ch12).

**Message quota** — a per-user daily cap on AI messages, enforced via Redis, distinct from rate limiting (Ch9, Ch10).

**`MockLanguageModelV1`** — Vercel AI SDK's official test double for `generateText`/`generateObject`, overriding `doGenerate` for deterministic tests (Ch8).

**MultiHop-RAG** — a RAG variant recursively retrieving additional context based on initial results (Ch7).

**Persona** — the defined style/behavioral traits an AI app exhibits (Ch1, Ch2).

**PII redaction/anonymization** — replacing sensitive user data with placeholders before it reaches an LLM or gets logged (Ch9).

**RAG (retrieval-augmented generation)** — enhancing LLM responses by retrieving relevant external context before generation; four components: indexing, retrieval, augmentation, generation (Ch5, Ch7, Ch11).

**RAG architecture (offline/online split)** — offline document indexing vs. online query embed → retrieve → augment → generate (Ch7).

**ReAct (Reasoning + Acting)** — a prompting framework where a model alternates between reasoning and taking actions (tool calls) (Ch6).

**`RecursiveCharacterTextSplitter`** — LangChain's recommended text splitter; splits on cascading separators with configurable chunk size/overlap (Ch6).

**Refine summarization** — sequentially updating a running summary with each new document chunk; coherent but error-propagating (Ch7).

**Route group (Next.js)** — a folder wrapped in parentheses, e.g. `(chat)`, organizing files without adding a URL segment (Ch2).

**RSC (React Server Component)** — components run exclusively server-side, streamed to the client as rendered nodes rather than JSON-then-render (Ch4).

**Runnable / LCEL** — LangChain's composable unit-of-work abstraction (invoke/batch/stream), chained via `.pipe()` or `RunnableSequence` (Ch6).

**Security middleware layer** — a composable chain (CORS, rate limit, auth, security headers) intercepting all requests before core app logic (Ch9).

**Sliding window rate limiter** — caps requests per identifier (IP/user) within a rolling time window, typically Redis-backed (Ch9).

**Standalone question transformation** — rewriting an ambiguous, context-dependent user query into a self-contained one before retrieval (Ch6).

**Streamable UI (`streamUI`/`createStreamableUI`)** — Vercel AI SDK functions letting the server stream whole rendered React components, not just text (Ch4).

**Structured data generation (`generateObject`/`streamObject`)** — producing schema-validated (Zod) typed output from an LLM instead of free text (Ch4).

**Stuffing summarization** — passing an entire document into one prompt call; only works within the context window (Ch7).

**System prompt** — a standing instruction/persona set via the `system` attribute for an entire session (Ch5).

**Text splitter** — transforms long documents into context-window-sized chunks (Recursive, HTML, Markdown, Code, Token, Character types) (Ch6).

**Threat model** — the process of identifying public endpoints, user input points, and data sensitivity before designing input validation (Ch9).

**Tool/function calling** — a model requesting a registered external function's execution, with results fed back into its response (Ch4, Ch6).

**Tree of thoughts** — extends chain-of-thought to explore multiple branching reasoning paths rather than one linear path (Ch5).

**`useChat` / `useCompletion`** — Vercel AI SDK React hooks managing streaming multi-turn conversation state / single-prompt completion respectively (Ch3).

**Vector store** — a specialized database for high-dimensional embedding vectors enabling similarity search (Ch6, Ch7, Ch11).

**Zero-shot learning** — a model performing a task with no task-specific examples, relying purely on pretrained knowledge (Ch5).

**Zod schema** — TypeScript-first validation library used as the Vercel AI SDK's default structured-output validator (Ch4, Ch9).
