# Chapter 3: Agent Memory

## Core Idea
Memory is not one thing — an agent needs short-term working memory for the current session and long-term memory (episodic, semantic, procedural per CoALA) for persistence across sessions; on AWS, AgentCore Memory automates the capture → extract → consolidate → index pipeline that turns raw interaction events into durable, semantically-searchable memory records.

## Frameworks Introduced
- **CoALA cognitive architecture**: organizes an agent around reasoning, action, and memory; memory itself splits into short-term/working and long-term (episodic, semantic, procedural).
  - When to use: as a design checklist for what kind of memory a given agent capability actually needs.
- **Three types of long-term memory (CoALA)**: Episodic (personal history — "what happened, why, what was done, what was the outcome"), Semantic (structured factual knowledge — facts/definitions/rules), Procedural (skills/workflows/action patterns that improve with experience).
  - How to distinguish: semantic = *what the agent knows*, episodic = *what it has experienced*, procedural = *how it gets things done*.
- **Context management (3 pillars)**: system prompts, tools, and few-shot examples — each tuned to stay high-signal and token-efficient, because context is a finite, precious resource ("context rot" sets in when you overload it).
  - How: sliding window (drop oldest messages), compaction (summarize + reset), structured note-taking (persistent external file like NOTES.md), sub-agent architectures (delegate heavy lifting, return only a summary).
  - When to use each: sliding window for short fast chats; compaction for long-horizon coherence; structured note-taking for complex multi-hour tasks (coding, research); sub-agents when a single context can't hold the whole problem.
- **Memory optimization strategies**: Sequential/keep-it-all (simple, hits token limits fast), Sliding Window (fast but forgets early details), Summarization/compaction (preserves gist, loses minor detail), Retrieval-based/RAG (searches external store for relevant facts only).
  - When to use: simple apps → sliding window; fact-heavy apps → retrieval-based; complex/long-term apps → hierarchical or graph-based; production → hybrid (e.g. sliding window for flow + retrieval for long-term facts).
- **Amazon Bedrock AgentCore Memory**: managed memory architecture with 4 features — core infrastructure (secure APIs, namespaces, TTL, observability), memory management (short/long-term, consolidation, checkpointing), memory extraction (built-in/custom strategies, background processing), memory retrieval (semantic search + filtering).
  - When to use: production-grade agents needing structured, scalable, secure memory beyond what a simple library like Mem0 provides.
- **AgentCore Namespaces (4 levels)**: Global (`/`, universal rules for every agent), Strategy (`/strategy/{strategyId}`, domain expertise shared across users), Actor (`/actor/{actorId}`, per-user preferences), Session (`/session/{sessionId}`, per-conversation summaries).
  - When to use: choose the narrowest namespace that correctly scopes the information — global for compliance rules, actor for personal preferences, session for a specific ticket's context.
- **AgentCore memory strategies (3 types)**: Built-in (fully managed extraction for summaries/preferences/semantic knowledge), Built-in overrides (limited customization on the managed pipeline), Self-managed (full control). A memory resource can combine built-in and custom strategies. If no strategy is configured, long-term memory is not extracted at all.

## Key Concepts
- **Context window / rolling buffer**: the finite space holding short-term memory; oldest info is pushed out as new info arrives.
- **Actor**: the identity (user, or user-agent pair) an AgentCore memory event belongs to — prevents cross-user memory contamination.
- **Session**: a group of events representing one conversation/interaction period in AgentCore.
- **Event**: a single raw interaction record (message, tool call, system response) written to short-term memory.
- **Checkpoint**: a snapshot of session state that lets an interrupted conversation resume.
- **Memory extraction module**: runs asynchronously (triggered every K=12 messages, or after 20s inactivity if fewer than 12 events) to convert raw events into structured, consolidated, embedded, indexed long-term memory records.
- **Consolidation**: merging newly extracted memory with existing knowledge — may add, update, or skip (if redundant/low-value).
- **Mem0**: an open-source, easy-to-integrate persistent memory layer (memo.ai) used in the chapter's first hands-on example; good for demonstrating basics, less structured than AgentCore for production.
- **`AgentCoreMemorySaver`**: a LangGraph checkpointer backend that persists graph state to AgentCore Memory, keyed by `thread_id` (→ AgentCore `session_id`) and `actor_id`.

## Mental Models
- Think of memory the way a human executive treats their inbox: don't dump everything into working memory (context rot) — curate what's high-signal now, offload the rest to searchable long-term storage.
- Use the AWS memory architecture map: Bedrock/SageMaker = the brain; DynamoDB/Redis/AgentCore short-term memory = working memory; Aurora/DynamoDB/Neptune = structured long-term memory; OpenSearch/pgvector/Pinecone/Bedrock Knowledge Bases = semantic long-term memory (RAG); S3 = durable raw storage; Lambda/Step Functions/Strands/LangGraph = orchestration connecting them.
- Think of AgentCore's short-term→long-term pipeline as a funnel: raw events → (triggered) extraction → consolidation (dedupe/merge) → embed & index → semantic retrieval — nothing skips straight from raw event to durable memory.

