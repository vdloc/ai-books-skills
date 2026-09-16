# Chapter 4: Managing Conversation and State in Your Application

## Core Idea
Beyond plain text streaming, the Vercel AI SDK lets the server stream whole React components (RSCs + `streamUI`/`createStreamableUI`), generate schema-validated structured data (`generateObject`/`streamObject` + Zod), and call custom tools/functions — the three techniques that turn a chatbot into a real generative UI application.

## Frameworks Introduced
- **RSC (React Server Component) generative UI**: components run and render exclusively server-side, streamed to the client as fully rendered React nodes rather than JSON-then-render.
  - When to use: when you want the AI response itself to *be* a rich UI element (a card, a table) rather than plain text the client has to interpret and render.
  - How: use `streamUI({ model, messages, text, tools })` — its `text` callback renders plain responses as components; its `tools` map lets the model trigger richer component generation.
- **Tool/function calling loop**: AI model → recognizes need for external data → emits structured tool-call request → SDK executes the registered tool (API/DB/calc) → result fed back to model → model produces final response.
  - When to use: any time the model needs current/external data (weather, DB lookups) it wasn't trained on.
  - How: define a Zod-typed `parameters` schema + an async `generate` function per tool, register under `tools: { toolName: {...} }` in `streamUI`.
- **Structured data generation pipeline**: UI action → SDK formats a structured prompt → AI provider returns raw response → SDK validates against a Zod schema → validated typed object returned to UI.
  - When to use: whenever AI output must be consumed programmatically (tables, forms, other services) rather than displayed as prose.
  - How: `generateObject({ model, schema, prompt })` for one-shot; `streamObject` for incremental structured output.

## Key Concepts
- **`streamUI`**: SDK function that sends input to an LLM and returns streamable React components (text or tool-driven).
- **`createStreamableUI`**: complements `streamUI` for updating a component's content incrementally after the fact (real-time in-place updates).
- **AI state vs. UI state**: AI state is the source-of-truth conversation history sent to the model; UI state is the rendered representation — the SDK's RSC model separates these explicitly.
- **Zod schema**: TypeScript-first schema/validation library used by the SDK as its default structured-output validator.
- **Tool manager**: the SDK's internal component that interprets a model's tool-call request, executes the real function, and feeds the result back to the model — keeping tool *execution* under app control while the model only decides *when* to call.

## Mental Models
- Six named techniques exist for getting structured output from a model (prompt engineering, output parsing, function calling, Zod schema validation, iterative reprompting, template-based generation, postprocessing) — in practice combine several; don't rely on prompt wording alone.
- Prefer **`generateObject`/Zod validation** over manual output parsing/regex whenever the consuming code needs guaranteed types — validation failure triggers automatic retry up to a limit, which raw parsing does not give you.
- Simple context injection (fetch data → paste into the prompt string) is valid *only* when the app always needs the same fixed prompt shape; reach for a proper tool-call once you need flexibility in when/whether the model requests the data.

## Anti-patterns
- **Manually parsing free-text LLM output with regex to extract structured fields**: fragile and untyped — the book explicitly ranks Zod schema validation above output parsing for reliability.
- **Executing "tool calls" without a manager boundary (letting the model directly trigger side effects)**: breaks the separation between AI reasoning and application-controlled execution — always route tool execution through the SDK/tool manager, never let raw model output directly call external systems.
- **Overusing prompt-injection for external data when flexibility is needed**: works only for a fixed prompt template; falls apart as soon as varied user intents are required — use registered tools instead.

