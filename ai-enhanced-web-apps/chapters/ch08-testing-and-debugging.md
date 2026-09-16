# Chapter 8: Testing and Debugging Techniques

## Core Idea
Production AI apps need layered resilience (error tracking, token/rate-limit management, model fallbacks) plus a deliberate testing strategy that mocks only the parts of the system that don't need real model behavior — mocking LLM output entirely is both necessary (cost/determinism) and dangerous (hides real capability drift).

## Frameworks Introduced
- **Error-tracking + user-facing fallback pattern**: catch provider errors in the server action → classify via an `AIErrorTracker` (pattern-matched `ErrorType`/`ERROR_PATTERNS`) → generate a user-friendly message + request ID → render as an error `ChatBubble` with an optional retry action.
  - When to use: any production AI feature — providers change APIs/limits without warning, so generic try/catch isn't enough.
  - How: extend `ErrorType`/`ERROR_PATTERNS` as new provider quirks are discovered; tailor recovery copy per error class (network → "check your connection", auth → "log in again").
- **Token/rate-limit defense in depth**: `maxTokens` caps response size → rolling context window (keep last N messages) bounds prompt size → token count monitoring (js-tiktoken) tracks usage → `maxRetries` handles transient errors (429/500) → model fallback (`ai-fallback`) reroutes to a different provider entirely when one is down.
  - When to use: any app with meaningful traffic against a rate-limited provider API.
  - How: layer these — don't rely on retries alone; combine proactive limits (maxTokens, rolling context) with reactive recovery (retries, fallback).
- **What-to-mock decision rule**: mock simple/predictable logic (input sanitization, response processing, error handling, flow control); do NOT mock complex/evolving capabilities (sentiment analysis quality, model reliability, performance characteristics).
  - When to use: designing any test suite touching an LLM.
  - How: ask "am I testing my system's logic, or the model's evolving capability?" — mock only the former.

## Key Concepts
- **`AIErrorTracker`**: custom class encapsulating known error types + user-facing messages, keyed by error patterns.
- **Rolling conversation context**: `while (messageHistory.length > 10) messageHistory.shift()` — bounds prompt size by dropping oldest messages.
- **`maxRetries`**: SDK parameter on `streamText`/`generateText`/`streamUI` for auto-retrying transient failures (esp. HTTP 429, which is time-based and retryable, unlike a quota exhaustion error).
- **`ai-fallback` / `createFallback`**: library wrapping multiple provider models behind one `LanguageModelV1`-compatible interface, auto-switching on error with configurable `shouldRetryThisError` and `modelResetInterval`.
- **`MockLanguageModelV1`**: Vercel AI SDK's official test double for `generateText`/`generateObject`, letting you override `doGenerate` to return controlled output with zero live API calls.
- **`simulateReadableStream`**: test helper mimicking streamed chunk delivery, paired with `MockLanguageModelV1`'s `doStream` to test `streamText`/`streamObject` consumers.

## Mental Models
- Treat mocking depth as a trade-off: an overly detailed mock response (full provider response shape with `logprobs`, `model_version`, etc.) creates brittle tests coupled to implementation details; a minimal mock (`{ text, isComplete }`) tests only what your app logic actually needs — prefer the latter.
- A 429 (rate limit) is fundamentally different from a quota-exhaustion error even though both look like "the request failed" — 429 is time-based and safe to retry; quota errors are persistent and retrying wastes calls.
- Model fallback is not just an outage mitigation — because it sits behind the same `LanguageModelV1` interface established in Ch3, adding a fallback costs zero changes to the rest of the app.

## Anti-patterns
- **Retrying persistent errors (quota exhaustion) the same way as transient ones (429/500)**: wastes calls and delays user-facing failure — distinguish retryable vs. non-retryable error classes explicitly (`shouldRetryThisError`).
- **Mocking LLM output for tasks that depend on evolving model capability** (sentiment quality, reasoning correctness): the mock silently goes stale as the real model improves/changes, giving false test confidence.
- **Encoding full raw provider response shapes into mocks** (`logprobs`, `finish_reason`, `model_version`, etc.): couples tests to a specific provider's response format, breaking the moment that format changes or a provider is swapped.
- **Not implementing user-facing error recovery**: silent failures or raw error dumps in the chat UI erode trust — always classify the error and offer a next step (retry button, "try again later", "log in again").

