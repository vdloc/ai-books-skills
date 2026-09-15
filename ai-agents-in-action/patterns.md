# Patterns, Platforms, and Techniques — AI Agents in Action

## GPT Assistants Platform (OpenAI, via ChatGPT UI)
**When to use**: fast, no-code prototyping of an agent with persona, code interpretation, custom actions, and static file-upload knowledge; publishing to the GPT Store.
**How**: persona paragraph + explicit RULES block in Configure panel; enable Code Interpreter for computed output and file uploads; add Custom Actions via OpenAPI spec + ngrok tunnel; upload docs (up to 512MB) for static knowledge.
**Trade-offs**: zero-code and fast, but usage/cost tracked on the consuming user's account; unknown users can trigger public custom actions; heavy features (image gen, code interpreter, vision) can trip resource-usage blocks.

## OpenAI Function/Tool Calling
**When to use**: any agent needing to trigger external code/data based on natural language, in raw API code (not the ChatGPT UI).
**How**: register a `tools` JSON-schema array on a chat completion call; LLM returns `tool_calls` (name + args, doesn't execute); your code executes and appends results as `role: "tool"` messages; second LLM call synthesizes a natural-language reply.
**Trade-offs**: simple and universal (becoming a cross-platform standard), but you own the two-step execution loop yourself; cheaper models suffice for the selection step.

## Semantic Kernel (SK, Microsoft)
**When to use**: orchestrating multiple plugins (semantic + native functions) with composition, or building a reusable "GPT interface" (semantic service layer) over an existing API.
**How**: `Kernel` + AI service; semantic functions (`type: completion`) via `PromptTemplateConfig`/`create_function_from_prompt`; native functions via `@kernel_function`-decorated methods; register both via `import_plugin_from_*`; embed native function calls directly inside semantic prompt templates (`{{ClassName.method}}`).
**Trade-offs**: cleaner and more composable than raw tool calling, but adds a framework/learning curve; creating a function doesn't register it as a plugin — an easy mistake.

## AutoGen (Microsoft, multi-agent)
**When to use**: exploratory/research-style multi-agent coding tasks needing flexible, conversation-driven delegation.
**How**: `UserProxyAgent` <-> `AssistantAgent`(s); nested chats (`register_nested_chats`) for critic/review loops; `GroupChat`+`GroupChatManager` for shared-context multi-agent conversation; `Cache.disk(cache_seed=...)` to persist/resume long runs.
**Trade-offs**: flexible and fast to prototype, but conversation-based delegation can lose information across hops (the "telephone game") and can consume tokens unpredictably; recommend Docker isolation for code-execution agents.

## AutoGen Studio
**When to use**: prototyping multi-agent tasks/skills via a web UI without writing orchestration code first.
**How**: Build tab -> Skills -> paste a Python function -> attach to an agent workflow -> run in Playground.
**Trade-offs**: great for exploration; still inherits AutoGen's token-cost and code-execution-isolation concerns.

## CrewAI (role-based multi-agent)
**When to use**: enterprise-style, controllable multi-agent pipelines with explicit role/task contracts.
**How**: define `Agent` (role, goal, backstory, memory, `allow_delegation`) + `Task` (description, `expected_output`, agent); assemble `Crew(agents, tasks, process=Process.sequential|hierarchical)`; add `manager_llm` for hierarchical coordination.
**Trade-offs**: more structure/control than AutoGen out of the box, but hierarchical processing can roughly double cost without materially improving output — validate the need before adding a manager layer.

## AgentOps (observability)
**When to use**: any nontrivial multi-agent run — cost and repetitive non-convergent "thoughts" are invisible without it.
**How**: `pip install agentops` (or `crewai[agentops]`), get API key, one init line; inspect dashboard for duration, tokens, cost, and "Repeat Thoughts" plot.
**Trade-offs**: near-zero integration cost; currently most integrated with CrewAI but designed to be platform-agnostic.

## Behavior Trees / Agentic Behavior Trees (ABTs)
**When to use**: explicit, debuggable, modular control flow over multi-step, multi-assistant autonomous workflows, as an alternative to pure emergent conversation.
**How**: `py_trees` (Selector/Sequence/Condition/Action/Decorator/Parallel nodes, SUCCESS/FAILURE only) with LLM-prompted assistants backing action/condition nodes (e.g., via GPT Assistants Playground's `create_assistant_action[_on_thread]`); build via back chaining (goal -> required actions -> conditions -> communication mode -> tree); tick in a loop until root SUCCESS.
**Trade-offs**: much more debuggable/scalable than FSMs or rule-based systems for agentic control, but adds structure/setup overhead versus free-form conversation; introduces variability ("stochastic behavior trees") absent from classic deterministic trees.

## Siloed vs. Conversational Assistant Threads
**When to use**: silo (isolated OpenAI Assistants thread) for clean, noise-free, debuggable steps (e.g., an independent Verifier); shared thread for iterative back-and-forth roles (e.g., Hacker+Judge, Debugger+Verifier).
**How**: create one thread and pass it to multiple assistant actions/conditions (shared) vs. a fresh thread per action (siloed).
**Trade-offs**: shared threads improve context/feedback quality but risk noise/drift; siloed threads reduce noise but lose shared context — mix both patterns as needed.

## Nexus Agent Platform (author's teaching platform)
**When to use**: reference architecture for assembling profile + actions + (later) knowledge/planning into one pluggable agent host; also usable directly for prototyping.
**How**: YAML profiles in `nexus_profiles/`; `@agent_action`-decorated functions (native or semantic, via docstring) in `nexus_actions/`; `BaseAgent` subclasses in `nexus_agents/` for new engines; Streamlit or Gradio UI.
**Trade-offs**: simpler ceremony than SK for defining actions; as of the book's writing, action access isn't yet restricted per profile — guard against over-provisioning by convention.

## RAG (Retrieval Augmented Generation)
**When to use**: grounding agent responses in documents too large for a prompt (knowledge), or in captured conversational facts (memory).
**How**: load -> split/chunk (with overlap; prefer token-based splitting) -> embed (prefer trained embeddings over TF-IDF) -> store in a vector DB (e.g., ChromaDB) -> at query time embed the query -> similarity search -> inject top-N as context -> generate.
**Trade-offs**: dramatically better than stuffing whole documents into a prompt on cost and often quality, but requires careful chunking/embedding choices; knowledge and memory share the mechanism but differ in how they're populated.

## Memory/Knowledge Compression
**When to use**: when a knowledge or memory store accumulates redundant, repetitive, or unbalanced content over time.
**How**: cluster (e.g., k-means) -> summarize each cluster into a more succinct representation; memory benefits from periodic re-compression, knowledge usually only once on load; multiple passes can further help.
**Trade-offs**: improves retrieval relevance and reduces clutter, but adds pipeline complexity and periodic maintenance cost for memory stores.

## Semantic Memory Augmentation
**When to use**: when plain memory retrieval isn't surfacing the most relevant facts.
**How**: run an LLM "memory function" pass (e.g., summarize + categorize into JSON statements) before embedding/storing new input, rather than embedding raw conversation text.
**Trade-offs**: better recall precision at the cost of an extra LLM call per memory write.

## Prompt Flow (Microsoft, prompt/profile evaluation)
**When to use**: systematic (not ad hoc) comparison of prompt/profile variants, LLMs, temperatures, or advanced params at scale.
**How**: build a `flow.dag.yaml` (Inputs -> LLM/Python blocks -> Outputs); write prompts as Jinja2 templates; define variants; batch-run against a JSONL input file; use a separate evaluation flow (rubric-based, often LLM-as-judge) to score and aggregate results; compare via Visualize Runs.
**Trade-offs**: powerful for evidence-based profile selection, but DAG execution can't natively loop (self-consistency-style repetition must be simulated via batch processing over duplicated inputs).

## Rubric + Grounding Evaluation
**When to use**: whenever "is this response good?" isn't a simple right/wrong check.
**How**: identify purpose -> define measurable criteria -> create a rating scale -> write per-level descriptions -> apply (manually or via LLM judge) -> aggregate -> ensure evaluator consistency -> iterate.
**Trade-offs**: turns subjective quality assessment into comparable, repeatable scores; requires upfront design effort and a judge model choice (different model for absolute baselines, same model OK for relative comparison).

## Direct/Few-Shot/Zero-Shot Prompting
**When to use**: question-answer for grounded Q&A; few-shot for pattern-following (even fictitious concepts); zero-shot for generalization tasks (e.g., classification) using only instructions.
**How**: question-answer injects context+question; few-shot gives example input/output pairs; zero-shot gives only rules/format, no examples.
**Trade-offs**: cheap and simple but least reliable for hard multi-step reasoning problems.

## Chain of Thought (CoT) / Zero-Shot CoT
**When to use**: multi-step logic/math/word problems.
**How**: CoT = few-shot examples with worked reasoning steps; zero-shot CoT = add "Let's think step by step" without needing worked examples.
**Trade-offs**: CoT is expensive to author per problem class; zero-shot CoT is a cheap, surprisingly effective substitute; neither guarantees correctness.

## Prompt Chaining
**When to use**: when visibility/control over each reasoning stage matters, or a single CoT prompt is too unreliable/verbose.
**How**: sequential LLM calls — decompose steps (list only) -> calculate each step -> synthesize final solution from calculated steps.
**Trade-offs**: more transparent/debuggable than monolithic CoT, but still can converge on wrong answers; more LLM calls than direct prompting.

## Self-Consistency Prompting
**When to use**: reducing variance/sampling luck on a single CoT run.
**How**: batch-run the same CoT prompt N times (simulating a loop via duplicated input rows); embed each answer; return the answer closest to the mean embedding (cosine similarity).
**Trade-offs**: better than trusting one sample, but "most consistent" isn't "most correct" — works better for simpler problems.

## Tree of Thought (ToT) Prompting
**When to use**: complex problems where CoT/self-consistency both fail, when the extra LLM-call cost is justified.
**How**: combine prompt chaining with per-node self-evaluation; only propagate to children above an evaluation threshold; execute breadth-first (depth-first impractical in DAG-based tools).
**Trade-offs**: most robust to bad reasoning paths among the reasoning techniques, but can cost a dozen-plus LLM calls per answer.

## External Sequential Planner (prompt-based, e.g., Nexus's `BasicNexusPlanner`)
**When to use**: when your LLM/platform supports tool use but not native sequential planning (most commercial LLMs besides OpenAI Assistants/Claude).
**How**: few-shot prompt with `[SPECIAL FUNCTIONS]` (e.g., `for-each` iteration), `[AVAILABLE FUNCTIONS]` (auto-populated), `[GOAL]` -> LLM returns JSON plan -> local executor walks/executes it, threading a shared context -> final LLM call summarizes the full context.
**Trade-offs**: retrofits sequential planning onto parallel-only tool-calling LLMs without switching platforms; plan execution happens fully outside the LLM (doesn't even require native tool-use support), but bypasses in-loop feedback correction that native planners (OpenAI Assistants, Claude) provide.

## Feedback-Elicitation Prompting
**When to use**: when a model gets a reasoning-heavy problem wrong and you want to improve future attempts.
**How**: reveal the correct answer, then ask the LLM to self-critique ("please review what you did wrong and suggest feedback for future similar problems"); fold the resulting feedback into future system instructions.
**Trade-offs**: cheap, no extra tooling, but effectiveness depends on model capability — works consistently on stronger reasoning models, less so on weaker ones.
