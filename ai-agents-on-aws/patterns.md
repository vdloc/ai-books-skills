# Patterns

## Single-Purpose Tool Design
**When to use**: Always, when building any `@tool`-decorated function for an agent.
**How**: Build narrow, focused tools (`get_sales_data`, `analyze_sales`, `send_email`) instead of one mega-tool with modes/flags. Type-hint every parameter and return value; write a docstring `Args:` section — the agent uses name + params + docstring to decide when/how to call it. Let the agent compose tools into a sequence itself; don't hand-write the `if X then steps 1,2,3` orchestration logic.
**Trade-offs**: More tools to define upfront, but each is reusable across agents and easier for the LLM to select correctly than a broad multi-mode tool.

## Class-Based Tools for Shared Resources
**When to use**: Independent `@tool` functions each open their own expensive resource (DB connection, API client) and concurrent load risks exhausting connection limits (MySQL default 151, PostgreSQL default 100).
**How**: Group related `@tool`-decorated methods inside a class with a shared `__init__` that opens one connection/client, reused by every method.
**Trade-offs**: Slightly more structure upfront; avoids "too many connections" production incidents under concurrent multi-user load.

## Async Tools for Parallel I/O
**When to use**: An agent must call the same/similar slow I/O-bound tool multiple times with independent inputs (e.g. checking 3 warehouses).
**How**: Mark the tool `async def`, `await` the I/O call, invoke the agent via `agent.invoke_async(...)` so Strands runs independent async tool calls concurrently.
**Trade-offs**: Sequential calls (2s×3 + 2s combine = 8s) become concurrent (2s + 2s combine = 4s) — but adds complexity with no measurable gain for tools under ~100ms or pure calculation; skip async there.

## CoALA Memory Typing
**When to use**: Deciding what kind of memory a given agent capability actually needs.
**How**: Split long-term memory into Episodic (personal history — what happened/why/outcome), Semantic (structured facts/definitions/rules — what the agent knows), Procedural (skills/workflows that improve with experience — how it gets things done). Combine with short-term/working memory for the active session.
**Trade-offs**: None — this is a classification tool, not an implementation; use it to choose the right AgentCore namespace/strategy, not as a runtime mechanism itself.

