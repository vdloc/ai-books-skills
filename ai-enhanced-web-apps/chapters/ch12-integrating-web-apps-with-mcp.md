# Chapter 12: Integrating Web Apps with the Model Context Protocol

## Core Idea
The Model Context Protocol (MCP) standardizes how AI applications discover and invoke external tools/data sources via a client-server protocol, replacing framework-specific, non-reusable tool integrations (LangChain tools, Vercel AI SDK tool calling) with a portable format any MCP-compatible client or server can use — and the ecosystem is evolving toward gateways, MCP-as-a-service, and directories.

## Frameworks Introduced
- **MCP client-server architecture**: MCP client (AI agent/SDK) requests capabilities and invokes tools → MCP server exposes tools, executes them, returns results → both communicate over a well-defined bidirectional protocol.
  - When to use: whenever tool integrations need to be reusable across frameworks/assistants rather than hardwired into one app's tool-calling code (contrast with Ch4's Vercel-native tool calling or Ch6's LangChain agents, both framework-specific).
  - How: an app can be an MCP client (consuming external tools), an MCP server (exposing its own tools to other agents), or both.
- **Stdio-transport local MCP integration**: `StdioServerTransport`/`StdioClientTransport` runs the MCP server as a separate local Node.js process communicating over stdin/stdout (JSON-RPC-style messages) — no network config needed for local demos.
  - When to use: local development, single-machine tool isolation, before moving to HTTP-based remote MCP servers.
  - How: the Next.js API route spawns the MCP server process via `StdioClientTransport({ command: 'node', args: [...] })`, retrieves its tools via `mcpClient.tools()`, and passes them straight into `streamText({ tools })` — the model never touches the external API directly.
- **MCP gateway pattern (emerging)**: a single AI-aware proxy that an agent connects to once; the gateway itself talks to every registered MCP server, centralizing routing, security/auth, context management, and orchestration.
  - When to use: apps needing many integrations (calendar, email, commerce, CRM) where duplicated per-server auth/state becomes unmanageable.

## Key Concepts
- **MCP server**: exposes named, schema-typed tools (e.g., `get-chuck-joke`) with a description and handler; returns MCP-compliant structured responses (`{ content: [{ type: "text", text }] }`).
- **`experimental_createMCPClient`**: Vercel AI SDK function creating an MCP client bound to a transport, exposing `.tools()` to fetch the server's tool list for use directly in `streamText`/`generateText`.
- **Model isolation from external APIs**: the LLM only ever "knows about" the tools the MCP server exposes — it never has direct network access to the Chuck Norris API (or any external service) itself, which is mediated entirely by the MCP server.
- **MCP-as-a-service**: businesses expose their own services as an MCP endpoint (e.g., `mybookstore.com/mcp`) so any compatible AI assistant can "Add this store" and interact with inventory/orders conversationally — analogous to how RSS/SaaS changed content/software distribution.
- **MCP registries/directories**: the official MCP Registry plus community catalogs (Glama, Pulse MCP) and CLI tools (mcpreg) addressing the current fragmented discovery problem for finding trustworthy MCP servers.

## Mental Models
- Think of MCP as solving the *reuse* problem that framework-specific tool calling (Ch4's Vercel AI SDK tools, Ch6's LangChain agents) doesn't: a tool built for LangChain doesn't transfer to LlamaIndex or the Vercel AI SDK without rewriting, whereas an MCP server is consumable by any MCP-compatible client regardless of framework.
- The stdio transport pattern enforces a hard security boundary by construction: because the model only receives tool definitions (not API credentials or network access), all external API calls are mediated and centralized in one place (the MCP server) — this is a security *architecture* choice, not just a convenience.
- Treat the MCP ecosystem's current stage (gateways, MCP-as-a-service, directories all "emerging") as roughly analogous to the pre-search-engine internet — expect rapid evolution; the protocol's value proposition (standardization) is durable even as specific tooling churns.

## Anti-patterns
- **Letting the AI model call external APIs directly**: reintroduces the "messy, hard to maintain" coupling MCP is designed to eliminate — always mediate external access through a tool-exposing server (MCP or otherwise).
- **Rebuilding the same tool integration per framework** (once for LangChain, again for Vercel AI SDK, again for LlamaIndex): exactly the fragmentation problem MCP solves — build it once as an MCP server instead.
- **Connecting an agent directly to many independent MCP servers without a gateway** at scale: duplicates authorization logic and creates fragile, inconsistent-state orchestration — introduce a gateway once integration count grows.
- **Trusting an MCP server from an unverified/niche catalog without review**: the discovery ecosystem is still immature (no mature trust/rating system yet) — apply the same scrutiny as installing any third-party dependency.

