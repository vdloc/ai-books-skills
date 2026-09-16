# Cheatsheet

## Is it RAG or an Agent?
| Signal | RAG | Agent |
|---|---|---|
| User wants... | an answer grounded in data | something *done* (book, update, send) |
| Control flow | fixed one-shot retrieve→generate | plan → act → observe → re-plan loop |
| If you build RAG when they wanted action | disappoints — closes no loop | — |

## AWS Agentic Stack — Which Tier?
| Tier | Effort/control | Pick when |
|---|---|---|
| Amazon Q | Zero-config | Out-of-box assistant suffices, no custom logic needed |
| Bedrock + AgentCore | Moderate (this book's focus) | Need custom agent logic + managed infra |
| SageMaker AI (DIY) | Full control | Need custom model training/fine-tuning as the "brain" |

Anti-pattern: picking SageMaker-from-scratch when managed Bedrock would suffice adds needless infra burden; picking Amazon Q when you need code control under-delivers.

## Maturity Ladder (design checklist — don't overbuild)
LLM → LLM+RAG → Augmented LLM (tools+memory) → Agentic workflow (plan/execute/synthesize) → Autonomous agent (persistent, self-correcting) → Multi-agent collaboration.
Rule: build only as far up the ladder as the problem requires. Don't reach for multi-agent when a single Augmented LLM with tools solves it.

## MCP vs A2A
| | MCP | A2A |
|---|---|---|
| Scope | Agent ↔ tool/data | Agent ↔ agent |
| Analogy | "USB-C port for tools" | "phone number for other agents" |
| Use when | Agent needs external tools/data, framework-agnostic | Orchestrator delegates to specialist agents (possibly different teams/vendors) without knowing internals |
| Complementary? | Yes — not competing standards | Yes |

## MCP Resource vs Tool
| | Resource | Tool |
|---|---|---|
| Purpose | Read static/semi-static context | Perform an action |
| Side effects | None | Yes |
| Example | Browse a course directory | Send a message |
| Combined use | Read Resource to find *who*, then call Tool to act | |

## A2A Communication Mode Selection
| Mode | Use when |
|---|---|
| Synchronous (`SendMessage`) | Quick task, blocking OK |
| Asynchronous/polling (`GetTask`) | Long-running job, client will poll |
| Streaming (SSE) | Need incremental progress; requires `capabilities.streaming=true` |
| Push notification (webhook) | Long-running/event-driven, client can't stay connected |

## Multi-Agent Pattern Selection
| Pattern | Use when | Watch out for |
|---|---|---|
| Supervisor-Worker | Centralized coordination, easy debugging (default choice) | Supervisor = bottleneck/SPOF; workers can't collaborate directly |
| Agent-as-Tool | Reusable specialist called by multiple parents | Adds indirection — worth it past 1 caller |
| Swarm | Path unpredictable, agents must collaborate peer-to-peer | No fixed sequence → less auditable; watch for ping-pong loops (use `repetitive_handoff_*` params) |
| Graph (DAG/cyclic) | Compliance/auditability — must prove order of checks | Requires path known upfront |

## Memory: Which Optimization Strategy?
| App shape | Strategy |
|---|---|
| Simple, short chats | Sliding window |
| Fact-heavy | Retrieval-based (RAG) |
| Complex/long-term | Hierarchical or graph-based |
| Production | Hybrid (sliding window for flow + retrieval for long-term facts) |

## AgentCore Namespace: Pick the Narrowest Scope
Global (`/`) → compliance rules for every agent. Strategy (`/strategy/{id}`) → domain expertise shared across users. Actor (`/actor/{id}`) → per-user preferences. Session (`/session/{id}`) → per-conversation summaries.
Rule: narrowest namespace that still correctly scopes the data.

## Deployment Target Selection
| Target | Best for | Ceiling |
|---|---|---|
| Lambda | Short-lived, stateless, unpredictable traffic | 15 min exec, 2-5s cold start |
| ECS (Fargate) | Multi-turn, warm state, persistent connections | None (always-on) |
| AgentCore Runtime | Multi-turn, per-user isolation, managed auth/policy/observability | 15 min sync / 60 min streaming / 8h async |

## Evaluation Level → What It Diagnoses
| Level | Examples | Diagnoses |
|---|---|---|
| Session | GoalSuccessRate | Did the whole conversation succeed? |
| Trace | Correctness, Helpfulness, Faithfulness | Was this one turn good end-to-end? |
| Span/Tool | Tool/parameter selection accuracy | Exactly where did it go wrong? |

## Built-in vs Custom Evaluator
Use built-in for general quality (helpfulness, relevance, clarity) on common scenarios. Use custom whenever a domain-specific rule exists that a generic evaluator can't know (e.g. "any out-of-scope answer scores 0.0") — built-in metrics structurally cannot see business rules.

## Observability: 4 Layers
1. Infrastructure metrics (CPU/memory) — is compute healthy?
2. Request-level telemetry (latency/errors) — is something slow/broken?
3. Agent execution traces — *why* (reasoning, tool calls, tokens) — the agent-specific layer
4. Behavioral analytics (patterns across traces) — is quality silently degrading?
Tell: a 200 OK with correct output can still hide 3x redundant tool calls burning latency/cost — only Layer 3 traces reveal it.

## Guardrails vs Policy — Both Required
| | Bedrock Guardrails | AgentCore Policy (Cedar) |
|---|---|---|
| Controls | Model text output | Tool call execution |
| Enforcement | Content filtering | Deterministic rule evaluation |
| Bypassable by prompting? | Difficult but possible | No — evaluated outside the model |
Rule: never rely on a system-prompt instruction ("don't issue refunds over $1,000") as a compliance control — use Cedar policy for anything with real consequences.

## Production-Readiness Gate (checklist)
Before deploying, verify: error handling beyond `try/except: pass`, request-level observability, graceful degradation on model-provider outage, resource limits, someone else has run the code, rollback capability, cost estimate exists.
Tell: most "worked in dev, broke in prod" incidents trace back to skipping one of these.

## Tool Design Defaults
- Single-purpose tools, not mega-tools with modes.
- Class-based + shared `__init__` when tools share an expensive resource (DB connection) — avoids connection exhaustion under concurrency.
- `async def` only for slow I/O called multiple times with independent inputs; skip for sub-100ms/pure-calc tools.
