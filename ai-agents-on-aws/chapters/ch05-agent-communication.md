# Chapter 5: Agent Communication

## Core Idea
MCP and A2A solve two different interoperability problems and are complementary, not competing: MCP standardizes how an agent connects to tools/data/prompts (agent↔tool, turning N×M custom integrations into N+M), while A2A standardizes how independently-built agents discover and collaborate with each other (agent↔agent) while keeping internal implementation opaque.

## Frameworks Introduced
- **MCP (Model Context Protocol)**: Host (the AI app/agent) → MCP Client (1:1 with a server, discovers/invokes capabilities) → MCP Server (exposes Tools/Resources/Prompts) → Data sources.
  - When to use: any time an agent needs to call external tools/read external data/use reusable prompt templates without hardcoding a custom integration per tool.
  - How it collapses integration cost: traditional per-tool integration is N×M (N models × M tools); MCP reduces this to N+M by giving every model and every tool one connection to the shared protocol layer.
- **MCP transport mechanisms**: Stdio (local, same-machine, simple/lightweight), SSE/Streamable HTTP (remote/web, incremental real-time delivery), Custom transports (e.g. gRPC for enterprise high-performance needs).
  - When to use: Stdio for a locally-run agent; Streamable HTTP for remote/cloud-hosted servers needing live updates; custom transport only when built-ins don't fit.
- **MCP lifecycle (3 stages)**: Initialization (handshake — agree on protocol version/capabilities) → Operation (normal request/response) → Shutdown (graceful client-initiated termination).
- **MCP server features (3 capability types)**: Tools (actions — same concept as Ch2, each execution needs explicit user approval, discovered via `tools/list`, invoked via `tools/call`), Resources (read-only data via `resources/list`/`resources/read`, no side effects — for browsing/referencing, not acting), Prompt templates (versioned, parameterized, reusable instruction templates fetched via `get_prompt`).
  - When to use Resources vs Tools: use a Resource when the agent just needs to *read* static/semi-static context (a course directory); use a Tool when an *action* needs to happen (sending a message) — often combined: read a Resource to find who, then call a Tool to act.
- **MCP client features**: Sampling (server asks the client's model to run inference on the server's behalf, with user approval — reverses the usual client→server direction), Roots (client exposes filesystem boundaries the server may operate within), Elicitation (server requests missing info mid-session via form mode or URL mode, keeping sensitive data off the client where needed).
- **FastMCP vs FastAPI**: FastMCP is MCP-native, minimal boilerplate, best for a dedicated MCP server; FastAPI is a general web framework, better when MCP is one part of a larger service (REST, auth, dashboards).
- **A2A (Agent-to-Agent Protocol)**: Client Agent (discovers others via Agent Cards, initiates requests) ↔ Remote/Server Agent (opaque — hosts capabilities at a URI, publishes its Agent Card at `/.well-known/agent-card.json`, manages Task lifecycle to produce Artifacts).
  - When to use: cross-framework, cross-organization agent collaboration where agents must stay opaque (no shared internals) — e.g. a Diagnosis Agent and an Insurance Agent from different systems negotiating a claim, each independently using its own MCP tools.
  - Why not just wrap an agent as an MCP tool: tools are for stateless, predefined actions; agents solve open-ended, iterative, multi-turn problems — collapsing an agent into a tool call loses its reasoning/delegation capability.