## Context Management (4 techniques)
**When to use**: An agent's context window is filling with history and risking "context rot" (degraded reasoning from an overloaded, low-signal context).
**How**: Sliding window (drop oldest messages — for short fast chats), Compaction (summarize + reset — for long-horizon coherence), Structured note-taking (persistent external file like NOTES.md — for complex multi-hour tasks), Sub-agent delegation (offload heavy work to a sub-agent, keep only its summary — when a single context can't hold the whole problem).
**Trade-offs**: Sliding window is cheapest but forgets early details; compaction preserves gist but loses minor detail; production systems typically hybridize (e.g. sliding window for flow + retrieval for long-term facts).

## AgentCore Namespace Scoping
**When to use**: Structuring where a piece of agent memory lives so it's visible to exactly the right scope.
**How**: Pick the narrowest namespace that correctly scopes the information — Global (`/`, universal rules for every agent), Strategy (`/strategy/{strategyId}`, domain expertise shared across users), Actor (`/actor/{actorId}`, per-user preferences), Session (`/session/{sessionId}`, per-conversation summaries).
**Trade-offs**: Too broad a namespace leaks information across users/sessions; too narrow duplicates data that should be shared. Default to Actor for personal preferences, Session for one conversation's context.

## Supervisor-Worker Multi-Agent
**When to use**: The most common production multi-agent pattern — centralized coordination, easy debugging (isolate by worker), easy to extend (add a worker, tell the supervisor, done).
**How**: One supervisor agent receives the request, delegates to specialist workers via `worker.as_tool()`, and synthesizes their independent outputs. Workers never talk to each other — all communication routes through the supervisor.
**Trade-offs**: Supervisor is a bottleneck/single point of failure (mitigate with horizontal scaling, caching, redundant supervisors + failover); workers can't collaborate directly — use Swarm if they need to.

## Agent-as-Tool
**When to use**: Building a reusable specialist agent (e.g. `math_tutor`) that multiple parent agents need to call without hardcoded routing logic.
**How**: Wrap the agent with a `description` so other agents can discover and call it exactly like a regular `@tool` function. Composable — specialists can themselves use other agents as tools, creating layered expertise.
**Trade-offs**: Adds an indirection layer; worth it once more than one caller needs the same specialist behavior.

## Swarm (Peer Handoff)
**When to use**: Collaborative work where the path can't be predicted upfront (e.g. iterative game-level design where issues surface unpredictably across disciplines).
**How**: `Swarm([agents], entry_point=..., max_handoffs=N, max_iterations=N, repetitive_handoff_detection_window=W, repetitive_handoff_min_unique_agents=M)`. Agents hand off via `handoff_to_agent`, carrying full context forward. The last two params detect and break "ping-pong" loops.
**Trade-offs**: No fixed sequence means less predictability/auditability than Graph or Supervisor-Worker — only use when the path genuinely can't be predetermined.

## Graph (DAG / Cyclic Workflow)
**When to use**: Established procedures needing compliance/auditability — order processing, approval chains, content moderation — anywhere you must prove certain checks always happen in a specific order.
**How**: `GraphBuilder()`, `add_node(agent, name)`, `add_edge(from, to, condition=fn)` where `condition` inspects `state.results` from the prior node, `set_entry_point(name)`, `build()`. DAGs (no loops) suit most production workflows; cyclic graphs suit iterative review loops (e.g. developer↔reviewer until approval).
**Trade-offs**: Requires the path to be known upfront — use Swarm instead when it can't be.

## MCP Integration
**When to use**: An agent needs to call external tools/read external data/use reusable prompt templates without a custom integration per tool.
**How**: Host (the AI app) → MCP Client (1:1 with a server) → MCP Server (exposes Tools/Resources/Prompts) → Data sources. Use a Resource when the agent just needs to *read* static/semi-static context; use a Tool when an *action* needs to happen. Choose transport: Stdio for local, Streamable HTTP for remote/cloud with live updates.
**Trade-offs**: Collapses N×M per-tool integration cost to N+M (every model and every tool connects to one shared protocol layer) — the standardization cost is worth it past a handful of tools/models.

## A2A Cross-Agent Delegation
**When to use**: Cross-framework, cross-organization agent collaboration where agents must stay opaque (no shared internals) — e.g. a Diagnosis Agent and an Insurance Agent from different systems negotiating a claim.
**How**: Client Agent discovers the Remote Agent via its Agent Card (`/.well-known/agent-card.json`), then communicates via one of 4 modes: Synchronous (`SendMessage`, blocking, quick tasks), Asynchronous/polling (`GetTask`, long jobs), Streaming (`SendStreamingMessage` over SSE), Push notifications (webhook on completion).
**Trade-offs**: Don't collapse an agent into an MCP tool call to avoid building A2A — tools are for stateless predefined actions, agents solve open-ended iterative problems; wrapping loses the reasoning/delegation capability.

## AgentCore Gateway Auto-Tooling
**When to use**: Enterprises with hundreds of existing REST APIs that would otherwise each need a hand-built MCP wrapper.
**How**: Point Gateway at an OpenAPI spec / Smithy model / Lambda function / remote MCP server (a "target") via `create_gateway_target(...)` — it auto-generates one MCP tool per endpoint, no wrapper code. Past ~100+ tools, use `x_amz_bedrock_agentcore_search` for semantic tool search by embedding similarity instead of loading every tool definition into the prompt.
**Trade-offs**: None significant — this removes integration work that would otherwise be manual per API.

## Deployment Target Selection (Lambda / ECS / AgentCore Runtime)
**When to use**: Choosing where to run a production agent.
**How**: Lambda for short-lived, stateless, unpredictable-traffic tasks (scales to zero, 15-min ceiling, 2-5s cold starts). ECS for multi-turn conversations needing warm state, persistent connections, or in-process multi-agent calls (Fargate first, EC2 only if you need GPU/instance control). AgentCore Runtime for multi-turn, per-user-isolated workloads needing managed auth/policy/observability — each session gets a dedicated microVM that sanitizes memory on termination.
**Trade-offs**: Lambda is cheapest when idle but can't hold warm state across invocations without external storage (e.g. DynamoDB); ECS costs more at idle but avoids cold starts and supports long-lived state; AgentCore Runtime adds the most built-in isolation/governance at the cost of AWS lock-in.

## Domain-Scoped Custom Evaluation
**When to use**: A generic/built-in evaluator (Correctness, Helpfulness) would score a domain-rule violation as acceptable — e.g. a travel agent politely refusing an off-topic Python-scripting question scores "OK" on generic Correctness but should score "Very Poor" for violating domain scope.
**How**: Write a custom LLM-as-judge rubric or code-based/deterministic evaluator encoding the specific business rule, and run it alongside (not instead of) built-in evaluators at the appropriate level (session/trace/span).
**Trade-offs**: More evaluators to maintain, but built-in metrics structurally cannot see domain-specific rules — skipping custom evaluation lets exactly these violations pass silently.

## Policy-as-Code Authorization (Cedar)
**When to use**: Any rule with real-world consequences that must hold 100% of the time (refund limits, cross-tenant data isolation, role-based record access) — never rely on a system-prompt instruction for these.
**How**: Write declarative `forbid`/`unless` Cedar rules evaluated deterministically against every tool call, outside the model's reasoning loop — "no amount of prompt engineering overrides it." Check *authenticated* identity attributes (`principal.orgId`), never values a prompt-injected conversation could have influenced.
**Trade-offs**: Requires defining rules explicitly upfront; the alternative (trusting the system prompt) is not actually a trade-off — the book states plainly you cannot build a compliance program on "the model usually follows the system prompt."
