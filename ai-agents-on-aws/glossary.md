# Glossary

**A2A (Agent2Agent Protocol)** — standard for agent-to-agent discovery and communication; opaque agents interact via Agent Cards, Tasks, and Artifacts without sharing internals (Ch1, Ch5).

**Actor** — in AgentCore Memory, the identity (user or user-agent pair) an event belongs to; prevents cross-user memory contamination (Ch3).

**ADOT (AWS Distro for OpenTelemetry)** — AWS's distribution of OpenTelemetry; auto-instruments agent frameworks and exports OTel traces to CloudWatch/AgentCore Observability (Ch6, Ch7).

**Agent Card** — a JSON document (`/.well-known/agent-card.json`) describing an A2A agent's name, capabilities, skills, URL, and protocol version — the discovery mechanism (Ch1, Ch5).

**Agentic RAG** — RAG treated as one tool among many an agent can call, refine, or skip, vs. a fixed one-step retrieval pipeline (Ch1).

**Agent-as-Tool** — pattern wrapping an agent with a `description` so other agents can discover and call it like a regular tool (Ch4).

**AgentCore** — AWS's managed platform for production agents: Runtime, Gateway, Identity, Policy, Memory, Observability, Evaluations (Ch1, Ch6, Ch7).

**AgentCore Gateway** — converts OpenAPI specs, Lambda functions, or remote MCP servers into MCP-compatible tools automatically (Ch6).

**AgentCore Identity** — manages inbound (JWT) and outbound (OAuth token vault) authentication for agents (Ch6).

**AgentCore Policy** — Cedar-rule-based deterministic tool-call authorization enforced outside the model's reasoning loop (Ch6, Ch7).

**AgentCore Runtime** — per-user-session microVM hosting for any agent framework; sessions move Active → Idle → Terminated (Ch1, Ch6).

**Artifact** — an A2A Task's output container, made of one or more Parts (Ch5).

**Asynchronous communication (A2A)** — returns a Task ID immediately; client polls `GetTask` until completion (Ch5).

**Async tool** — a Strands `@tool async def` allowing concurrent execution of independent I/O-bound tool calls via `invoke_async` (Ch2).

**Behavioral analytics** — the 4th observability layer: patterns across many traces revealing slow degradation (Ch7).

**Built-in evaluator** — ready-to-use LLM-as-judge evaluator (GoalSuccessRate, Correctness, Helpfulness) for common quality checks (Ch7).

**Cedar** — AWS's open-source declarative policy language used by AgentCore Policy for deterministic tool-call rules (Ch6, Ch7).

**Checkpoint** — a snapshot of session state (AgentCore Memory / LangGraph) letting an interrupted conversation resume (Ch3).

**CoALA (Cognitive Architectures for Language Agents)** — framework organizing an agent around reasoning, action, and memory (short-term, episodic, semantic, procedural) (Ch3).

**Code-based evaluator** — deterministic, programmatic evaluator (exact match, regex, business rules) as opposed to LLM-as-judge (Ch7).

**Collaborative communication** — agents negotiate/discuss until they jointly resolve trade-offs with no single right answer (Ch4).

**Consensus voting** — running N independent parallel agents and voting on the result to cut error rates (Ch4).

**Consolidation** — merging newly extracted memory with existing knowledge in AgentCore Memory (Ch3).

**Context management** — deciding what stays in the model's context window vs. what's offloaded to storage; "context rot" is the failure mode of overloading it (Ch3).

**Custom evaluator** — LLM-as-judge or code-based evaluator with organization-specific criteria/rubric (Ch7).

**Elicitation (MCP)** — a server requesting missing information mid-session via form mode or URL mode (Ch5).

**Episodic memory** — CoALA long-term memory type recalling personal history/events (Ch3).

**Event** — a raw interaction record (message, tool call, response) written to short-term memory in AgentCore (Ch3).

**FastMCP** — Python framework purpose-built for defining MCP tools/resources/prompts with minimal boilerplate (Ch5).

**Function calling / tool use** — the general mechanism giving an LLM agent the ability to invoke external functions (Ch2).

**Graph pattern (DAG)** — multi-agent orchestration with explicit, code-defined nodes and edges; auditable, compliance-friendly (Ch4).

