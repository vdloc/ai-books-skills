---
name: ai-agents-in-action
description: "Knowledge base from \"AI Agents in Action\" by Micheal Lanham. Use when applying agent architecture patterns, building multi-agent systems (AutoGen, CrewAI, Nexus), designing agent memory/RAG, prompt engineering with Prompt Flow, or agent reasoning/planning, studying the book, or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# AI Agents in Action
**Author**: Micheal Lanham | **Pages**: ~346 | **Chapters**: 11 | **Generated**: 2026-09-15

## How to Use This Skill
- No arguments: read the Core Frameworks & Mental Models section below to load the book's central agent model and most load-bearing techniques.
- A topic or framework name (e.g., "AutoGen", "RAG", "behavior trees", "planning"): check the Topic Index, then open the linked chapter file(s) in `chapters/`.
- A chapter number (e.g., "chapter 5" or "ch5"): open `chapters/ch05-*.md` directly.
- "Browse" or "what's covered": scan the Chapter Index table below.
- A concrete build/decision task ("which multi-agent platform should I use," "how do I evaluate my agent prompt"): check `cheatsheet.md` first for decision rules and thresholds, then `patterns.md` for the full technique writeup.

## Core Frameworks & Mental Models

**The Five-Component Agent Model** (ch1): every agent decomposes into up to five components — profile/persona (system prompt, background), actions/tools (task completion, exploration, communication), knowledge/memory (RAG-backed context), reasoning/evaluation (thinking through and checking problems), and planning/feedback (organizing multi-step execution toward a goal). Use this as your default architecture checklist for any agent you build.

**The Agent Autonomy Spectrum** (ch1): direct user interaction -> agent/assistant proxy -> agent/assistant (human-approved tool calls) -> autonomous agent (independent planning and execution). Classify any system you're building or evaluating by which mode it occupies before choosing tools — most production-ready systems deliberately stay non-autonomous because trust in autonomous decision-making, guardrails, and goal definition takes time to establish.

**Actions are the universal extension mechanism** (ch5): the LLM never executes a function itself — it only selects a tool and parses parameters; your code executes it and feeds results back for a second LLM call. Every framework in this book (Semantic Kernel plugins, AutoGen/CrewAI skills, Nexus actions, GPT Assistant custom actions) is a variation on this same two-step loop. Design actions to return maximal structured data (JSON), not pre-filtered text — this is what lets the LLM apply its own reasoning downstream ("thinking semantically").

**Parallel vs. sequential planning is the critical capability gate** (ch11): most commercial LLMs (GPT-4o, Groq, raw Azure OpenAI) support only parallel, independent tool calls. A goal with dependent steps ("search, then download each result, then save") will stall past the first step unless you use a platform with native sequential planning (OpenAI Assistants, Claude) or add an external prompt-based planner (a few-shot prompt that outputs a JSON plan, executed locally, with results summarized back). Diagnose this before choosing a platform.

**RAG is one mechanism powering both knowledge and memory** (ch8): embed -> store in a vector DB -> retrieve by similarity -> inject as context -> generate. Knowledge = static documents loaded once; memory = an agent's own captured facts, continuously written (often via an LLM "memory function" extraction pass) and periodically compressed. Use trained embeddings (not TF-IDF) whenever true semantic (paraphrase-tolerant) matching matters.

**No single reasoning technique guarantees correctness** (ch10-11): direct/zero-shot prompting -> chain of thought -> prompt chaining -> self-consistency -> tree of thought form a cost/reliability ladder, but the book's own running example (a multi-step time-travel word problem) is gotten wrong by every technique, including OpenAI's reasoning-native o1-preview model. Always pair generation with evaluation (rubric scoring, LLM-as-judge, or consistency voting) — never trust a single unverified plan or answer for a nontrivial task.

**Behavior trees give agentic systems explicit, debuggable structure** (ch6): Selector/Sequence/Condition/Action/Decorator/Parallel nodes, using SUCCESS/FAILURE (not booleans), scale better than FSMs or rule-based systems for controlling multi-step autonomous agent workflows. Build new trees via back chaining: start from the goal, work backward through required actions and conditions, then decide siloed vs. shared-thread communication before constructing the tree.

**Give agents only the actions they need** (ch3, ch6, ch11): a recurring, explicitly stated warning across the book — over-provisioned agents get confused, risk hitting API tool-count limits, and have documented "gone rogue" (unintended file deletion, unwanted code execution) in the author's own testing.

**Systematic prompt engineering requires infrastructure, not vibes** (ch9): "Test Changes Systematically" means comparing prompt/profile variants via batch runs against varied inputs, scored with an explicit rubric (often via a second LLM as judge). Small-sample manual comparison often looks inconclusive; only batch evaluation reveals which profile design actually wins.

---

