# Chapter 2: Building Your First Generative AI Web Application

## Core Idea
Build a minimal conversational AI app ("Astra") twice — first React + Express.js, then migrated to Next.js — to learn the request/response lifecycle and see concretely why an integrated full-stack framework beats a split frontend/backend for LLM apps.

## Frameworks Introduced
- **The 10-step chat request lifecycle**: UI render → user types+Enter → UI submits to backend → backend validates → backend calls AI model with context → model responds → backend verifies/cleans response → backend returns JSON → UI updates message list → user can follow up.
  - When to use: as the reference architecture for any synchronous chat feature before adding streaming/memory.
  - How: keep API keys and prompt construction server-side only; never expose them to the browser.
- **Custom hook construction method** (5 steps): define hook function (name starts with `use`) → identify state to manage internally vs. expose externally → create/inject helper functions → implement core functionality → return an object of state + handlers.
  - When to use: any time chat/form logic starts cluttering a component.
  - How: applied concretely to `useChatFormSubmit(getAssistantResponse)`, which owns `messages`, `inputValue`, `isLoading` and returns `handleSubmit`.

## Key Concepts
- **Persona**: the defined style/behavioral traits an AI app exhibits (here, "Astra").
- **File-based routing (Next.js)**: folder structure under `src/app` defines URL routes; a `page.js` in a folder makes it a public route; `[slug]` folders create dynamic segments.
- **Route group**: a folder wrapped in parentheses, e.g. `(chat)`, that organizes files without adding a URL segment.
- **`NEXT_PUBLIC_` prefix**: marks an env var as safe to embed in the client bundle — never use it for secrets.
- **Separation of concerns**: keeping state/API logic in hooks instead of components (illustrated by `useChatFormSubmit`).

## Mental Models
- Treat the backend as "a secure, reliable intermediary" — its whole job is keeping API keys server-side and shielding the browser from provider details.
- Migrating Express.js → Next.js is not a tooling swap, it's collapsing two deployable services into one cohesive framework — evaluate this trade whenever a project has a separate frontend+backend for one logical app.
- A flat folder structure (`components/`, `hooks/`) is fine for a single-feature MVP but explicitly flagged as not scaling — treat as a signal to introduce feature-based structure once a second feature appears.

## Anti-patterns
- **No conditional autoscroll**: naive autoscroll-to-bottom on every new message disrupts a user reading earlier history — should pause autoscroll when the user has scrolled up and offer a "Scroll to Latest" affordance instead.
- **Sending full un-streamed LLM responses**: waiting for the entire generation before showing anything causes perceived latency; flagged as a gap to fix with streaming in later chapters.
- **No conversation memory across reloads**: message history isn't persisted or resent, so context is lost — explicitly called out as a "still missing" characteristic of the v1 app.
- **Using `NEXT_PUBLIC_` for secrets**: embeds the value directly in the client bundle, visible to anyone inspecting page source.

## Code Examples
```jsx
// ChatPage.jsx — top-level chat component wiring hooks to UI
const ChatPage = () => {
  const { formRef, onKeyDown } = useEnterSubmit();
  const inputRef = useFocusOnSlashPress();
  const { messages, isLoading, handleSubmit, inputValue, setInputValue } =
    useChatFormSubmit(getAssistantResponse);
  ...
  return (
    <div>
      {messages.length === 0 && <h1>Hello, I'm Astra</h1>}
      {messages.length > 0 && <ChatList messages={messages} isLoading={isLoading} />}
      <form ref={formRef} onSubmit={handleSubmit}>
        <Textarea ref={inputRef} value={inputValue} onChange={onInputChange} onKeyDown={onKeyDown} />
      </form>
      <AutoScroll trackVisibility />
    </div>
  );
};
```
```js
// use-chat-form-submit.js — custom hook encapsulating chat state
function useChatFormSubmit(getAssistantResponse) {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState("");
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    const value = inputValue.trim();
    if (!value) return;
    setIsLoading(true);
    setInputValue("");
    const userMessage = { content: value, role: "user", id: generateUniqueId() };
    setMessages((cur) => [...cur, userMessage]);
    try {
      const { message } = await getAssistantResponse(value);
      setMessages((cur) => [...cur, message]);
    } catch (error) {
      console.error(error);
    } finally {
      setIsLoading(false);
    }
  };
  return { messages, isLoading, handleSubmit, inputValue, setInputValue };
}
```
```js
// server.openai.js — Express backend calling OpenAI chat.completions
class OpenAIHandler {
  constructor(openai) { this.openai = openai; }
  async handleRequest(req, res) {
    try {
      const { text } = req.body;
      const { data: completion } = await this.openai.chat.completions
        .create({
          messages: [
            { role: "system", content: "I'm happy to assist you..." },
            { role: "user", content: text },
          ],
          model: "gpt-3.5-turbo",
          stop: null,
          max_tokens: 150,
        })
        .withResponse();
      res.json({ message: { id: completion.id, role: "assistant", content: completion.choices[0].message.content } });
    } catch (e) {
      res.status(500).send("Internal server error");
    }
  }
}
```
- **What it demonstrates**: minimal secure backend pattern — env-var API key, JSON validation, single provider call, formatted response, generic error handling. `stop: null` + `max_tokens` are the two knobs controlling response length/shape.

## Reference Tables
| Approach | Frontend | Backend | Build tool | Notes |
|---|---|---|---|---|
| v1 | React | Express.js | Vite | Separate services, CORS needed, no SSR |
| v2 | React (Next.js) | Next.js route handlers | Next.js built-in | One deployable, file-based routing, `NEXT_PUBLIC_` for client-safe env vars |

## Worked Example
The chapter walks the same feature (Astra chat) through two full implementations. v1 (React+Express, `ch02/client-chat`): a flat `components/`+`hooks/` structure, `server.js` (or `server.openai.js`) exposing a single `POST /` handled by `OpenAIHandler`. v2 (`ch02/chat-client-next`): the same UI ported into Next.js's `src/app` file-based router, with a `(chat)` route group, a `layout.js` replacing `AppLayout.jsx`, and the Express POST handler replaced by a Next.js route handler calling `GoogleGenerativeAI` directly — collapsing two services into one. Both ship with Vitest unit tests (`npm run test:unit`) covering initial render, input typing, and loading state.

## Key Takeaways
1. Always route AI provider calls through a backend/server layer — never call the provider API directly from the browser (protects API keys).
2. Build custom hooks by first deciding what state stays internal vs. exposed, then wiring one helper function (e.g., `getAssistantResponse`) as an injected dependency — this keeps hooks testable and swappable across providers.
3. Next.js's file-based routing (`page.js` per folder, `[slug]` for dynamic segments, `(group)` for organization without URL impact) replaces manual Express route wiring.
4. Only prefix an env var `NEXT_PUBLIC_` if it is genuinely safe to expose in the browser bundle.
5. A working v1 chat app still lacks memory/context, streaming, and access control — treat these as the explicit backlog for subsequent chapters, not afterthoughts.

## Connects To
- **Ch3**: replaces the raw OpenAI/Gemini client calls here with the Vercel AI SDK's unified abstraction.
- **Ch4**: solves the "no memory/context across reloads" gap flagged here.
- **Ch8**: delivers the "robust error handling and retries" deferred in this chapter's backend discussion.
