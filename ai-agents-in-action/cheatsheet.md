# Cheatsheet — AI Agents in Action

## Platform Decision Guide

| Need | Pick | Because |
|---|---|---|
| No-code prototype with persona + actions + knowledge | GPT Assistants (ChatGPT UI) | Fastest path to a working agent; publishable to GPT Store |
| Raw API tool-calling, single integration | OpenAI function/tool calling | Simplest, becoming cross-platform standard; no framework needed |
| Composable plugins (semantic + native functions), or wrapping an API as a "GPT interface" | Semantic Kernel | Clean composition; embed native calls inside prompt templates |
| Flexible, research-style multi-agent conversation | AutoGen (+ Studio) | Emergent delegation, nested/group chat, good for exploratory coding tasks |
| Enterprise-style, controllable multi-agent pipeline | CrewAI | Explicit role/task contracts, sequential or hierarchical process |
| Explicit, debuggable, structured multi-step autonomy | Agentic Behavior Trees (ABTs) | Selector/Sequence/Condition nodes > free-form conversation for control |
| Sequential dependent multi-step goal execution | OpenAI Assistants or Claude (native planning) OR an external planner (Nexus-style) | Most LLMs (GPT-4o, Groq, Azure OpenAI raw) support only parallel tool calls |
| Document-grounded Q&A or fact memory | RAG (embeddings + vector DB, e.g., ChromaDB) | Standard mechanism for both knowledge and memory |
| Systematic prompt/profile evaluation at scale | Prompt Flow | Variant comparison + batch runs + rubric-based grounding |
| Multi-step math/logic reasoning | CoT / zero-shot CoT / prompt chaining / self-consistency / ToT (escalate as needed) | Match cost to problem difficulty |
| Observability into multi-agent cost/repetition | AgentOps | Otherwise cost and repeated-thought loops are invisible |

## Decision Rules

- **When a goal has dependent sequential steps, use a platform with native planning (OpenAI Assistants, Claude) or add an external planner** — because parallel-tool-calling-only LLMs (GPT-4o, Groq, raw Azure OpenAI) will stall past the first dependent step.
- **When giving an agent tools, give it only what the goal needs** — because excess tools cause confusion, risk hitting API tool-count limits, and increase the odds of unintended/unsafe behavior (the book documents agents deleting files and running unintended code when over-provisioned).
- **When wrapping an API as an agent action, return maximal structured data (JSON), not pre-filtered text** — because pre-filtering denies the LLM the ability to apply its own downstream filtering/reasoning ("thinking semantically," ch5).
- **When choosing a judge model for rubric evaluation, use a different (often stronger) model for absolute baselines, and the same model only for relative (profile A vs. B) comparisons** — because same-model judging can bias toward what that model favors.
- **When a memory or knowledge store shows redundant/repetitive/unbalanced content, compress it (cluster + summarize)** — memory benefits from periodic re-compression; knowledge stores usually only need it once, on load; multiple passes can help further.
- **When a DAG-based orchestration tool (Prompt Flow) needs looping behavior (e.g., self-consistency), simulate it via batch processing over duplicated inputs** — DAGs can't natively loop.
- **When adding a hierarchical manager to a multi-agent crew, validate the coordination overhead first** — the book's own experiment showed hierarchical CrewAI processing roughly doubling cost with no material quality gain over sequential.
- **When a single reasoning technique (direct/zero-shot/CoT) fails a hard problem, escalate rather than trust a lucky sample** — try zero-shot CoT -> prompt chaining -> self-consistency -> tree of thought, in increasing cost order; even reasoning-native models (o1-preview) can still get hard problems wrong.
- **When publishing a custom action (OpenAPI + ngrok) on a public assistant, never expose fee-incurring or private endpoints** — unknown users can trigger it.
- **When running agent-generated code, isolate it (Docker/WSL)** — AutoGen explicitly recommends Docker; unisolated local execution risks host-level side effects.

## Author-Stated Thresholds / Defaults

| Item | Value | Source context |
|---|---|---|
| GPT Assistants file-upload knowledge limit | Up to 512 MB per assistant | Ch3 |
| OpenAI Code Interpreter cost | ~$0.03 per run | Ch6 |
| Temperature: deterministic vs. max variability | 0 vs. 1.0 | Ch2, Ch9 |
| Default recommendation splitting (character-based) | chunk_size=100, chunk_overlap=25 | Ch8 |
| Token-based splitting example | chunk_size=50, chunk_overlap=10 | Ch8 |
| Cosine similarity range | -1 (dissimilar) to 1 (identical) | Ch8 |
| Cosine distance range | 0 (identical) to 2 (opposite) | Ch8 |
| OpenAI embedding dimensionality (ada-002) | 1536 dimensions | Ch8 |
| Rubric rating scale (book's example) | 1 (poor) to 5 (excellent) | Ch9 |
| ToT evaluation propagation threshold (example) | score > 25 (of 100) to continue | Ch10 |
| ToT LLM call cost (example problem) | up to ~27 calls per answer | Ch10 |
| Recommended posting cadence for autonomous social-posting ABTs | A few posts/day max | Ch6 (avoid account blocking) |

## Tells and Smells — When Your Approach Is Wrong

- **Agent stalls after the first step of a multi-step goal** -> you're on a parallel-tool-calling-only LLM; add a planner or switch to a platform with native sequential planning (ch11).
- **Retrieval returns word-matchy but semantically wrong results** -> you're likely using TF-IDF instead of trained embeddings (ch8).
- **LLM judge scores look inconsistent or hard to interpret** -> you skipped explicit rubric criteria/scale/descriptions before automating scoring (ch9).
- **Agent conversation "resolves" quickly but the answer is subtly wrong** -> no evaluation/grounding step exists; add rubric scoring or a second-opinion LLM pass (ch9, ch10).
- **AgentOps "Repeat Thoughts" plot shows high repetition** -> the agent isn't being decisive; change roles/tasks/process rather than throwing more calls at it (ch4).
- **Hierarchical multi-agent run costs much more with no better output** -> the manager/coordination layer isn't earning its overhead; revert to sequential (ch4).
- **Custom action assistant gets blocked or costs spiral** -> check for heavy-resource features (image gen, code interpreter, vision, file uploads) without usage-aware rules (ch3).
- **Agent "goes rogue" (unexpected file/code actions)** -> it has more actions available than the goal requires; trim the action list (ch11).
- **Knowledge/memory store returns duplicate or diluted results over time** -> compress via clustering + summarization (ch8).