## Code Examples
```jsx
// streamUI — server action streaming a chat bubble component
export async function streamComponent(input, history) {
  'use server';
  const result = await streamUI({
    model: openai('gpt-3.5-turbo'),
    messages: [...history, { role: 'user', content: input }],
    text: ({ content, done }) => (
      <ChatBubble role="assistant" text={content} className="mr-auto border-none" />
    ),
  });
  return { id: generateId(), role: 'assistant', display: result.value };
}
```
```js
// generateObject — schema-validated structured data with Zod
import { z } from 'zod';
import { google } from '@ai-sdk/google';
import { generateObject } from 'ai';

const ProductSchema = z.object({ name: z.string(), description: z.string(), price: z.number(), category: z.string() });

async function generateProductList(prompt) {
  'use server';
  const { object: { products } } = await generateObject({
    model: google('models/gemini-2.0-flash'),
    schema: z.object({ products: z.array(ProductSchema) }),
    prompt: `Generate a list of 5 products related to: ${prompt}. Provide name, description, price, and category for each.`,
  });
  return products;
}
```
```jsx
// streamUI with a custom tool — weather assistant
const result = await streamUI({
  model: supportedModel,
  system: `You are a helpful weather assistant. Use 'getWeather' when asked about weather in a city. Always interpret temperatures in Celsius.`,
  messages: [{ role: 'user', content: "What's the weather like in Paris today?" }],
  tools: {
    getWeather: {
      description: 'Get the current weather for a specific city',
      parameters: z.object({ city: z.string().describe('The name of the city') }),
      generate: async function* ({ city }) {
        yield <LoadingSpinner />;
        const weatherData = await fetchWeatherData(city);
        return <WeatherCard city={city} temperature={weatherData.temperature} condition={weatherData.condition} />;
      },
    },
  },
});
```
- **What it demonstrates**: the `generate` function is an async generator — `yield` an interim state (loading spinner) then `return` the final rendered component once the real data arrives.

## Reference Tables
| Technique for structured output | Mechanism | Reliability notes |
|---|---|---|
| Prompt engineering | Precise instructions asking for JSON shape | Weakest alone |
| Output parsing (regex/JSON.parse) | Extract from free text | Fragile, no guarantees |
| Function calling | Provider-native structured request | Provider-dependent |
| Zod schema validation | SDK validates + auto-retries | SDK's primary mechanism |
| Iterative reprompting | Multiple rounds to refine | Costs extra tokens |
| Template-based generation | Give the model an example to fill in | Improves consistency |
| Postprocessing | Clean data after the fact | Final safety net |

| Function | Mode | Returns |
|---|---|---|
| `generateText` / `generateObject` | One-shot | Full text / validated typed object |
| `streamText` / `streamObject` | Streaming | Async-iterable text / incrementally-validated objects |
| `streamUI` | Streaming, UI-producing | React components (text- or tool-driven) |
| `createStreamableUI` | Incremental in-place update | A streamable component you push updates into |

## Worked Example
The weather-assistant tool end-to-end: a `system` prompt tells the model it may call `getWeather`; the SDK registers `getWeather` with a Zod `{ city: string }` parameter schema and a `generate` async generator that first `yield`s a `<LoadingSpinner/>`, awaits `fetchWeatherData(city)`, then returns a `<WeatherCard/>` populated with real data. When the user asks "What's the weather like in Paris today?", the model decides to call the tool (model-dependent — not guaranteed), the SDK executes `fetchWeatherData`, and the client sees the spinner replaced by the finished card — all without a page reload or a hand-built polling mechanism. The example ships in `ch04/chat-rsc-tool-calls`.

## Key Takeaways
1. Use `streamUI`/`createStreamableUI` when the AI response itself should be a rendered component, not just text to be parsed by the client.
2. For any AI output your code will consume programmatically, use `generateObject`/`streamObject` with a Zod schema — don't hand-roll regex parsing.
3. Tool calling keeps side-effect execution under application control: the model only requests a tool by name/params; your registered function actually runs it.
4. A tool's `generate` function can be an async generator, letting you show interim state (loading) before the final result — critical for UX on slow external calls.
5. Simple prompt-injection of fetched data is a legitimate shortcut only for fixed-shape prompts; it doesn't scale to flexible, model-decided tool use.

## Connects To
- **Ch3**: extends `generateText`/`streamText`/`useChat` with UI-producing and structured-output variants.
- **Ch6**: LangChain.js (next chapter) offers an alternative, more agent-oriented way to compose tools and chains.
- **Ch7**: agentic patterns reuse this chapter's tool-calling loop as the foundation of the agentic loop.
