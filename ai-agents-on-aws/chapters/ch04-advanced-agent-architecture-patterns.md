# Chapter 4: Advanced Agent Architecture Patterns

## Core Idea
Multi-agent systems exist to fix specific failure modes of a single agent (context overload, "jack of all trades," no parallelization, hard-to-debug) — but the pattern choice (Supervisor-Worker, Agent-as-Tool, Swarm, Graph) must match how much control vs. autonomy the problem needs, and a single well-tooled agent still beats a poorly designed multi-agent system.

## Frameworks Introduced
- **Supervisor-Worker pattern**: one supervisor agent receives the request, delegates to specialist workers (via `worker.as_tool()`), and synthesizes their independent outputs into one coherent answer. Workers never talk to each other — all communication routes through the supervisor.
  - When to use: the most common production pattern — centralized coordination, easy debugging (isolate by worker), easy to extend (add a worker, tell the supervisor, done).
  - Trade-off: supervisor is a bottleneck and single point of failure (mitigate with horizontal scaling, caching, redundant supervisors + failover); workers can't collaborate directly (use Swarm/collaborative communication if they need to).
- **Agent-as-Tool pattern**: wrap an agent with a `description` so any other agent can discover and call it exactly like a regular `@tool` function.
  - When to use: building reusable specialist agents (e.g. `math_tutor`) that multiple parent agents (`teacher`, `tutoring_assistant`, `homework_grader`) can call without hardcoded routing logic; composable — specialists can themselves use other agents as tools, creating layered expertise.
