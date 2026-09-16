# Chapter 9: Deployment and Security

## Core Idea
Production AI web apps need a layered security pipeline (threat model → input validation → composable middleware → auth/rate-limit/quota → secrets management → PII anonymization) built before deployment, then a deployment strategy chosen by expected traffic tier rather than one-size-fits-all defaults.

## Frameworks Introduced
- **Threat modeling for AI web apps**: identify public endpoints → identify user input points → evaluate data sensitivity — done *before* deciding what to validate.
  - When to use: at the start of hardening any app, not as an afterthought.
  - How: enumerate every POST endpoint (RSC server actions post to `/`, API routes post to `/api`) and classify what data each accepts.
- **Server vs. client validation split**: client validation is UX-only (immediate feedback); server validation is the actual security boundary — always assume client-submitted data is tampered with.
  - When to use: every user input path, always both, never client-only.
  - How: mirror the same constraint on both sides (e.g., `maxLength={1000}` in the UI *and* a Zod `z.string().max(1000)` on the server) — the server copy is the one that matters for security.
- **Composable middleware pipeline**: `composeMiddleware([handleCORS, rateLimit, authenticate, securityHeaders])` — an ordered chain where each middleware can short-circuit (return a response) or halt (`continue: false`), all wrapped in shared error handling.
  - When to use: any Next.js app needing more than one cross-cutting security concern (CORS + rate limit + auth + headers).
  - How: position this at the very front of the request pipeline — on Vercel, middleware runs on the edge, closest to the user, before any app logic executes.
- **Defense-in-depth for abuse/cost control**: rate limiting (per-IP, sliding window) → authenticated message quota (per-user, per-day, Redis-backed) → registration friction (invite-only, CAPTCHA, business email) to stop quota-limit evasion via disposable accounts.
  - When to use: any app exposing paid LLM API calls to end users.

