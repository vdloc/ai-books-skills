# Chapter 10: Building an AI Interview Assistant (Project Walk-Through)

## Core Idea
A full production-shaped project — Next.js + Vercel AI SDK + Upstash Redis + Clerk.js — that combines everything from prior chapters (auth, streaming chat, rate limiting, security middleware) into a complete AI interview-practice app, using Redis as a lightweight session/history store instead of a relational database.

## Frameworks Introduced
- **Redis-as-session-store pattern**: `user:sessions:{userId}` is a Redis *set* of session IDs (uniqueness guaranteed); `session:{sessionId}` is a Redis *hash* of session details; sessions are fetched via `SMEMBERS` then `HGETALL` per ID, sorted client-side by `createdAt`.
  - When to use: intermediate data needs where a full relational DB is unnecessary — fast reads/writes, simple key structure, easy to scale to Postgres later if relational features (transactions, joins) become necessary.
  - How: design Redis keys explicitly around access patterns (list-all-sessions-for-user, get-one-session) rather than normalizing data as you would in SQL.
- **Feature flag via env var**: `NEXT_PUBLIC_FEATURE_TTS_ENABLED` toggles an entire feature (text-to-speech) client-and-server-wide.
  - When to use: simple, low-frequency on/off toggles during early development.
  - How: explicitly flagged as a stopgap — the book recommends a dedicated feature-management service once you need per-segment rollout control.
- **Cached derived-output pattern**: expensive LLM-generated feedback is cached in Redis under `feedback:${sessionId}` so revisiting a session doesn't regenerate it.
  - When to use: any expensive, deterministic-per-input LLM call whose result doesn't need to change between views (e.g., a report, a summary, feedback).

## Key Concepts
- **Custom vs. structured interview type**: two user-facing entry paths — free-form job description input vs. a more guided/templated flow — both feeding the same underlying chat pipeline.
- **`ChatThread` component**: the core chat UI plus TTS enable/disable controls.
- **`InterviewSidebar` + `fetchInterviewSessions`**: lists past sessions per user, fetched in parallel via `Promise.all` over Redis `HGETALL` calls, then sorted descending by creation time.
- **`/api/tts` route**: a dedicated API route (not a server action) streaming Google Cloud Text-to-Speech audio back to the client — chosen deliberately to demonstrate API-route flexibility vs. RSC server actions.

## Mental Models
- Treat this chapter as an integration exercise: nothing here is a new primitive — auth (Ch9), streaming chat (Ch3/4), rate limiting/security middleware (Ch9), and Redis-backed state are combined into one coherent app. When evaluating your own project's architecture, check it against this same checklist (auth boundary, state store, security middleware, cost-control caching).
- A key-value store like Redis can substitute for a relational DB when the data model is simple (session → set of messages, user → set of sessions) — but the book is explicit that this requires "ad hoc" application-level relationship maintenance that SQL would give you natively (transactions, joins); don't over-extend this pattern to genuinely relational data.
- Caching LLM output (feedback) by a stable key (`sessionId`) is a direct, cheap latency/cost win whenever the output doesn't need to be regenerated per view.

## Anti-patterns
- **Using Redis for genuinely relational, transactional data**: the chapter explicitly flags that Redis "is not inherently designed for complex relational models" — reach for SQL/Postgres once you need transactions or multi-entity joins.
- **Using an env-var feature flag for anything beyond simple/early-stage toggles**: lacks per-user-segment control and requires a redeploy to change — fine short-term, but the book calls out a dedicated feature-management service as the production-grade solution.
- **Regenerating expensive LLM feedback on every page view**: wastes tokens/cost and adds latency — cache by session ID instead.
- **Building session/state management without checking existing chapter patterns first**: this project reuses Ch9's security middleware and Ch3/4's chat streaming rather than inventing new mechanisms — treat prior-chapter patterns as the default toolkit for new features.

## Code Examples
```js
// Fetching all past interview sessions for a user, sorted newest-first
export async function fetchInterviewSessions(userId) {
  const sessionIds = await redis.smembers(`user:sessions:${userId}`);
  const sessions = await Promise.all(
    sessionIds.map(async (sessionId) => {
      const session = await redis.hgetall(`session:${sessionId}`);
      return { ...session, id: sessionId };
    }),
  );
  sessions.sort((a, b) => b.createdAt - a.createdAt);
  return sessions;
}
```
```js
// Feedback generation prompt with Redis caching (pattern)
const prompt = `Please provide comprehensive feedback for this interview...`;
// cached under: feedback:${sessionId} — checked before regenerating
```

## Reference Tables
| Component | Responsibility | Backing tech |
|---|---|---|
| `ChatThread` | Core chat UI + TTS controls | Vercel AI SDK (`useChat`) |
| `InterviewSidebar` | List past sessions | Redis sets/hashes |
| Feedback page | Generate + cache session feedback | LLM + Redis cache |
| `/api/tts` | Stream synthesized audio | Google Cloud Text-to-Speech |
| `middleware.js` | Auth, CORS, rate limit, security headers | Clerk.js + Upstash Redis |

| Redis key pattern | Type | Purpose |
|---|---|---|
| `user:sessions:{userId}` | Set | Unique session IDs per user |
| `session:{sessionId}` | Hash | Session metadata (job type, date, difficulty) |
| `feedback:{sessionId}` | String/cache | Cached LLM-generated feedback |

## Worked Example
A user logs in via Clerk → selects "Custom interview" and submits a job description → the app streams interview questions from Gemini via the Vercel AI SDK, storing each session under a new `session:{sessionId}` hash and adding that ID to `user:sessions:{userId}`. The sidebar (`InterviewSidebar`) calls `fetchInterviewSessions` to show past sessions, letting the user reopen and review any prior thread. When the user requests feedback, the app checks `feedback:{sessionId}` in Redis first; on a cache miss, it sends the full thread to the LLM with a feedback-generation prompt, caches the result, and displays it — on any subsequent visit, the cached version is served instantly with zero additional LLM cost.

## Key Takeaways
1. This project is an integration checklist: auth, streaming chat, security middleware, and state management should all be present in any production AI chat app — use this chapter's stack as a reference architecture.
2. Design Redis key structure around access patterns (list sessions, get one session) rather than trying to normalize data relationally.
3. Cache expensive, stable LLM outputs (like generated feedback) by a natural key (session ID) to cut cost and latency on repeat views.
4. Env-var feature flags are fine for early development but don't scale to per-segment control — plan to graduate to a real feature-flag service.
5. Choose API routes vs. server actions deliberately based on the capability needed (e.g., streaming binary audio favored an API route here) rather than defaulting to one pattern everywhere.

## Connects To
- **Ch3/Ch4**: reuses the Vercel AI SDK streaming chat foundation directly.
- **Ch9**: reuses the full security middleware stack (auth, CORS, rate limiting, security headers) verbatim.
- **Ch11**: the next hands-on project (AI RAG agent) follows the same integration-project structure.
