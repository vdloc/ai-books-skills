# Chapter 3: Connecting AI Models with the Vercel AI SDK

## Core Idea
The Vercel AI SDK provides a provider-agnostic abstraction (`generateText`/`streamText` + a language-model specification) plus React hooks (`useChat`/`useCompletion`) that collapse per-provider API differences, streaming plumbing, and multimodal (vision) input handling into a small, swappable surface.

## Frameworks Introduced
- **Language model specification (abstract factory pattern)**: `generateText`/`streamText` are abstract factories; each provider package (`@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google`, …) is a concrete factory returning a model instance that implements a common `generate`/`stream` interface.
  - When to use: whenever an app must support more than one LLM provider or expects to swap providers later.
  - How: call `generateText({ model: provider('model-name'), prompt })` — switching providers is a one-line `model` change, no interface change.
- **Incremental integration strategy**: replace one API call at a time (route handler → text gen → streaming → multi-provider → multimodal) while keeping the frontend contract stable, testing after every step.
  - When to use: introducing any new framework/SDK into an existing production app.
  - How: change the backend route handler first, verify UI unaffected, then move to the next capability.

## Key Concepts
- **`generateText`**: non-streaming text generation call; takes `model` + `messages` or `system`+`prompt`, returns full `{ text }`.
- **`streamText`**: streaming counterpart; returns an async-iterable `textStream` consumed via `for await...of`.
- **Async iterable / async generator (`async function*` + `yield`)**: JS primitive underlying streaming — a generator function pauses on `await`, yields chunks, and the consumer drains it with `for await...of`.
- **`useChat` hook**: manages a full conversation (message list) + streaming + input state; drop-in replacement for a hand-rolled `useChatFormSubmit`.
- **`useCompletion` hook**: single-prompt-in, single-completion-out variant (no message history).
- **Multimodal AI**: models (GPT-4o, Gemini) that accept image + text input and return text, via a 7-step flow: upload → encode to binary/base64 → API request → route to model → visual understanding → combine with text → response.

## Mental Models
- Treat provider SDKs as interchangeable **concrete products** behind one interface — never call `openai.chat.completions` or a provider client directly in app code if more than one provider is in play; go through `generateText`/`streamText`.
- Streaming exists because LLM throughput is fundamentally slow (~21 tokens/sec for GPT-4 per the book's benchmark) — it's a UX mitigation for a hard latency ceiling, not an optional nicety.
- When adding multimodal input, budget for image constraints as a design input up front: one image per prompt (not several), provider file-size ceilings (20MB for OpenAI), and image quality (contrast/lighting) — these are real failure modes, not edge cases.

## Anti-patterns
- **Hardcoding a single provider's client directly in app logic**: creates duplicated, provider-specific code paths (shown explicitly with parallel OpenAI + Cohere client calls) that don't share an interface — makes swapping/adding providers expensive.
- **Sending multiple images in one prompt "for context"**: increases complexity/latency and degrades result quality — prefer separate prompts per image.
- **Skipping image-quality preprocessing**: low contrast, poor lighting, or noisy images measurably reduce vision-model accuracy.

## Code Examples
```js
// generateText — provider-agnostic text generation
import { generateText } from 'ai';
const { text } = await generateText({
  model,
  messages: [
    { role: 'system', content: "I'm happy to assist you..." },
    { role: 'user', content: text },
  ],
});
```
```js
// streamText — async-iterable streaming
import { createGoogleGenerativeAI } from '@ai-sdk/google';
import { streamText } from 'ai';
const model = createGoogleGenerativeAI({ apiKey: process.env.GEMINI_API_KEY || '' });
const { textStream } = await streamText({
  model: model('gemini-2.0-flash'),
  prompt: 'Generate 400 words of random content for me please',
});
for await (const textPart of textStream) {
  process.stdout.write(textPart);
}
```
```jsx
// useChat — streaming conversational UI in one hook
import { useChat } from 'ai/react';
export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat({ api: '/api' });
  return (
    <div>
      <Messages data={messages} />
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} placeholder="Ask me something..." />
      </form>
    </div>
  );
}
```
```js
// Provider-swap via the language model specification — one-line change
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';
const a = await generateText({ model: openai('gpt-3.5-turbo'), prompt: 'What is 7+7 and why?' });
const b = await generateText({ model: anthropic('claude-v1'), prompt: 'What is 7+7 and why?' });
```

## Reference Tables
| Provider | Package |
|---|---|
| OpenAI | `@ai-sdk/openai` |
| Anthropic | `@ai-sdk/anthropic` |
| Google generative AI | `@ai-sdk/google` |
| Google Vertex | `@ai-sdk/google-vertex` |
| Mistral | `@ai-sdk/mistral` |

| streamText helper | Purpose |
|---|---|
| `onFinish` callback | fires with full text/tool calls/usage when generation completes |
| `toAIStreamResponse()`, `toTextStreamResponse()`, `pipeTextStreamToResponse()` | integrate with SDK UI components / raw HTTP responses |
| `result.text`, `result.toolCalls`, `result.usage` (promises) | resolve once all data is available |

## Worked Example
Astra AI is migrated in three concrete steps, each independently testable: (1) `npm i ai @ai-sdk/google`, replace the raw Gemini/OpenAI client call in the Next.js route handler with `generateText({ model: model('gemini-2.0-flash'), messages })` — UI untouched, behavior unchanged; (2) swap `generateText` for `streamText` in the backend and swap the custom `useChatFormSubmit` hook for `useChat({ api: '/api' })` in the frontend — now responses render incrementally; (3) add a second provider (e.g., Anthropic) by importing `anthropic` from `@ai-sdk/anthropic` and changing only the `model` argument — proving the abstraction actually decouples client code from provider. A fourth extension adds vision: the client base64-encodes an uploaded image, sends it alongside the text prompt, and the backend passes it through Gemini's vision-capable model to get an image-aware response.

## Key Takeaways
1. Route all LLM calls through `generateText`/`streamText` with a provider instance as the `model` argument — never call a provider's raw client SDK from app code once more than one provider might be needed.
2. Streaming is a UX necessity, not a feature toggle, given real-world LLM token throughput (~21 tok/s for GPT-4) — reach for `streamText` + `useChat` for any user-facing conversational UI.
3. `useChat` (multi-turn) vs. `useCompletion` (single prompt) is the correct dimension to choose on: pick based on whether conversation history matters.
4. Multimodal (vision) input has real constraints: one image per prompt, provider file-size limits, and image-quality sensitivity — design around these rather than discovering them in production.
5. The abstract factory pattern (language model specification) is *why* switching providers is a one-line change — understanding this pattern lets you extend to custom/community providers (Ollama, LlamaCpp) the same way.

## Connects To
- **Ch1**: cashes in the "Vercel AI SDK as the cherry on top" claim with concrete code.
- **Ch2**: directly replaces the hand-rolled Express/Next.js backend and `useChatFormSubmit` hook built there.
- **Ch4**: builds conversation memory/state on top of the `useChat`-based streaming foundation established here.
- **Ch6**: LangChain.js later provides a similar (but more agent/RAG-oriented) abstraction layer.