**Guardrails (Bedrock)** — content-filtering layer between agent and model controlling what the agent *says* (PII, denied topics, harmful content) (Ch7).

**LLM-as-a-judge** — a model scoring another agent's trace/output against a rubric, usable without ground truth (Ch3 mention, Ch7).

**Long-term memory** — persistent memory across sessions; CoALA splits it into episodic, semantic, and procedural (Ch3).

**MCP (Model Context Protocol)** — standard connecting agents/models to tools, resources, and prompt templates via Host/Client/Server/Data-source architecture (Ch1, Ch5).

**Memory extraction module** — AgentCore component converting raw short-term events into structured long-term memory records asynchronously (Ch3).

**Memory strategy** — configuration (built-in, built-in-override, self-managed) controlling how AgentCore Memory extracts and organizes long-term memories (Ch3).

**microVM** — AgentCore Runtime's per-session isolated execution environment; zero cross-session data leakage (Ch6).

**Multi-agent pattern** — structured way of organizing multiple agents (Supervisor-Worker, Agent-as-Tool, Swarm, Graph) toward a common goal (Ch4).

**Namespace (AgentCore Memory)** — 4-level partitioning (global/strategy/actor/session) scoping where memories are organized and retrieved (Ch3).

**OpenTelemetry (OTel)** — open standard for traces/metrics/logs; the portability layer letting one instrumentation feed multiple observability backends (Ch7).

**Parallel communication** — independent agents work simultaneously; results combined afterward; total time = slowest agent (Ch4).

**Ping-pong behavior** — Swarm failure mode where two agents repeatedly hand off without progress; mitigated by repetitive-handoff detection (Ch4).

**Procedural memory** — CoALA long-term memory type encoding skills/workflows/action patterns (Ch3).

**Protocol binding (A2A)** — the transport (HTTP, gRPC, JSON-RPC) an agent supports, declared in its Agent Card (Ch5).

**Push notifications (A2A)** — server POSTs to a client-provided webhook on task completion; for long-running work where the client can't stay connected (Ch5).

**RAG (Retrieval Augmented Generation)** — Retrieve → Augment → Generate pipeline grounding LLM output in external data (Ch1).

**Resource (MCP)** — read-only, persistent data an MCP server exposes for browsing/referencing, no side effects (Ch5).

**Resource template (MCP)** — a parameterized URI pattern (e.g. `travel://activities/{city}`) for querying specific data instances (Ch5).

**Root (MCP)** — a filesystem boundary a client exposes to a server, scoping what the server may access (Ch5).

**Sampling (MCP)** — a server asking the client's model to run inference on the server's behalf, with user approval (Ch5).

**Semantic memory** — CoALA long-term memory type storing structured factual knowledge (facts, definitions, rules) (Ch3).

**Sequential communication** — one agent finishes and hands off to the next; used when step B needs step A's output (Ch4).

**Session (observability/A2A/memory)** — the top-level grouping of a user's full multi-turn interaction (Ch3, Ch5, Ch7).

**Short-term memory** — an agent's working/session-scoped memory; resets when the session ends unless persisted (Ch3).

**Span** — a single operation inside a trace (one model call, one tool call, one memory retrieval) (Ch7).

**Streaming (A2A)** — real-time incremental updates over SSE, requires `capabilities.streaming=true` in the Agent Card (Ch5).

**Supervisor-Worker pattern** — one supervisor agent delegates to specialist workers and synthesizes their outputs; the most common production multi-agent pattern (Ch4).

**Swarm pattern** — agents autonomously hand off control to each other via `handoff_to_agent`, no fixed sequence (Ch4).

**Synchronous communication (A2A)** — `SendMessage`, blocking request-response for quick tasks (Ch5).

**Target (AgentCore Gateway)** — a registered backend (API spec, Lambda, MCP server) that becomes one or more auto-generated MCP tools (Ch6).

**Task (A2A)** — tracks a unit of work the remote agent executes over time, producing Artifacts (Ch5).

**Tool (MCP/Strands)** — an executable action an agent can invoke; `@tool` decorator + type hints + docstring is the full contract in Strands (Ch2, Ch5).

**Trace** — one complete agent execution/invocation within a session, from request to final response (Ch7).
