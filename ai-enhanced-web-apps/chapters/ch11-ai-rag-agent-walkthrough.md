# Chapter 11: Building an AI RAG Agent (Project Walk-Through)

## Core Idea
A multi-tenant knowledge-base RAG application (create/manage knowledge bases, upload PDFs/DOCX, chat grounded in their content) that surfaces the real production trade-offs a single-user chat app doesn't: shared-vs-dedicated vector storage per tenant, document-upload security, and API/URL design that minimizes information exposure.

## Frameworks Introduced
- **Namespaced shared vector store**: one Upstash Vector database for all users, with strict namespacing/metadata filtering by knowledge-base and user ID enforced at query time, instead of provisioning a dedicated vector store per user/tenant.
  - When to use: default choice — optimizes cost/ops complexity while maintaining logical isolation, appropriate unless regulatory/compliance requirements demand physical separation.
  - How: every vector search/storage operation must carry and enforce the owning user/knowledge-base ID as a filter — treat this as a security control, not an implementation detail.
- **Document-upload security layering**: client+server file-type/size validation → server-side parsing/chunking with error handling → (recommended next steps) rate limits/upload quotas, malware scanning, background processing, encrypted storage.
  - When to use: any feature accepting user-uploaded files that get parsed and processed server-side.
  - How: treat "validation passed" as necessary but not sufficient — resource-intensive parsing on untrusted files is itself an attack surface (DoS via large/malicious files).
- **Minimal-information-exposure API/URL design**: hierarchical, scoped routes (`/api/knowledgebase/[knowledgebaseId]`, `/api/chat/[knowledgebaseId]`) + authentication gating on every access + UUIDs (not sequential/guessable IDs) for resource identifiers.
  - When to use: any multi-tenant API where resource IDs appear in URLs.
  - How: never use predictable/sequential IDs for user-facing resource identifiers; always authorize at the resource level, not just at the route level.

## Key Concepts
- **Knowledge base**: a user-created container (name + optional description) grouping one or more uploaded documents that a chat session can be scoped to query.
- **`UpstashVectorStore`**: LangChain.js's community retriever integration for Upstash Vector, used exactly like any other retriever (`asRetriever()`) — demonstrating LangChain's retriever abstraction is genuinely swappable.
- **Vector index dimensionality**: the Upstash Vector index is created with 768 output dimensions specifically to match the Google AI embedding model's output size — embedding model and vector store dimensionality must match.
- **Monolith vs. microservices**: explicitly framed as a scaling decision, not a correctness one — the book's example app is intentionally a single unified codebase (MVP, educational) and flags decomposition into dedicated services as the next step for production scale.

## Mental Models
- Treat "shared vs. dedicated data store per tenant" as a spectrum trading cost/ops simplicity (shared + filtering) against isolation guarantees (dedicated instances) — pick based on actual compliance requirements, not default caution or default convenience.
- Resource-intensive user-triggered operations (document parsing/vectorization) are both a *performance* problem (should be backgrounded) and a *security* problem (abuse surface) simultaneously — solve both with the same lever: move heavy work off the request path into background workers.
- API design decisions (URL structure, ID scheme) are security decisions, not just REST style choices — a predictable ID scheme is itself a vulnerability class (IDOR-style enumeration).

## Anti-patterns
- **Using sequential/predictable IDs for user-facing resources** (knowledge bases, documents): enables enumeration attacks — use UUIDs.
- **Processing uploaded documents synchronously in the request path**: blocks responsiveness and creates a DoS vector via large/many uploads — offload to background workers/serverless functions.
- **Treating client+server validation as sufficient for uploaded files**: doesn't address malware or storage-at-rest risk — add malware scanning and encryption (or discard after processing) for production.
- **Skipping metadata/namespace filtering on every vector query**: without enforcing user/knowledge-base scoping at the query layer, a shared vector store becomes a cross-tenant data leak vector.
- **Building a full microservices architecture for an MVP/educational-scope project**: the book explicitly chose a monolith deliberately for this project's scope — don't over-architect before scale actually demands it.

## Code Examples
```js
// Knowledge base creation API route
import { NextResponse } from 'next/server';
import { createKnowledgeBase, getAllKnowledgeBases } from '@/lib/database';

export async function POST(request) {
  try {
    const { name, description } = await request.json();
    if (!name) return NextResponse.json({ error: 'Name is required' }, { status: 400 });
    const knowledgeBase = await createKnowledgeBase({ name, description: description || '' });
    return NextResponse.json(knowledgeBase);
  } catch (error) {
    return NextResponse.json({ error: 'Failed to create knowledge base' }, { status: 500 });
  }
}
```

## Reference Tables
| API route | Scope | Purpose |
|---|---|---|
| `/api/knowledgebase` | Global (per user) | CRUD for knowledge bases |
| `/api/knowledgebase/[knowledgebaseId]/document/[id]` | Per document | Upload, fetch, delete a document |
| `/api/upload` | Upload pipeline | Receives file, triggers parsing/vectorization |
| `/api/chat/[knowledgebaseId]` | Per knowledge base | Conversational retrieval + generation |

| Vector store isolation model | Isolation | Cost/ops |
|---|---|---|
| Shared DB + namespacing/metadata filtering (chosen) | Logical | Low overhead |
| Dedicated instance per user/tenant | Physical | High overhead, required for strict compliance |

| Recommended production hardening (beyond MVP) | Addresses |
|---|---|
| Rate limits / upload quotas | Abuse, DoS |
| Malware scanning | Malicious file content |
| Background processing | Responsiveness, fault tolerance |
| Encrypted storage (or discard post-processing) | Data-at-rest risk |
| OpenAPI spec + API gateway | Documentation, centralized auth/rate-limit/logging |

## Worked Example
A user creates a knowledge base named "HR Policies," uploads several PDFs via `DocumentUploader` (drag-and-drop, PDF/DOCX supported); each upload hits `/api/upload`, which parses and chunks the document, embeds chunks with the Google AI embedding model (768-dim), and stores them in the shared Upstash Vector index tagged with that knowledge-base's and user's IDs. When the user later chats via `/api/chat/[knowledgebaseId]`, the query is embedded, a similarity search retrieves relevant chunks filtered to that knowledge-base/user, and the Vercel AI SDK generates a grounded response — the same retrieval pattern from Ch7, now backed by an external, scalable vector database (Upstash Vector) instead of the filesystem-persisted HNSWLib index.

## Key Takeaways
1. Choose shared vs. dedicated vector storage per tenant based on actual compliance/isolation requirements, and always enforce user/knowledge-base scoping as a query-level filter, not an afterthought.
2. Treat document upload/processing as both a performance and a security surface — validate, then background the heavy work, and plan for malware scanning and encryption before production.
3. Design API URLs and resource IDs defensively: hierarchical scoping, authentication on every access, UUIDs instead of guessable IDs.
4. Match vector index dimensionality to your embedding model's output size explicitly.
5. Don't over-architect (microservices) beyond what current scale/compliance needs demand — a well-organized monolith is a legitimate, deliberate choice for MVP/educational scope.

## Connects To
- **Ch7**: directly extends the RAG retrieval chain pattern, swapping `HNSWLib` for the externally hosted `UpstashVectorStore`.
- **Ch9**: reuses Clerk.js auth and validation patterns; extends them with multi-tenant vector-store scoping.
- **Ch10**: shares the same Redis-backed session-state architecture pattern.
- **Ch12**: the book's next/final technical chapter extends this application-integration mindset to MCP-based external tool access.
