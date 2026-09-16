# Chapter 7: From RAG to AI Agents

## Core Idea
An AI agent extends RAG from a static, single-shot retrieve-then-generate flow into a dynamic, goal-driven reasoning loop (observe → reason/plan → act) where an LLM decides which tools to call, in what order, and synthesizes the results — trading predictability for flexibility, and introducing an entirely new class of nondeterministic failure modes that traditional software observability cannot catch.

## Frameworks Introduced
- **The Agentic Loop** (Observation → Reasoning/Planning → Action, repeated until goal completion): the core execution cycle of every agent, single or multi-agent. Debugging requires instrumenting each stage since a failure in Action cascades into a flawed Observation next cycle.
- **The Agentic Stack** (3 layers): Reasoning LLM (the "brain" — decomposes goals, plans, decides tool calls, synthesizes final answer) → Agent Orchestration (executes tool calls, formats results back to the LLM) → Tools (APIs, DB queries, RAG/retrieval calls, action tools). Map any agent framework (LangChain, LlamaIndex, CrewAI) onto these three layers to understand what it's actually doing.
- **Retrieval-as-a-tool vs. RAG-as-a-tool**: give the agent a raw retrieval tool (agent does reasoning/synthesis itself — more control/transparency, good for exploratory workflows) or a full RAG tool (tool does retrieval+generation, returns a grounded answer — more consistent, less agent-side orchestration). Neither is "more agentic"; pick based on how much grounding logic you want centralized vs. agent-controlled.
- **Single-agent vs. multi-agent decision**: default to a single, well-prompted agent (faster, cheaper, simpler to deploy — 30–50% faster response times, no inter-agent overhead). Justify multi-agent only when you hit one of three limits: distinct security domains (isolate sensitive data access), vast tool surfaces (tool dilution/confusion from too many tools in one context), or organizational boundaries (independent teams owning separate subtask logic).
- **Orchestrator-worker pattern**: the recommended middle ground for multi-agent design — a central orchestrator decomposes tasks and calls specialized subagents in parallel, but subagents never talk directly to each other. Avoids "emergent chaos" of peer-to-peer agent communication while giving context isolation (each subagent sees only what it needs).
- **MCP (Model Context Protocol)**: standardizes how agents discover and call tools/data via three primitives — Tools (executable functions), Resources (URI-addressable context/state, for large results the LLM reads separately), Prompts (reusable parameterized instruction templates). Client-host-server architecture decouples the agent from tool implementation details.
- **A2A (Agent-to-Agent) protocol**: the "horizontal" complement to MCP's "vertical" tool access — lets independently built agents (possibly from different vendors) discover each other's capabilities via an "Agent Card" and delegate tasks, enabling true cross-vendor multi-agent interoperability.
- **Agentic Failure Taxonomy** (distinct from RAG failures in Ch6): tool hallucination (agent trusts a wrong tool output), response hallucination (tool was right, agent misused/distorted it), goal misinterpretation, plan generation failure (wrong step ordering), incorrect tool use (wrong tool or bad arguments — mitigate via read/write permission boundaries to limit "blast radius"), verification/termination failure (stops too early or loops forever), prompt injection.
- **Agentic Observability**: extends traditional observability (metrics/logs/traces) with two new layers — Evaluations ("how well is the agent performing?") and Governance ("is it operating within its rules?"). Necessary because an agent can be operationally "healthy" (low latency, zero system errors) while completely failing its actual task — a disconnect traditional monitoring can't see.

## Key Concepts
- **ReAct (Reason + Act)**: the foundational LLM prompting paradigm — interleave a "thought" (reasoning about what to do), an "action" (tool call), and an "observation" (result) in a loop; the basis for most modern tool-calling agents.
- **Tool calling / function calling**: LLM capability to emit a structured request (tool name + typed arguments as JSON) for the orchestration layer to execute. Requires precise tool naming/description to avoid "tool confusion" (ambiguous names cause wrong-tool selection).
- **Parallel tool calling**: an LLM issuing multiple independent tool calls in a single turn (e.g. three `get_revenue(year, ticker)` calls for three different years) instead of sequentially.
- **Tool dilution**: degraded tool-selection accuracy when an agent has too many tools available in context — a key driver toward multi-agent decomposition (each subagent gets a tightly scoped toolset).
- **Session-based storage vs. semantic search/retrieval** (agentic memory): short-term memory = simple session-scoped interaction list (with periodic "session consolidation" into compressed summaries to save tokens); long-term memory = vector-store-backed semantic search over persisted "memories," auto-injected into working context per query.
- **Memory poisoning**: an attacker plants a false/malicious "memory" (e.g. via prompt injection) that later gets treated as trusted fact — mitigate with a validation gate (a lightweight secondary LLM check before any memory is persisted).
- **Trace / span**: a trace is the full execution record of one agent request; each span is one discrete unit of work (an LLM call, a tool call) within it — the primary debugging tool for agents, since it reveals things a final-output log can't (e.g. 5 failed retries on a tool before success).
- **Agentic observability metrics**: token usage, inference latency (time-to-first-token + end-to-end), LLM/API call counts (unexpectedly high counts signal loops or convoluted reasoning), tool call success/failure rate, response quality (hallucination rate + tool-use efficiency + multiturn coherence + autonomy alignment), human handoff rate.
- **OpenTelemetry (OTel) GenAI semantic conventions**: the emerging vendor-neutral standard for AI telemetry (traces/metrics/logs), preventing observability vendor lock-in across agent frameworks.