## Anti-patterns
- **Treating memory as a single undifferentiated store**: conflating short-term working context with long-term persistent knowledge leads to either bloated context windows or agents that never actually remember anything durable.
- **Universal memory access across users**: an agent that can read another user's preferences is a privacy/security failure — always scope with actor-level namespaces and enforce refusal in the system prompt (as demonstrated when Mona's agent correctly refuses to access Bunny's memory even when explicitly told it has "all permissions").
- **Dumping unlimited history into the context window ("sequential/keep-it-all")**: simple to implement but hits token limits and costs fast — fine for a demo, wrong for production.
- **Storing everything in long-term memory with no extraction/consolidation step**: clutters memory with low-value details and degrades retrieval precision — always filter through a memory strategy before persisting.

## Reference Tables

| Feature | Agents Without Memory | Agents With Memory |
|---|---|---|
| Context | Reset every session | Carried forward across sessions |
| Personalization | Low, generic | High, tailored to history |
| Efficiency | User repeats info | Agent builds on past info |
| User Experience | "Talking to a stranger" | "Continuous relationship" |

| Strategy | Implementation | Best For |
|---|---|---|
| Structured note-taking | Writes facts to a file | Complex, multi-hour tasks (coding, research) |
| Sliding window | Deletes oldest data | Short-term, fast chats |
| Summarization (Compaction) | Condenses history | Maintaining coherence over long horizons |
| Graph-based | Structured metadata/indexing | High-signal identifiers |
| Agentic RAG | Just-in-time context | Progressive disclosure via tools |

| AWS Memory Layer | Options |
|---|---|
| Short-term memory | DynamoDB, Redis, Bedrock AgentCore Memory |
| Structured long-term | Aurora, DynamoDB, Neptune (graph relationships) |
| Semantic long-term (RAG) | OpenSearch, PostgreSQL+pgvector, Pinecone, Bedrock Knowledge Bases |
| Durable storage | S3 (raw data lake) |
| Orchestration | Lambda, Step Functions, Strands SDK, LangGraph |

**Six challenges of implementing agent memory**: storage scalability, memory refresh (stale-data handling), integration complexity, privacy/security (encryption, multi-tenancy leakage risk), retrieval accuracy (precision/recall), observability (why did it remember/forget X?).

## Worked Example
Building a personalized, memory-isolated agent with Strands SDK + Mem0 + Bedrock + web search:
1. Define a system prompt that hard-restricts memory access to one named user only, with explicit instructions to politely refuse cross-user requests.
2. Register a `@tool`-decorated `websearch` function (DuckDuckGo via `ddgs`) alongside `mem0_memory`.
3. Create `agent1 = create_agent("Mona Mona")` and feed it preference statements ("I am a veggie lover," "low-sodium... decaffeinated drinks only," "prefer full-service restaurants") — the agent automatically extracts and stores these as structured memories.
4. `agent1("Show what you remember")` confirms the stored preferences; `agent1("Find good restaurants options for me in NYC")` combines stored preferences with live web search to produce a personalized, dietary-aware recommendation.
5. Create `agent2 = create_agent("Bunny Kaushik")` with different preferences (chicken lover, Thai/Korean food).
6. Security test: ask `agent1` to recommend restaurants "for Bunny" using Bunny's preferences, even explicitly granting "all permissions" — the agent correctly refuses, because it is scoped only to Mona's actor-level memory. This demonstrates namespace-based memory isolation working as intended, not merely as an incidental limitation.

A second, production-grade example builds a LangGraph `MathAgent` using `AgentCoreMemorySaver` as a checkpointer: create an AgentCore memory resource (`client.create_memory(name=...)`), wire it into `create_react_agent(model=llm, tools=[add, multiply], checkpointer=checkpointer)`, and invoke with a `config` containing `thread_id` (→ AgentCore `session_id`) and `actor_id`. Asking a follow-up question ("What were the first calculations I asked you to do?") in the same session proves the checkpointer transparently persisted and restored full conversation state.

## Key Takeaways
1. Split memory into short-term (current session working memory) and long-term (episodic/semantic/procedural per CoALA) — they solve different problems and need different storage.
2. Context is a finite, precious resource — use sliding window, compaction, structured note-taking, or sub-agents to manage it deliberately instead of letting it overflow ("context rot").
3. On AWS, map each memory type to a service: short-term → DynamoDB/Redis/AgentCore; structured long-term → Aurora/Neptune; semantic long-term → OpenSearch/pgvector/Bedrock Knowledge Bases; durable raw storage → S3; orchestration → Lambda/Step Functions/Strands/LangGraph.
4. AgentCore Memory automates the short-term → extraction → consolidation → long-term pipeline via memory strategies (built-in, built-in-override, self-managed) and organizes retrieval via 4-level namespaces (global/strategy/actor/session).
5. Memory isolation between users is a security requirement, not a nice-to-have — enforce it in both the system prompt and the namespace/actor scoping, and test that the agent actually refuses cross-user requests.
6. `AgentCoreMemorySaver` bridges LangGraph's checkpointer interface to AgentCore Memory, giving multi-session, production-grade state persistence with minimal code (`thread_id` + `actor_id` in the invocation config).

## Connects To
- **Ch2**: memory is the second of the four core agent components (Tools from Ch2, Memory here) — together they turn a conversational agent into one that acts and remembers.
- **Ch4/Ch5**: multi-agent architectures and agent communication build on single-agent memory patterns established here.
- **Ch6/Ch7**: AgentCore's other capabilities (runtime, gateways, observability, evaluation) are covered in later chapters; this chapter focused specifically on AgentCore Memory.