## Code Examples
```js
// Error tracking + user-facing fallback in a server action
import { AIErrorTracker } from '@/lib/error-tracking';
export async function continueConversation(input, provider, model) {
  try { /* ... */ }
  catch (error) {
    const errorData = await AIErrorTracker.trackError(error, { provider, model, input });
    const userError = AIErrorTracker.createUserFacingError(errorData);
    return {
      id: userError.requestId,
      role: 'assistant',
      display: <ChatBubble role="error" text={`${userError.message} (Request ID: ${userError.requestId})`} />,
    };
  }
}
```
```js
// Layered token/rate-limit defenses
const response = await streamText({ model, prompt, maxTokens: 100, maxRetries: 3 });
while (messageHistory.length > 10) messageHistory.shift(); // rolling context
```
```js
// Model fallback across providers
import { createFallback } from 'ai-fallback';
const supportedModel = createFallback({
  models: [
    createGoogleGenerativeAI({ apiKey: googleAPIKey })('models/gemini-2.0-flash'),
    createOpenAI({ apiKey: openAPIKey })('gpt-3.5-turbo'),
  ],
  onError: (error, modelId) => console.error(`Error with model ${modelId}:`, error),
  modelResetInterval: 60000,
  shouldRetryThisError: (error) => [429, 500].includes(error.statusCode),
});
```
```js
// Minimal, robust mocking with MockLanguageModelV1
import { generateText } from 'ai';
import { MockLanguageModelV1 } from 'ai/test';

test('should return predefined text from mock model', async () => {
  const result = await generateText({
    model: new MockLanguageModelV1({
      doGenerate: async () => ({
        rawCall: { rawPrompt: null, rawSettings: {} },
        finishReason: 'stop',
        usage: { promptTokens: 10, completionTokens: 20 },
        text: 'Hello, world!',
      }),
    }),
    prompt: 'Hello, test!',
  });
  expect(result.text).toBe('Hello, world!');
});
```

## Reference Tables
| Mechanism | Problem solved | Trigger condition |
|---|---|---|
| `maxTokens` | Bound response size | Every request (proactive) |
| Rolling context window | Bound prompt/history size | Every request (proactive) |
| Token count monitoring (js-tiktoken) | Visibility into usage | Continuous |
| `maxRetries` | Transient failures | 429, temporary server errors |
| `ai-fallback` model fallback | Provider outage | Persistent provider failure |
| `AIErrorTracker` + user-facing message | User trust/recovery | Any caught error |

| What to mock | What NOT to mock |
|---|---|
| Input sanitization | Sentiment analysis quality |
| Response processing/formatting | Model reasoning correctness |
| Error handling / flow control | Performance characteristics |
| Basic control flow branches | Model reliability under load |

## Worked Example
Testing a streaming chat response without live API calls: `MockLanguageModelV1`'s `doStream` returns a `simulateReadableStream` with chunked `{ type: 'text-delta', textDelta: '...' }` events; the test asserts the consumer (`useChat`/`streamText` caller) correctly assembles and displays incremental chunks — validating the app's streaming-consumption logic in isolation from any real provider, at zero cost and fully deterministically. Run via `npm run test:vercel:ai` in `ch08/chat-testing`.

## Key Takeaways
1. Classify and handle AI provider errors explicitly (regional restrictions, capability mismatches, silent empty responses) rather than relying on generic try/catch — surface actionable, user-facing recovery messages with a request ID.
2. Layer token/rate-limit defenses: `maxTokens` + rolling context (proactive) with `maxRetries` + provider fallback (reactive) — don't rely on just one.
3. Distinguish retryable (429, transient 5xx) from non-retryable (quota exhaustion) errors before configuring retry logic.
4. Mock only what tests your own system's logic (sanitization, formatting, error handling); never mock away the parts of a test meant to validate actual model capability.
5. Keep mock response shapes minimal and focused on what your app consumes — avoid encoding full provider response schemas into test fixtures.
6. Use the SDK's own test helpers (`MockLanguageModelV1`, `simulateReadableStream`) rather than hand-rolled mocks — they track the SDK's real interface contract.

## Connects To
- **Ch3**: `ai-fallback`'s `LanguageModelV1` interface directly reuses the language model specification (abstract factory) established there.
- **Ch6/Ch7**: LangChain chain latency/propagation concerns flagged there are addressed here with monitoring (Sentry/Elastic Stack) and error-handling patterns.
- **Ch9**: deployment and security chapter builds on this chapter's reliability foundation for production readiness.