## Mental Models
- Think of the historical arc as: rule-based symbolic agents (1980s-90s, brittle, no NLU) → API-connected but still-scripted assistants (Siri/Alexa, 2010s, reactive single-command) → LLM-reasoning-driven agents (ReAct-era, proactive goal-driven loops) — the "missing ingredient" that finally closed the loop was the LLM as a flexible general-purpose reasoning engine.
- Treat multi-agent systems as a cost you pay for a specific capability, not a default architecture — the "complexity tax" (latency, token cost, harder tracing, concurrency bugs) is real, so justify it against the three specific limits (security domains, tool surface, org boundaries), not "it sounds more sophisticated."
- Use the read/write permission boundary as your primary blast-radius control for agent mistakes: a read-only email tool can't delete anything no matter how badly the LLM reasons — permission scoping is a stronger safety net than better prompting.
- View long-term memory as a governance liability, not just a feature: every persisted memory is a GDPR/CCPA-relevant record requiring compliance deletion, redaction, and decay policies — only add it when the agent's value is genuinely cumulative across sessions (a persistent assistant), not for one-off transactional agents.
- Human-in-the-loop is not a stopgap for weak agents — it's the correct architecture for regulated/irreversible-action domains (moving money, clinical decisions) regardless of how good the agent's reasoning becomes.

## Anti-patterns
- **Defaulting to multi-agent architecture for a simple, well-defined task**: adds latency, cost, and observability difficulty without justification — start with a single well-prompted agent unless you hit one of the three specific multi-agent triggers.
- **Peer-to-peer agent communication for predictable production workflows**: creates "emergent chaos" that's hard to trace and debug — prefer the orchestrator-worker pattern unless the task genuinely needs dynamic, emergent collaboration.
- **Blindly trusting tool output without validation**: leads to tool hallucination — a RAG tool or text2SQL tool that returns a wrong answer/query gets propagated straight into the agent's final response if there's no verification step.
- **Vague or overlapping tool names/descriptions** (e.g. generic `data_lookup`): causes tool confusion and wrong-tool selection — use semantically precise names, typed arguments, and scope/boundary-explicit descriptions.
- **Persisting every interaction to long-term memory by default**: unnecessary privacy risk and compliance burden for transactional/one-off agents — only persist when cumulative personalization genuinely improves the user experience.
- **Applying traditional infrastructure observability (CPU, latency, error rate) as your only agent monitoring**: an agent can look perfectly healthy operationally while completely failing its task (wrong tool, wrong plan, hallucinated synthesis) — traditional metrics cannot detect this; you need tracing + AI-specific evaluation.
- **Full autonomy for irreversible or regulated actions** (moving money, clinical orders): the "black box" nature of agentic reasoning is often a compliance nonstarter — require human approval gates before any irreversible action in these domains.

## Code Examples

```python
# OpenAI tool/function calling definition and invocation
weather_tool = {
    "type": "function",
    "name": "get_weather",
    "description": "Get current temperature for provided coordinates in Celsius.",
    "parameters": {
        "type": "object",
        "properties": {"latitude": {"type": "number"}, "longitude": {"type": "number"}},
        "required": ["latitude", "longitude"],
        "additionalProperties": False,
    },
}
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What's the weather like in Oakland today?"}],
    functions=[weather_tool],
    function_call={"name": "get_weather"},
)
# response.choices[0].message.function_call ->
# FunctionCall(arguments='{"latitude":37.8044,"longitude":-122.2711}', name='get_weather')
```
- **What it demonstrates**: the LLM resolves "Oakland" to actual coordinates and emits a structured, typed tool call — the orchestration layer then executes the real Python function with those arguments.

```python
# ReAct-style agent with a single RAG tool (LangChain)
@tool
def rag_gpt_tool(question: str) -> str:
    """Use this tool to answer questions about the 'GPT-2' paper."""
    return rag_chain.invoke(question)

agent = create_react_agent(llm, tools=[rag_gpt_tool])
# Given a compound question, the agent decomposes it into two sub-questions,
# calls rag_gpt_tool twice, and synthesizes a combined final answer.
```
- **What it demonstrates**: ReAct's core value — decomposing a compound question ("size of GPT-2 AND its performance impact") into independently retrievable sub-questions, then synthesizing.