## Key Concepts
- **Sliding window rate limiter** (`Ratelimit.slidingWindow(5, "10 s")`): allows N requests per rolling time window per identifier (IP or user ID), backed by Redis (Upstash in the book's examples).
- **Message quota**: a separate, coarser control from rate limiting — caps total daily messages per authenticated user (e.g., 10/day), tracked via a Redis key like `message_count:{userId}:{date}`.
- **PII redaction/anonymization**: replacing sensitive data (names, emails, phone numbers, SSNs) in user input with placeholders (`PERSON_NAME`, `EMAIL_ADDRESS`) *before* it reaches the LLM or is logged — via `redact-pii` (simple) or `@google-cloud/dlp` (production-grade, non-English support).
- **`NEXT_PUBLIC_` vs. server-only env vars**: reiterated from Ch2 — never expose API keys to the client bundle.
- **Application-level vs. user-provided API keys**: app-level keys (your own account, cost centralized on you) vs. user-provided keys (each user supplies their own, cost shifted to them) — a deployment/business-model decision, not just a technical one.
- **Impersonation logging**: when support staff access a user's chat history, those actions must be strictly logged, scoped, and time-limited (principle of least privilege applied to internal access, not just external).

## Mental Models
- Treat the middleware chain order as security-critical: CORS/rate-limit/auth should run *before* any expensive logic (including hitting the LLM) so abusive requests are rejected as cheaply as possible.
- A rate limiter alone doesn't stop determined abuse — pair it with a coarser quota (daily message cap) and, if still exploited via disposable accounts, add registration friction (invite codes, CAPTCHA, business email requirement).
- PII protection is a data-flow problem, not a point fix: redact at the input boundary so the LLM *and* your logs never see raw PII — debug by checking each stage (input → redaction → LLM → storage → UI) independently when something leaks.
- Deployment choice (Vercel vs. Docker/Kubernetes vs. self-hosting) trades operational simplicity against control — pick based on current traffic tier, not aspirational scale.

## Anti-patterns
- **Relying on client-side validation alone**: trivially bypassed by a user manipulating client code or calling the API directly — server validation is the only real boundary.
- **Incrementing a Redis quota counter even for rejected/rate-limited requests**: lets an attacker exhaust another user's or their own future quota via rapid rejected requests — always place quota checks *after* rate limiting in the pipeline.
- **Sending raw user input (containing PII) directly to an LLM or into logs**: risks data leakage to a third-party provider and compliance violations — always redact before either destination.
- **Assuming a message-quota-per-account is sufficient abuse prevention**: disposable email signups can trivially create new accounts to reset quotas — add invitation-only registration or CAPTCHA if this becomes a problem.
- **Choosing a deployment platform based on aspirational scale rather than current tier**: over-engineering infra (Kubernetes) for a <10K req/day app adds operational burden with no corresponding benefit; the book explicitly recommends Vercel as the default starting point.

## Code Examples
```js
// Server-side input validation with Zod — always the source of truth
import { z } from 'zod';
const promptSchema = z.object({ prompt: z.string().min(1).max(1000) });
export default async function POST(req, res) {
  try {
    const validatedData = promptSchema.parse(req.body);
    const response = await process(validatedData.prompt);
    res.status(200).json({ response });
  } catch (error) {
    return res.status(400).json({ error: error.errors });
  }
}
```
```js
// Composable middleware chain
const composeMiddleware = (middlewares) => async (request) => {
  let response = NextResponse.next();
  for (const middleware of middlewares) {
    try {
      const result = await middleware(request, response);
      if (result.response) return result.response;
      if (result.continue === false) break;
    } catch (error) {
      return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 });
    }
  }
  return response;
};
const middlewareChain = composeMiddleware([handleCORS, rateLimit, authenticate, securityHeaders]);
export async function middleware(request) { return await middlewareChain(request); }
export const config = { matcher: '/api/:path*' };
```
```js
// Sliding-window rate limiter (Upstash Redis)
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
const redis = new Redis({ url: process.env.UPSTASH_REDIS_REST_URL, token: process.env.UPSTASH_REDIS_REST_TOKEN });
const ratelimit = new Ratelimit({ redis, limiter: Ratelimit.slidingWindow(5, "10 s") });
const rateLimit = async (request) => {
  const { success } = await ratelimit.limit(request.ip || '127.0.0.1');
  if (!success) return { response: NextResponse.json({ message: "Too many requests" }, { status: 429 }), continue: false };
  return { continue: true };
};
```
```js
// Daily per-user message quota (Redis)
const checkMessageQuota = async (userId) => {
  const key = `message_count:${userId}:${new Date().toISOString().split('T')[0]}`;
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, 24 * 60 * 60);
  return count <= 10;
};
```
```js
// PII redaction before the prompt reaches the LLM
import { SyncRedactor } from 'redact-pii';
const redactor = new SyncRedactor();
function anonymizeText(text) { return redactor.redact(text); }
// ... anonymizedInput = anonymizeText(validatedData.text); // send this to the LLM, not raw input
```

## Reference Tables
| Security layer | Purpose | Position in pipeline |
|---|---|---|
| Input validation (Zod) | Reject malformed/oversized input | At the API boundary |
| CORS / security headers | Block cross-origin abuse | Middleware, first |
| Rate limiting (sliding window) | Cap requests per IP/user per window | Middleware, early |
| Authentication (Clerk.js) | Verify identity | Middleware, after rate limit |
| Message quota (daily) | Cap total LLM cost per user | After auth, before LLM call |
| PII redaction | Prevent sensitive data reaching LLM/logs | At input boundary, before storage/LLM call |

| Deployment option | Best for | Trade-off |
|---|---|---|
| Vercel (dedicated) | Getting started, Next.js-native | Less control at extreme scale |
| Docker + Kubernetes | Advanced, consistent multi-env | Higher operational complexity |
| Self-hosting | Maximum control/customization | Highest maintenance burden |

| Traffic tier | Requests/day | Typical needs |
|---|---|---|
| Tier 1 (small) | <10,000 | Minimal monitoring/scaling infra |
| Tier 2/3 (larger) | Higher | Progressively more robust monitoring, scaling, backup (book details further tiers) |

## Worked Example
End-to-end request lifecycle for a hardened chat endpoint: request hits edge middleware → `handleCORS` checks origin → `rateLimit` checks the sliding-window Redis counter (reject with 429 if exceeded) → `authenticate` verifies the Clerk session → request reaches the API route → Zod validates prompt length/shape → `checkMessageQuota` verifies the user hasn't exceeded 10 messages/day (reject with 429 + friendly message if so) → `anonymizeText` redacts PII from the validated prompt → the redacted prompt (never the raw one) is sent to the LLM → response returned and displayed. Each stage is independently testable and independently debuggable (e.g., "logs show input is redacted but UI implies LLM saw PII" → bug is in UI display code, not anonymization).

## Key Takeaways
1. Build a threat model (public endpoints, input points, data sensitivity) before deciding what to validate — validation without a threat model is guesswork.
2. Server validation is the only real security boundary; client validation is UX polish only.
3. Order your middleware chain deliberately: cheap rejects (CORS, rate limit) before expensive ones (auth, LLM calls) — and always place quota checks *after* rate limiting to avoid quota-counter abuse.
4. Redact PII at the input boundary, before it reaches the LLM or gets logged — verify by checking each pipeline stage independently when a leak is suspected.
5. A rate limiter and a quota system solve different problems (burst abuse vs. total cost control) — use both, and add registration friction if disposable accounts start evading quotas.
6. Choose deployment infrastructure (Vercel / Docker+K8s / self-host) based on your current traffic tier, not hypothetical future scale.

## Connects To
- **Ch2**: reuses the `NEXT_PUBLIC_` env var boundary established there for secrets management.
- **Ch3**: extends the "backend as secure intermediary" principle first introduced when wiring the Vercel AI SDK.
- **Ch8**: builds directly on the error-tracking and rate-limit/retry mechanisms from the testing/debugging chapter — this chapter adds the abuse-prevention layer in front of them.
- **Ch1**: operationalizes the PII/GDPR/CCPA compliance concerns raised abstractly there.
