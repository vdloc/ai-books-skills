# Chapter 2: Building Agents with Tools

## Core Idea
Turning a Python function into an agent tool is just `@tool` + type hints + a clear docstring — the agent reads that metadata to decide *when* to call the tool and *how* to extract parameters from natural language, so you never write intent-classification or routing logic yourself.

## Frameworks Introduced
- **`@tool` decorator (Strands)**: converts any typed, docstringed Python function into an agent-callable tool.
  - When to use: any time an agent needs to do something beyond generating text (query an API, read a file, calculate).
  - How: add `@tool`, type-hint every parameter and the return value, write a docstring with an `Args:` section — the agent uses exactly these three signals (name, params, docstring) to decide when/how to call it.
- **Single-purpose tool design**: build focused tools (`get_sales_data`, `analyze_sales`, `send_email`) rather than one mega-tool.
  - When to use: always — the agent composes multiple single-purpose tools into a sequence itself; you never write the `if X then steps 1,2,3` orchestration logic.
  - Why it works: reusability (the same `analyze_sales` tool works regardless of data source) and reasoning simplicity for the LLM (it's easier to pick the right narrow tool than to guess the right mode of a broad one).
- **Class-based tools for shared resources**: group related `@tool`-decorated methods inside a class with a shared `__init__` (e.g. one DB connection for `check_stock`/`update_stock`).
  - When to use: when independent `@tool` functions each open their own expensive resource (DB connection, API client) and you're hitting connection limits (MySQL default 151, PostgreSQL default 100) under concurrent load.
  - Failure mode without this: 5 tools called once per request × dozens of concurrent users = connection exhaustion and "too many connections" errors — a real production incident, not a theoretical one.
- **Async tools for parallel I/O**: mark a tool `async def` with `await` on the I/O call; Strands runs independent async tool calls concurrently via `agent.invoke_async(...)`.
  - When to use: when an agent needs to call the same/similar slow I/O-bound tool multiple times with independent inputs (e.g. checking 3 warehouses) — sequential calls (2s×3 + 2s combine = 8s) become concurrent (2s + 2s combine = 4s).
  - When NOT to use: tools that run in under ~100ms or do pure calculation — async adds complexity with no measurable gain there.
- **Strands Agent Builder**: a CLI (`pipx install strands-agents-builder`, then `strands`) for building/testing agents and tools via plain-English description, no code required.
  - When to use: rapid prototyping/learning, not production (use hand-written agents with full control once you're past experimentation).

## Key Concepts
- **Function calling / tool use**: the general term for giving an LLM-based agent the ability to invoke external functions.
- **`strands-agents-tools`**: the pre-built community tools package (`pip install strands-agents-tools`) covering file ops, math, web/search, code execution, AWS services, memory, communication, image/video, automation, and multi-agent coordination.
- **`use_aws` tool**: a single Strands tool that translates natural-language requests into AWS CLI-equivalent calls across S3, DynamoDB, Lambda, and other services — one tool, many services.
- **MCP tools**: standardized tools from the broader MCP ecosystem (browsable at mcpmarket.com) that plug into Strands without custom integration code (full depth in Ch5).
- **`TOOL_SPEC`**: an advanced mechanism for defining explicit input-validation schemas on tools (formats, allowed values) beyond what type hints alone enforce.
- **`invoke_async`**: the Strands Agent method used to run an agent that has async tools, enabling concurrent tool execution.

## Mental Models
- Think of the LLM as "the brain," tools as "the hands and senses," and the agent as "the coordinator" managing the conversation between them (Figure 2.1) — this is the same three-part split as Ch1's Tools/Memory/Planning/LLM breakdown, focused specifically on the Tools↔LLM relationship.
- Use "start with the decorator, escalate only when a real problem forces it": plain `@tool` → class-based tools (shared-resource problem) → async tools (latency problem). Let production symptoms (a DBA's Slack message, a slow-response complaint), not speculation, decide which escalation you need.
- Think of the tool ecosystem as four pillars you draw from in order of preference: pre-built community tools first, custom `@tool` functions for business-specific logic, MCP tools for third-party services, AWS service integrations for cloud infrastructure — "the agent doesn't care where the tools come from; it just sees a unified catalog."

## Anti-patterns
- **One mega-tool that does everything**: harder for the agent to reason about which capability to invoke, harder to maintain, and impossible to reuse pieces independently — always prefer several single-purpose tools.
- **Opening a new resource connection per tool call in production**: works fine in a demo, then floods the database with connections under real concurrent load — group tools sharing a resource into a class with one shared connection instead.
- **Reaching for async on every tool "just in case"**: adds unnecessary complexity for tools that already run fast or do pure computation; only async-ify tools with genuinely slow, independent I/O calls.
- **Writing manual intent classifiers/regex to route user phrasing to the right tool**: this is exactly the work `@tool` + docstring is meant to eliminate — if you're writing "if the user says 'in stock' or 'available' or 'can I order'..." you've missed the point of tool calling.

## Code Examples
```python
from strands import tool
import requests

@tool  # Decorator
def check_server_status(server_url: str) -> str:  # Type hints
    """Check if a server is responding by making an HTTP request.

    Args:
        server_url: The URL of the server to check  # Docstring

    Returns:
        A message indicating whether the server is up or down
    """
    try:
        response = requests.get(server_url, timeout=5)
        return f"Server is up. Status code: {response.status_code}"
    except requests.exceptions.RequestException:
        return "Server is down or unreachable"
```
- **What it demonstrates**: the minimal three-part contract (`@tool`, type hints, docstring with `Args:`) that lets an agent discover and correctly call a plain Python function.

```python
import asyncio
from strands import Agent, tool

@tool
async def check_warehouse_inventory(product_id: str, warehouse: str) -> int:
    """Check inventory at a specific warehouse."""
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"https://api.warehouse-{warehouse}.com/inventory/{product_id}"
        )
        return response.json()["quantity"]

async def main():
    agent = Agent(tools=[check_warehouse_inventory])
    response = await agent.invoke_async(
        "Can we ship 100 units of PROD-123? Check all warehouses."
    )
    print(response.message['content'][0]['text'])
asyncio.run(main())
```
- **What it demonstrates**: converting a sequential 8-second, 3-warehouse check into a concurrent 4-second one by marking the tool `async` and calling the agent via `invoke_async` — Strands parallelizes independent async tool calls automatically.

## Reference Tables

| Category | Example Tools | What They Do |
|---|---|---|
| File Operations | `file_read`, `file_write`, `editor` | Read/write files, advanced editing with syntax highlighting |
| Mathematical Operations | `calculator` | Calculations, symbolic math, equation solving |
| Web and Search | `http_request`, `tavily_search`, `tavily_extract`, `exa_search`, `bright_data` | API calls, real-time web search, content extraction, scraping |
| Code Execution | `python_repl`, `shell`, `code_interpreter` | Run Python/shell code, isolated sandbox execution |
| AWS Services | `use_aws`, `retrieve`, `nova_reels`, `agent_core_memory` | S3/DynamoDB/Lambda interaction, Knowledge Base queries, video creation, memory storage |
| Memory and Storage | `memory`, `mem0_memory`, `mongodb_memory`, `elasticsearch_memory` | Store/retrieve documents and memories across runs |
| Communication | `slack`, `speak` | Post to Slack, text-to-speech |
| Image and Video | `generate_image`, `image_reader`, `search_video`, `chat_video` | Create/analyze images, semantic video search |
| Automation and Utilities | `browser`, `use_computer`, `cron`, `current_time`, `sleep`, `diagram`, `rss` | Web automation, desktop control, scheduling, diagramming |
| Advanced Features | `swarm`, `workflow`, `use_llm`, `batch`, `mcp_client` | Multi-agent coordination, automated workflows, nested AI loops, parallel execution |

## Worked Example
Escalating a tool implementation through all three levels as production problems surface, using the inventory-agent scenario from the book:
1. **Start simple** — a plain `@tool`-decorated `check_stock`/`update_stock` pair, each opening its own DB connection per call.
2. **Hit the connection-limit wall** — a DBA reports 50 connections/minute; the fix is grouping both methods into an `InventoryTools` class with one `self.db` connection created in `__init__`, then passing bound methods (`inventory.check_stock`, `inventory.update_stock`) to the agent — one connection now serves five tools.
3. **Hit the latency wall** — a product manager reports 8-second responses because `check_warehouse_inventory` is called sequentially across 3 warehouses (2s each) plus a 2s combine step. Marking the tool `async def` with `httpx.AsyncClient` and switching the agent call to `await agent.invoke_async(...)` lets Strands run all three warehouse checks concurrently, cutting total time from 8s to 4s.
The lesson: don't pre-optimize — start with the decorator, and let a real Slack complaint or a real latency metric tell you which escalation (class-based sharing, or async) you actually need.

## Key Takeaways
1. `@tool` + type hints + docstring is the entire contract between a Python function and an agent — no separate registration or intent-mapping step is needed.
2. Build many single-purpose tools, not one do-everything tool; the agent handles sequencing and passing data between them on its own.
3. When independent tool calls share an expensive resource (DB connections), group them into a class with a shared `__init__` state — this is the fix for connection exhaustion under concurrent load.
4. When independent tool calls are slow, independent I/O operations (multiple API/warehouse checks), mark them `async` and call the agent with `invoke_async` — Strands parallelizes them automatically.
5. `strands-agents-tools` (`pip install strands-agents-tools`) gives you dozens of pre-built tools (calculator, `use_aws`, http_request, etc.) before you should write anything custom.
6. `use_aws` is the single tool pattern for AWS integration: one tool, many services (S3, DynamoDB, Lambda), driven entirely by natural-language requests.
7. Strands Agent Builder (`strands` CLI) is for rapid prototyping only — move to hand-written agents for production control.

## Connects To
- **Ch1**: extends the "Tools" component of the four-part agent architecture (Tools, Memory, Planning, LLM) introduced there.
- **Ch3**: picks up the next limitation — agents built here have no memory across conversations; Ch3 fixes that with AgentCore Memory and Mem0.
- **Ch5**: goes deeper into MCP tools, only briefly introduced here as one of the four tool-ecosystem pillars.
- **Ch6**: production deployment will need the connection-management lessons from this chapter (class-based shared resources) at scale.