```python
# CrewAI: two-agent sequential crew (researcher -> writer)
researcher = Agent(role="Senior Market Research Analyst", goal="Find emerging AI trends", allow_delegation=False)
writer = Agent(role="Technology Content Strategist", goal="Craft a blog post from research findings", allow_delegation=False)

research_task = Task(description="Identify top 3 AI trends...", agent=researcher)
writer_task = Task(description="Write a 500-word blog post...", agent=writer, context=[research_task])

crew = Crew(agents=[researcher, writer], tasks=[research_task, writer_task], process=Process.sequential)
result = crew.kickoff(inputs={'topic': 'AI trends'})
```
- **What it demonstrates**: explicit task dependency (`context=[research_task]`) ensures the writer only runs after and uses the researcher's output — the sequential multi-agent pattern.

## Reference Tables

| Multi-agent trigger | Why single agent falls short |
|---|---|
| Distinct security domains | Sensitive data access must be isolated per task |
| Vast tool surfaces | Too many tools in one context → tool dilution/confusion |
| Organizational boundaries | Independent teams need to own/version separate subtask logic |

| Traditional observability | Agentic observability |
|---|---|
| Focus: system health (CPU, latency, errors) | Focus: agent behavior, reasoning, goal alignment |
| Failure modes: hardware/software errors, request failures | Tool hallucination, response hallucination, goal misinterpretation, incorrect tool use, planning failure, prompt injection |
| Core assumption: operational stability ⇒ functional correctness | An agent can be operationally healthy yet still fail its task |

| Observability tool | Type | Strength |
|---|---|---|
| Langfuse | Open source | Tracing, cost/latency monitoring, eval dataset creation from production traces |
| Arize Phoenix | Open source (OTel-based) | Tracing + evaluation templates for routers/planners/retrieval |
| LangSmith | Commercial | Deep LangChain integration, prompt playground, evaluation framework |

## Worked Example
A LlamaIndex `FunctionAgent` (Claude Sonnet 4.5) is given three tools — `web_search` (Tavily), a calculator, and a RAG tool over the GPT-2 paper — and a system prompt establishing it as a "research & planning assistant." Asked to plan a Santa Cruz day trip (find 2-3 activities with links, estimate ticket costs for two adults, build a 6-hour itinerary), the agent: (1) issues three parallel `web_search` calls for activities and prices (Boardwalk, Natural Bridges, Monterey Bay Aquarium), (2) calls the `calculator` tool with `44.95 + 44.95 + 10 + 65 + 65` to sum the total cost (~$229.90), then (3) synthesizes a final structured itinerary with costs, links, and a timed schedule. This demonstrates the full agentic loop end-to-end — observation (user goal) → reasoning (decide which tools, in what sequence) → action (parallel search calls, then a dependent calculator call) → final synthesis — and shows why *streaming the agent's intermediate events* (not just the final answer) is essential for debugging: without it, you'd never see that three separate searches were needed to gather enough grounding data before the calculation step could run.

## Key Takeaways
1. An AI agent is RAG extended into a loop: the LLM decides *what* to retrieve, *when*, and *how many times*, rather than executing one fixed retrieve-then-generate pass.
2. Default to a single, well-prompted agent; only adopt multi-agent architecture when you hit a specific limit (security isolation, tool-surface dilution, or organizational ownership boundaries) — the complexity tax is real.
3. The orchestrator-worker pattern is the safer entry point into multi-agent design versus full peer-to-peer collaboration, because it keeps orchestration predictable and preserves context isolation.
4. MCP standardizes agent-to-tool access (the "vertical" stack); A2A standardizes agent-to-agent collaboration (the "horizontal" stack) — together they enable interoperable, vendor-agnostic agentic ecosystems.
5. Long-term memory is a governance liability as much as a capability — implement compliance deletion, memory-poisoning validation gates, and decay policies whenever you persist agent memory.
6. Traditional infrastructure observability cannot detect agent task failures — you need tracing (spans per LLM/tool call) plus AI-specific metrics (tool success rate, hallucination rate, human handoff rate) to make agent behavior legible.
7. Permission scoping (read-only vs. write access on tools) is a stronger safety control than prompt engineering alone — it bounds the "blast radius" of an incorrect-tool-use failure regardless of the LLM's reasoning quality.

## Connects To
- **Ch1**: agentic RAG was previewed there as an "advanced RAG" technique; this chapter is its full treatment.
- **Ch3**: prompt injection and guardrails introduced for RAG generation are "even more dangerous" once agents can take real-world actions — this chapter extends those controls to the agentic context.
- **Ch6**: RAG evaluation metrics (faithfulness, hallucination) are the foundation for the additional agent-specific dimensions (tool-use efficiency, multiturn coherence, autonomy alignment) introduced here.
- **Ch9**: multi-hop/sensemaking queries flagged as an "architectural limit" of standard retrieval (Ch6) are addressed by either agentic RAG (this chapter) or knowledge graphs (next).