- **A2A communication styles (4 modes)**: Synchronous (`SendMessage`, blocking request-response, quick tasks), Asynchronous/polling (returns a Task ID immediately, client polls `GetTask` until completion — long-running jobs), Streaming (`SendStreamingMessage` over SSE, requires `capabilities.streaming=true` in the Agent Card, incremental progress), Push notifications (server POSTs to a client-provided webhook on completion, requires `capabilities.pushNotifications=true` — for long-running/event-driven work where the client can't stay connected).
- **A2A implementation layers (3-tier)**: A2A SDK (`a2a-sdk`, raw protocol primitives — `Message`, `TextPart`, `AgentSkill`, `A2ACardResolver`, `ClientFactory`) → framework integrations (Strands `A2AServer`/`A2AAgent`, Google ADK — implement the protocol while preserving the framework's programming model) → platform-specific (LangSmith Agent Server — managed hosting, protocol details handled for you).

## Key Concepts
- **Agent Card**: a JSON document (`/.well-known/agent-card.json`) describing an agent's `name`, `capabilities` (how it communicates — streaming, push notifications), `skills` (what work it does), `url`, and `protocolVersion` — the A2A discovery mechanism, analogous to a Swagger/OpenAPI doc.
- **Protocol binding**: the transport an A2A agent supports (HTTP, gRPC, or JSON-RPC) — declared in the Agent Card; two agents must share a binding to talk directly.
- **Task / Artifact / Part**: a Task tracks a unit of work the remote agent is executing; it produces Artifacts (output containers) made of Parts (e.g. TextPart); `GetTask` retrieves current status and artifacts.
- **Resource template**: a parameterized URI pattern (e.g. `travel://activities/{city}/{category}`) letting an MCP client query specific data instances.
- **Opaque agent**: an A2A design principle — Agent A never needs Agent B's internal code/prompts/data source, only the A2A request/response contract.
- **JSON-RPC**: the UTF-8-encoded message format MCP uses for all client-server communication, regardless of transport.

## Mental Models
- Think of MCP as USB for AI agents: one standard port (the protocol) lets any compliant tool plug into any compliant agent, instead of a custom cable (integration) per device (tool) per computer (model).
- Think of A2A as HTTP for agents: just as HTTP let arbitrary web servers and browsers interoperate without knowing each other's internals, A2A lets agents built on different frameworks (LangGraph, CrewAI, Strands) interoperate through Agent Cards and standardized messages.
- Use "tools are for actions, agents are for problems" to decide MCP vs A2A: if the capability is a deterministic, stateless action (send an email, query a DB) — expose it as an MCP tool. If the capability requires reasoning, delegation, or multi-turn negotiation — it needs to be a peer agent reached via A2A, not squeezed into a tool call.
- Pick the A2A communication style by how long the work takes and whether the client can wait: instant → Synchronous; slow but client will poll → Asynchronous; slow and client wants live progress → Streaming; slow and client can disconnect entirely → Push notifications.

## Anti-patterns
- **Wrapping an agent as a plain MCP tool to make it "callable"**: this is "fundamentally limiting" — it strips away the agent's ability to handle ambiguity, iterate, and delegate, reducing a problem-solver to a deterministic function call. Use A2A for agent-to-agent interaction instead.
- **Building custom point-to-point integration code for every tool/data source your agent needs**: this is the "fragmented app development" problem MCP exists to solve — N apps × M tools worth of bespoke, unmaintainable connectors, none of which benefit each other when one app fixes a bug.
- **Using Synchronous A2A calls for long-running work**: blocks the client agent needlessly — use Asynchronous polling, Streaming, or Push notifications depending on whether progress visibility or disconnect tolerance matters more.
- **Assuming two A2A agents can talk without checking protocol bindings**: an agent that only supports gRPC and one that only supports HTTP cannot communicate directly even though both are "A2A-compliant" — bindings must be negotiated via the Agent Cards.

## Code Examples
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("WeatherServer")

@mcp.tool()
def get_weather(city: str) -> str:
    """Get the current weather for a given city."""
    fake_weather = {"london": "15°C, cloudy", "paris": "18°C, sunny"}
    result = fake_weather.get(city.lower())
    return f"Weather in {city}: {result}" if result else f"Sorry, no weather data available for '{city}'."

@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Return a personalized greeting."""
    return f"Hello, {name}! Welcome to the WeatherServer."

@mcp.prompt()
def weather_report(city: str) -> str:
    """Generate a prompt asking for a weather report."""
    return f"Please provide a detailed weather report for {city}, including temperature, humidity, and forecast."

if __name__ == "__main__":
    mcp.run(transport="stdio")
```
- **What it demonstrates**: FastMCP's three primitives (`@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()`) in one minimal server, run over stdio.

```python
from strands.tools.mcp import MCPClient
from mcp import stdio_client, StdioServerParameters
from strands import Agent

mcp_client = MCPClient(lambda: stdio_client(
    StdioServerParameters(command="uvx", args=["awslabs.aws-documentation-mcp-server@latest"])
))

with mcp_client:
    tools = mcp_client.list_tools_sync()
    agent = Agent(tools=tools)
    response = agent("What is AWS Lambda?")
```
- **What it demonstrates**: consuming a pre-built MCP server (AWS Documentation) with zero custom integration code — tools are discovered and handed straight to a Strands agent.

```python
from a2a.types import AgentSkill
from strands_tools_a2a import A2AServer  # conceptual import shown in book

WEATHER_SKILLS = [AgentSkill(id="get_weather", name="Get Weather",
    description="Returns current weather conditions for a given city...",
    tags=["weather", "travel"], examples=["What is the weather in London?"])]

weather_agent = Agent(name="Weather Agent", tools=[get_weather], callback_handler=None)

server = A2AServer(agent=weather_agent, host="127.0.0.1", port=9001,
    skills=WEATHER_SKILLS, version="1.0.0", enable_a2a_compliant_streaming=True)
server.serve()
```
- **What it demonstrates**: exposing a Strands agent as a standalone A2A-compliant HTTP microservice — skills become the Agent Card's `skills` field, discoverable at `/.well-known/agent-card.json`.

## Reference Tables

| MCP Component | Purpose | What It Handles |
|---|---|---|
| Base Protocol | Core communication structure | JSON-RPC message types |
| Transport | Communication channel | Stdio, Streamable HTTP |
| Lifecycle Management | Connection management | Init, capability negotiation, session control |
| Authorization | Secure communication | Auth framework for HTTP-based transports |
| Server Features | Server capabilities | Resources, prompts, tools |
| Client Features | Client capabilities | Sampling, root directory lists |

| A2A Protocol Binding | Analogy | Best for |
|---|---|---|
| HTTP | Standard mail | Most common, easy to use |
| gRPC | High-speed courier | Fast, efficient for heavy data |
| JSON-RPC | Telegram | Strict remote-control rule set |

| A2A Communication Style | Mechanism | Requires | Best for |
|---|---|---|---|
| Synchronous | `SendMessage`, blocks for response | — | Quick, short-lived tasks |
| Asynchronous | Returns Task ID, client polls `GetTask` | — | Long-running jobs, no live updates needed |
| Streaming | `SendStreamingMessage` over SSE | `capabilities.streaming=true` | Live progress visibility |
| Push notifications | Server POSTs to client webhook | `capabilities.pushNotifications=true` | Long-running, client can disconnect |

## Worked Example
Building a multi-agent travel system where MCP and A2A operate at different layers simultaneously:
1. **Weather Agent Server** (port 9001): a Strands `Agent` with a `get_weather` tool, wrapped in `A2AServer` with `skills=WEATHER_SKILLS` — its Agent Card is auto-published at `http://127.0.0.1:9001/.well-known/agent-card.json`.
2. **Flights Agent Server** (port 9002): same pattern, two tools (`search_flights`, `book_flight`) and two skills.
3. **Travel Orchestrator** (A2A client, no LLM itself): for each target agent it (a) uses `A2ACardResolver` to fetch the Agent Card, (b) builds a non-streaming `ClientFactory(config).create(card)`, (c) sends a `Message` via `client.send_message(msg)`, (d) extracts text from the returned `artifacts`.
4. Run: three terminals — `python weather_agent_server.py`, `python flights_agent_server.py`, `python travel_orchestrator.py "Rome" "Paris" "Rome"` — the orchestrator discovers both agents via their cards, sends independent A2A requests, and prints a combined weather + flights trip summary.
This shows the complementary layering explicitly: A2A handles orchestrator↔weather-agent and orchestrator↔flights-agent communication, while each of those agents independently uses its own tools (which could just as easily be MCP-exposed) to do its actual work.

## Key Takeaways
1. MCP standardizes agent-to-tool/data/prompt access, collapsing N×M custom integrations into N+M connections through Host→Client→Server→DataSource.
2. A2A standardizes agent-to-agent collaboration across frameworks/organizations while keeping each agent's internals opaque — discovery happens via Agent Cards at `/.well-known/agent-card.json`.
3. Never reduce an agent to a plain MCP tool — tools execute deterministic actions, agents solve open-ended problems; A2A exists specifically to preserve that distinction.
4. Choose MCP transport (Stdio for local, Streamable HTTP for remote) and A2A communication style (Sync/Async/Streaming/Push) based on latency tolerance and whether the client can stay connected.
5. The three-tier A2A implementation stack — SDK (protocol primitives) → framework integration (Strands `A2AServer`/`A2AAgent`) → platform (LangSmith Agent Server) — lets you pick the right abstraction level for your control/convenience trade-off.
6. MCP and A2A are complementary, not overlapping: "A2A enables collaboration between the agents, and MCP enables each agent to do its own work by using external tools and data."

## Connects To
- **Ch1**: expands the MCP and A2A protocols first introduced there into full architectural and implementation detail.
- **Ch2**: MCP Tools are the same concept as Ch2's `@tool`-decorated functions, now exposed through a standardized server instead of being hardcoded into one agent.
- **Ch4**: A2A is explicitly distinguished from the Swarm pattern — Swarm orchestrates agents *within* one system; A2A standardizes communication *between* independently built agents/systems.
- **Ch6**: production deployment and scaling of these MCP/A2A-based agentic applications on AWS is covered next.