- **Swarm pattern**: agents autonomously hand off control to each other via a `handoff_to_agent` tool, carrying forward full context (original task, prior agents' contributions, available-agent descriptions) — no supervisor, no fixed sequence.
  - When to use: collaborative work where the path can't be predicted upfront (e.g. iterative game-level design where issues surface unpredictably across disciplines).
  - How: `Swarm([agents], entry_point=..., max_handoffs=N, max_iterations=N, repetitive_handoff_detection_window=W, repetitive_handoff_min_unique_agents=M)` — the last two params detect and break "ping-pong" loops (fewer than M unique agents appearing in the last W handoffs forces termination).
- **Graph pattern (DAG / cyclic graph)**: you define nodes (agents, swarms, nested graphs, or deterministic functions) and edges (with optional conditions) upfront; execution follows the explicit, auditable path the data takes.
  - When to use: established procedures needing compliance/auditability — order processing, approval chains, content moderation — anywhere you must prove certain checks always happen in a specific order. DAGs (no loops) suit most production workflows; cyclic graphs suit iterative review loops (e.g. developer↔reviewer until approval).
  - How: `GraphBuilder()`, `add_node(agent, name)`, `add_edge(from, to, condition=fn)` where `condition` inspects `state.results` from the prior node, `set_entry_point(name)`, `build()`.

## Key Concepts
- **Orchestration**: how information flows between agents (central coordinator vs. direct peer communication).
- **Specialization**: giving each agent a focused role/tools rather than one agent doing everything.
- **Sequential communication**: strict hand-off, one agent must finish before the next starts (use when step B needs step A's output; total time = sum of each step; single point of LLM-call concurrency helps with rate limits).
- **Parallel communication**: independent agents run simultaneously, results combined afterward (use when tasks don't depend on each other; total time = slowest agent, not the sum — also enables voting/consensus for higher accuracy).
- **Collaborative communication**: agents negotiate back-and-forth until they jointly resolve trade-offs with no single right answer (e.g. security vs. performance in a code review) — slower/costlier but catches errors a single pass would miss.
- **Consensus voting**: running N independent agents in parallel and voting on the result; cited research (*The Six Sigma Agent*) shows 5-agent consensus can cut error rates from 5% to 0.11% — but only if agents are genuinely independent (different models/data sources/approaches), or they'll share the same blind spots.
- **Ping-pong behavior**: a Swarm failure mode where two agents repeatedly hand off to each other without progress; detected and broken via the repetitive-handoff parameters.

## Mental Models
- Diagnose the need for multi-agent the way you'd diagnose a team's need to specialize: context overload, mediocre jack-of-all-trades output, no parallelization, and hard-to-isolate bugs are the four signals — absent these, stick with one agent.
- Match pattern to "who decides the path": Supervisor-Worker → the supervisor decides; Agent-as-Tool → the orchestrator picks from available tools; Swarm → the agents decide autonomously at runtime; Graph → you decide in code, upfront.
- Think of Swarm vs. Graph as improv jazz vs. a musical score: Swarm lets agents figure out the next step live (unpredictable but adaptive); Graph fixes the path in advance (predictable, auditable, provable to a compliance auditor).

## Anti-patterns
- **Rebuilding a working single-agent app into a 5-agent graph "because it's impressive in a demo"**: debugging becomes a nightmare (5 execution paths, unclear failure point), latency balloons (3s → 12s from coordination overhead), and error rate rises (5 failure points instead of 1). Only add agents when you hit an actual limitation — context window overflow, real parallelization need, or genuinely non-overlapping expertise.
- **Letting agents run unbounded handoffs in a Swarm**: without `max_handoffs`/`max_iterations` and repetitive-handoff detection, agents can loop indefinitely (ping-pong).
- **Expecting Graph-level auditability from Swarm, or Swarm-level adaptability from Graph**: Graph's fixed structure can't handle unpredictable collaborative discovery; Swarm's autonomous routing can't prove a fixed compliance sequence to an auditor.
- **Building "voting" consensus from agents that share the same model, data source, or reasoning approach**: they'll make the same mistakes together, defeating the point of consensus.

## Code Examples
```python
from strands import Agent

news_agent = Agent(name="news_agent", system_prompt="Research recent news...", tools=[news_search_tool])
financial_agent = Agent(name="financial_agent", system_prompt="Analyze financial metrics...", tools=[financial_api_tool])
sentiment_agent = Agent(name="sentiment_agent", system_prompt="Analyze market sentiment...", tools=[sentiment_analysis_tool])

supervisor = Agent(
    name="research_supervisor",
    system_prompt="""You are a market research coordinator. When asked about investment opportunities:
    1. Analyze the question to determine what information is needed
    2. Delegate to specialist agents: news_agent, financial_agent, sentiment_agent
    3. Synthesize their responses into a coherent investment recommendation
    Always provide balanced analysis considering all perspectives.""",
    tools=[news_agent.as_tool(), financial_agent.as_tool(), sentiment_agent.as_tool()]
)
response = supervisor("Should I invest in electric vehicle companies?")
```
- **What it demonstrates**: the Supervisor-Worker pattern — workers registered via `.as_tool()`, the supervisor's `system_prompt` encodes the delegation/synthesis logic, no manual routing code.

```python
from strands.multiagent import GraphBuilder

builder = GraphBuilder()
builder.add_node(risk_analyzer, "risk_analysis")
builder.add_node(manual_review_agent, "manual_review")
builder.add_node(fraud_detection_agent, "fraud_check")
builder.add_node(inventory_agent, "inventory_check")
builder.add_node(shipping_agent, "shipping")
builder.add_node(refund_agent, "refund")

builder.add_edge("risk_analysis", "manual_review", condition=is_high_value)
builder.add_edge("risk_analysis", "fraud_check", condition=is_suspicious)
builder.add_edge("risk_analysis", "inventory_check", condition=is_low_risk)
builder.add_edge("manual_review", "inventory_check")
builder.add_edge("fraud_check", "inventory_check")
builder.add_edge("inventory_check", "shipping", condition=is_in_stock)
builder.add_edge("inventory_check", "refund", condition=is_out_of_stock)
builder.set_entry_point("risk_analysis")
graph = builder.build()
result = graph("Order #12345: $1,500 laptop from new customer")
```
- **What it demonstrates**: a DAG with conditional edges — routing is explicit and auditable in code, while each node still uses an LLM agent for its own reasoning.

## Reference Tables

| Pattern | Who decides the path? | Best for | Trade-off |
|---|---|---|---|
| Supervisor-worker | Supervisor agent | Centralized coordination with a single synthesized answer | Supervisor is a bottleneck and single point of failure |
| Agent as tool | Orchestrator picks from available tools | Reusable agents across different systems | Communication only through orchestrator; nested calls add latency |
| Swarm | Agents decide autonomously | Collaborative work where the path can't be predicted | Harder to debug; execution order varies between runs |
| Graph (DAG) | You define the paths in code | Compliance workflows, auditable pipelines | All possible paths must be defined upfront |

## Worked Example
Investment-research assistant escalation, showing why Supervisor-Worker beats naive parallel fan-out:
1. **v1 (single agent)**: one agent does everything (news, financials, sentiment) — 5 minutes/query, fine at low volume, fails at "hundreds of queries per hour."
2. **v2 (naive parallel)**: split into `NewsAgent`, `FinancialAgent`, `SentimentAgent` running concurrently — 3x faster, but the investor gets 3 disconnected fact-dumps and has to synthesize a recommendation themselves. Speed improved, usefulness didn't.
3. **v3 (Supervisor-Worker)**: add a `research_supervisor` that calls all three workers in parallel *and* synthesizes their outputs into one coherent recommendation ("Consider established players like Tesla or Ford... if you prefer stability, wait for valuations to normalize"). This is the version that actually ships — same speed as v2, but the output is usable.
The lesson: parallelizing work isn't enough — someone (the supervisor) has to own turning multiple expert perspectives into one decision-ready answer.

## Key Takeaways
1. Diagnose the need for multiple agents via four signals: context overload, mediocre "jack of all trades" output, no parallelization, and hard-to-isolate bugs — absent these, one well-tooled agent wins.
2. Supervisor-Worker is the default production pattern: centralized synthesis, easy debugging, easy extension — accept the bottleneck/SPOF trade-off and mitigate with scaling/redundancy.
3. Use Agent-as-Tool to make specialist agents reusable across multiple parent systems without hardcoded routing logic.
4. Use Swarm when the collaborative path genuinely can't be predicted upfront; always set `max_handoffs`/`max_iterations` and repetitive-handoff detection to prevent infinite ping-pong.
5. Use Graph (DAG) when you need an explicit, auditable, compliance-provable path — the routing logic lives in code, not hidden in LLM reasoning.
6. Consensus voting across parallel agents can dramatically cut error rates, but only with genuinely independent agents (different models/data/approaches).
7. Most production systems combine patterns (a supervisor coordinating parallel workers, graph nodes that are themselves swarms) — match the pattern to the constraint, don't default to complexity.

## Connects To
- **Ch1**: extends the maturity ladder's "Multi-agent collaboration" stage and the MCP/A2A protocols mentioned there into concrete orchestration patterns.
- **Ch2/Ch3**: every worker/node in these patterns is itself a Ch2-style tool-using, Ch3-style memory-aware agent.
- **Ch5**: goes deeper into agent communication protocols (MCP, A2A) that underlie how these multi-agent systems actually pass messages.
- **Ch6/Ch7**: production deployment and evaluation of these multi-agent systems (observability across multiple agents, governance) are covered later.