## Chapter Index
| # | Title | Key Frameworks |
|---|-------|----------------|
| 1 | [Introduction to Agents and Their World](chapters/ch01-introduction-to-agents-and-their-world.md) | Four LLM interaction modes, five-component agent model, multi-agent configuration pattern |
| 2 | [Harnessing the Power of Large Language Models](chapters/ch02-harnessing-the-power-of-large-language-models.md) | OpenAI chat completions request pattern, "Write Clear Instructions" prompt tactics, LLM selection criteria |
| 3 | [Engaging GPT Assistants](chapters/ch03-engaging-gpt-assistants.md) | GPT Assistant instruction template, iterative LLM-assisted design, custom actions (OpenAPI+ngrok), file-upload knowledge |
| 4 | [Exploring Multi-Agent Systems](chapters/ch04-exploring-multi-agent-systems.md) | AutoGen (+ Studio), CrewAI, AgentOps observability |
| 5 | [Empowering Agents with Actions](chapters/ch05-empowering-agents-with-actions.md) | OpenAI function/tool calling, Semantic Kernel (semantic + native functions), semantic service layer |
| 6 | [Building Autonomous Assistants](chapters/ch06-building-autonomous-assistants.md) | Behavior tree node taxonomy, agentic behavior trees (ABTs), back chaining, GPT Assistants Playground |
| 7 | [Assembling and Using an Agent Platform](chapters/ch07-assembling-and-using-an-agent-platform.md) | Nexus platform, Streamlit chat pattern, BaseAgent abstraction, decorator-based actions |
| 8 | [Understanding Agent Memory and Knowledge](chapters/ch08-understanding-agent-memory-and-knowledge.md) | RAG two-phase pattern, memory taxonomy, semantic memory augmentation, memory/knowledge compression |
| 9 | [Mastering Agent Prompts with Prompt Flow](chapters/ch09-mastering-agent-prompts-with-prompt-flow.md) | Prompt Flow, rubric + grounding evaluation, LLM-as-judge |
| 10 | [Agent Reasoning and Evaluation](chapters/ch10-agent-reasoning-and-evaluation.md) | Direct/few-shot/zero-shot prompting, CoT, prompt chaining, self-consistency, tree of thought |
| 11 | [Agent Planning and Feedback](chapters/ch11-agent-planning-and-feedback.md) | Sequential vs. parallel planning, external JSON planner, feedback-elicitation prompting, component application matrix |

## Topic Index
- **Actions/tools** -> Ch5, Ch7, Ch11
- **AgentOps (observability)** -> Ch4
- **Agentic behavior trees (ABTs)** -> Ch6
- **AutoGen** -> Ch4
- **Back chaining** -> Ch6
- **Behavior trees** -> Ch6
- **Chain of thought (CoT)** -> Ch10
- **Compression (memory/knowledge)** -> Ch8
- **CrewAI** -> Ch4
- **Custom actions (OpenAPI)** -> Ch3, Ch5
- **Embeddings** -> Ch8
- **Evaluation** -> Ch9, Ch10, Ch11
- **Feedback** -> Ch11
- **Few-shot prompting** -> Ch2, Ch10
- **Function/tool calling (OpenAI)** -> Ch5, Ch7
- **GPT Assistants** -> Ch3, Ch6, Ch11
- **GPT Assistants Playground** -> Ch6, Ch11
- **Knowledge (RAG)** -> Ch3, Ch8
- **Memory (agent)** -> Ch8
- **Multi-agent systems** -> Ch1, Ch4, Ch6
- **Nexus platform** -> Ch7, Ch8, Ch11
- **Persona/profile** -> Ch1, Ch3, Ch7, Ch9
- **Planning** -> Ch1, Ch6, Ch11
- **Prompt chaining** -> Ch10
- **Prompt engineering (fundamentals)** -> Ch2
- **Prompt Flow** -> Ch9, Ch10
- **RAG (retrieval augmented generation)** -> Ch8
- **Reasoning** -> Ch10, Ch11
- **Rubrics/grounding** -> Ch9
- **Self-consistency prompting** -> Ch10
- **Semantic Kernel (SK)** -> Ch5, Ch10
- **Streamlit** -> Ch7
- **Tree of thought (ToT)** -> Ch10
- **Vector databases (ChromaDB)** -> Ch8
- **Zero-shot prompting** -> Ch10

## Supporting Files
- [patterns.md](patterns.md) — every concrete technique/platform/pattern across all 11 chapters, with when-to-use/how/trade-offs
- [cheatsheet.md](cheatsheet.md) — platform decision guide, decision rules, author-stated thresholds, and tells/smells for when an approach is wrong

---

## Scope & Limits
This skill covers the book content only. For hands-on implementation in your codebase, combine with project-specific tools (e.g. actual OpenAI/AutoGen/CrewAI/Semantic Kernel SDKs). For topics beyond this book, check related skills or ask the agent directly.