## Code Examples
```js
// MCP server exposing a single tool (Node.js process, stdio transport)
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new McpServer({ name: 'chuck-norris-mcp', version: '1.0.0' });

server.tool("get-chuck-joke", "Fetch a random Chuck Norris joke", {}, async () => {
  try {
    const response = await fetch("https://api.chucknorris.io/jokes/random");
    if (!response.ok) return { content: [{ type: "text", text: "No joke available at the moment." }] };
    const data = await response.json();
    return { content: [{ type: "text", text: data.value }] };
  } catch (err) {
    return { content: [{ type: "text", text: `Error fetching joke: ${err.message}` }] };
  }
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}
main();
```
```js
// Next.js API route as MCP client, feeding tools into streamText
import { createGoogleGenerativeAI } from '@ai-sdk/google';
import { streamText, convertToModelMessages, experimental_createMCPClient } from 'ai';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio';

const gemini = createGoogleGenerativeAI({ apiKey: process.env.GEMINI_API_KEY || '' });

export async function POST(req) {
  const { messages } = await req.json();
  const transport = new StdioClientTransport({ command: 'node', args: ['src/stdio/server.js'] });
  const mcpClient = await experimental_createMCPClient({ transport });
  const tools = await mcpClient.tools();

  const result = streamText({
    model: gemini('gemini-2.5-flash'),
    messages: convertToModelMessages(messages),
    tools,
    system: 'You are a helpful assistant that can call tools when needed.',
    onFinish: async () => { await mcpClient.close(); },
    onError: async () => { await mcpClient.close(); },
  });
  return result.toUIMessageStreamResponse();
}
```

## Reference Tables
| Layer | Role |
|---|---|
| Browser (React + `useChat`) | Captures user input, renders streamed response |
| Next.js API route | Broker between frontend and Vercel AI SDK |
| Vercel AI SDK | Interprets model responses, manages MCP tool calls |
| MCP server | Exposes callable tools, mediates external API access |
| External API/data source | The actual capability being wrapped |

| MCP future direction | Problem addressed |
|---|---|
| MCP gateway | Duplicated auth/state across many direct server connections |
| MCP-as-a-service | Businesses need a standard "Add us to your AI" mechanism |
| MCP directories/registries | Fragmented, low-trust discovery of available servers |

## Worked Example
The Chuck Norris joke integration end-to-end: the MCP server (`server.js`) registers a `get-chuck-joke` tool that fetches from `api.chucknorris.io`; the Next.js `/api/chat` route spawns this server as a subprocess via `StdioClientTransport`, creates an MCP client, and retrieves its tool list with `mcpClient.tools()`; these tools are passed straight into `streamText({ model: gemini(...), tools })`. When a user asks for a Chuck Norris fact, the Gemini model — which has no direct network access — decides to call `get-chuck-joke()`; the call flows as a JSON-RPC message over stdio to the separate MCP server process, which performs the actual HTTP fetch and returns a structured MCP response; the result streams back through the SDK to the chat UI. The entire external-API surface is centralized in one file, `server.js`, independent of which LLM or frontend framework is used.

## Key Takeaways
1. MCP exists specifically to solve tool-integration reuse across frameworks — a tool built as an MCP server works with any MCP-compatible client, unlike LangChain-specific or Vercel-SDK-specific tool definitions.
2. The client-server split gives you a hard security boundary for free: the LLM never gets direct API/network access, only tool definitions — all external calls funnel through the MCP server.
3. Use stdio transport for local single-machine development; expect production/remote setups to use HTTP-based MCP servers instead.
4. As integration count grows, plan for an MCP gateway to centralize auth/routing/context rather than wiring many direct server connections.
5. The MCP ecosystem (registries, gateways, MCP-as-a-service) is still immature — apply normal third-party dependency scrutiny when adopting community MCP servers.

## Connects To
- **Ch4**: MCP is explicitly framed as a more portable alternative to the Vercel AI SDK's native tool-calling pattern introduced there.
- **Ch6**: contrasted with LangChain's agent/tool abstraction — MCP solves the cross-framework reuse problem LangChain tools don't.
- **Ch9**: the "model isolated from external APIs" security boundary echoes the backend-as-secure-intermediary principle from the security chapter.
- **Ch11**: this chapter's tool-exposure model is a natural next step for the RAG agent's knowledge-base tools to become reusable across assistants.
