# Chapter 11: Agent Planning and Feedback

## Core Idea
Planning is what separates an agent from a chatbot: raw LLMs (even with parallel tool/action support) cannot reliably execute *sequential*, dependent multi-step goals unless the platform has built-in planning (OpenAI Assistants, Claude) or you bolt on an external prompt-based planner; feedback then closes the loop by helping models correct wrong reasoning — but as the return of the chapter-10 time-travel problem shows, even OpenAI's o1-preview ("Strawberry," with reasoning built in) can still get a plan-heavy problem wrong, so evaluation and feedback remain necessary companions to planning, not optional extras.

## Frameworks Introduced
- **Parallel vs. Sequential Action Execution**: the key capability distinction between ordinary tool-calling LLMs and planning-capable platforms.
  - When to use: diagnose which you need before choosing a platform — if goal steps are independent, parallel tool calling (ch5/ch7) suffices; if step N depends on step N-1's output, you need sequential planning.
  - How: most commercial LLMs (GPT-4o, Groq, Azure OpenAI) support only parallel action calls; OpenAI Assistants and Anthropic Claude support built-in sequential planning; everything else needs an external planner layered on top.
- **External Prompt-Based Sequential Planner** (Nexus's `BasicNexusPlanner`): a few-shot prompt that asks the LLM to output a JSON plan of subtasks (restricted to a declared function list, plus a special `for-each` iteration construct), which is then parsed and executed locally, with results fed back to the LLM for final summarization.
  - When to use: when your LLM/platform supports tool use but not native sequential planning, and you want planning without switching platforms.
  - How: build a planning prompt template with a preamble + few-shot examples + `[SPECIAL FUNCTIONS]` (e.g., `for-each`) + `[AVAILABLE FUNCTIONS]` (auto-populated from the agent's action list) + `[GOAL]` -> LLM returns JSON `{"subtasks": [...]}` -> a local executor (`execute_plan`) walks the JSON, handling `for-each` specially (iterating a function over a list, threading results through a shared context) -> the full context (all subtask outputs) is sent back to the LLM for a final natural-language summary.
- **Feedback-Elicitation Prompting**: ask the LLM itself to generate corrective feedback once you reveal the correct answer, then reuse that feedback in future prompts/instructions.
  - When to use: when a model gets a reasoning-heavy problem wrong and you want to improve future attempts without manually authoring guidance.
  - How: after a wrong answer, prompt: "the correct answer is X, please review what you did wrong and suggest feedback you could give yourself when trying to solve similar future problems" — extract the LLM's self-critique and fold it into future system instructions.
- **Component Application Matrix** (planning / reasoning / evaluation / feedback x application type): a decision framework for where/when/how to implement each of the four agentic components across six application types (personal assistant, customer service bot, autonomous agent, collaborative workflows, game AI, research).
  - When to use: when scoping which agentic components a given product actually needs — not every application benefits from heavy reasoning or internal feedback.
  - How: see Reference Tables below; the general pattern is that customer service bots need the least of all four components (controlled, low-tool-use environments), while autonomous agents and research applications need the most (planning and reasoning inside the loop, evaluation/feedback often external and after the fact).

## Key Concepts
- **Planning vs. chatbot behavior**: an agent that can't plan and only follows simple interactions is "nothing more than a chatbot" — planning is what enables taking a goal, decomposing it, and returning results.
- **`for-each` special function**: the planner DSL construct enabling iteration (e.g., "for each topic, generate a joke") without true programming-language loop syntax — necessary because plans are expressed as static JSON, not executable code.
- **Plan isolation**: planning prompts are typically built without full conversation history/context, to save tokens and keep the LLM focused purely on the goal and available functions.
- **Local plan execution**: the executor that walks the parsed JSON plan runs independently of the LLM (calling APIs, running code, etc.) — meaning plan execution doesn't require an LLM that supports tool use at all, only one that can output a valid plan.
- **Bypassing agent-engine tool use**: when a planner is enabled, the agent engine's normal tool-calling mechanism is bypassed entirely — the planner owns action execution and the agent only sees results via context.
- **OpenAI Strawberry (o1-preview)**: a model integrating reasoning, planning, evaluation, and feedback directly at the LLM level rather than requiring external prompt engineering — still shown to get the chapter's signature hard problem wrong, underscoring that "smarter" models reduce but don't eliminate the need for external evaluation/feedback.
- **Action selection discipline**: giving an agent only the actions it needs (not everything available) reduces confusion, respects API tool-count limits, and reduces risk of unintended/unsafe tool use — the chapter explicitly warns of agents "going rogue" (downloading files, running unintended code, deleting files) when over-provisioned with actions.

## Mental Models
- Before choosing a platform, ask "does this goal require sequential, dependent steps?" — if yes, either pick a platform with native planning (OpenAI Assistants, Claude) or add an external planner; parallel-only tool calling will silently fail past the first dependent step.
- Treat feedback as a separate, often-external component from planning/reasoning — most applications (per the chapter's own tables) implement feedback after the interaction completes, not inline, because inline feedback loops are hard to get right and can loop pathologically.
- Match component intensity to application type: customer service bots want minimal reasoning/planning/feedback (controlled, fast, low-risk); autonomous agents and research applications want the most (complex tool use, in-loop reasoning, and both internal and external evaluation).

## Anti-patterns
- **Assuming any tool-calling LLM can execute a sequential, dependent multi-step goal**: GPT-4o/Groq/Azure OpenAI support parallel actions only — a goal like "search Wikipedia, then download each found page, then save to a file" will stall after the first step without a planner or a platform with native sequential planning.
- **Giving an agent every available action "just in case"**: increases confusion, risks hitting API tool-count limits, and increases the chance of unintended/unsafe behavior (file deletion, unwanted code execution) — the chapter's explicit "agents going rogue" warning.
- **Trusting a "smarter"/reasoning-native model's answer to a complex problem without verification**: o1-preview ("Strawberry") still answered the chapter's time-travel problem incorrectly despite integrated reasoning — model improvements reduce but don't eliminate the need for evaluation/feedback.
- **Applying heavy reasoning/planning/feedback uniformly across all application types**: e.g., adding deep reasoning to a real-time game AI or a customer service bot trades response time and simplicity for a benefit that context doesn't need — match component intensity to the application (see matrix).
- **Building in-loop feedback for research/complex agentic pipelines by default**: the chapter notes multi-agent feedback+evaluation loops "don't always perform well" with current models for research-style tasks, and isolating feedback/evaluation to the end often works better than continuous looping.

## Code Examples
```python
def execute_plan(self, nexus, agent, plan: Plan) -> str:
    context = {}
    plan = plan.generated_plan
    for task in plan["subtasks"]:
        if task["function"] == "for-each":
            list_name = task["args"]["list"]
            index_name = task["args"]["index"]
            inner_task = task["args"]["function"]
            list_value = context.get(list_name, [])
            for item in list_value:
                context[index_name] = item
                result = nexus.execute_task(agent, inner_task, context)
                context[f"for-each_{list_name}_{item}"] = result
            for_each_output = [context[f"for-each_{list_name}_{item}"] for item in list_value]
            context[f"for-each_{list_name}"] = for_each_output
            for item in list_value:
                del context[f"for-each_{list_name}_{item}"]
        else:
            result = nexus.execute_task(agent, task, context)
            context[f"output_{task['function']}"] = result
    return context
```
- **What it demonstrates**: how a JSON-described plan (produced by the LLM planner) gets executed entirely in local Python — walking each subtask, special-casing `for-each` iteration by threading a shared `context` dict through repeated calls, and returning the full context (all subtask outputs) for a final LLM summarization pass. This is the concrete mechanism behind "plan execution doesn't require the LLM to support tool use."

## Reference Tables
| Application | Planning | Reasoning | Evaluation | Feedback |
|---|---|---|---|---|
| Personal assistant | Within LLM, facilitates tool use | Within LLM, limited for real-time UX | External, after interaction | External or user-driven, via memory |
| Customer service bot | Not typical (restricted, no tool use) | Not typical | External monitor, after interaction | External monitor, after interaction (survey) |
| Autonomous agent | Within agent/LLM, essential | Within LLM, amount still unclear | External or internal, after/during | External, after interaction |
| Collaborative workflows | Within LLM | Within LLM, adds overhead | External, after interaction | During interaction (immediate) |
| Game AI | Within LLM | Within LLM, but latency-sensitive | External or internal | External or internal, after/during |
| Research | Anywhere, before/during/after | Anywhere, essential | Combined manual + LLM, after output | Combined manual + LLM, after output |

## Worked Example
The chapter runs one goal ("Search Wikipedia for pages on {topic}, download each page, save to Wikipedia_{topic}.txt") through three configurations to isolate planning's effect: (1) a plain Nexus agent (planner set to None) with `search_wikipedia`/`get_wikipedia_page`/`save_file` actions — it executes the search action but stalls, unable to sequence the dependent download/save steps; (2) the same goal given to an OpenAI Assistant (via GPT Assistants Playground) with the same three actions — the assistant completes the entire sequential chain autonomously via its built-in planning, and when an unsupported extra task ("summarize each page") is added to the goal, it improvises using its own summarization ability even though no tool exists for it; (3) the same goal run in Nexus with the `BasicNexusPlanner` enabled — the external JSON-based planner (Listing 11.3/11.4/11.5) builds a plan, executes it locally via `execute_plan`, and returns a completed file plus extra contextual commentary, demonstrating that external planning can retrofit sequential capability onto a parallel-only tool-calling LLM.

## Key Takeaways
1. Planning is the dividing line between "agent" and "chatbot" — without it, multi-step, dependent goals fail past the first step on most commercially available parallel-tool-calling LLMs.
2. Know your platform's actual capability: OpenAI Assistants and Claude support native sequential planning; GPT-4o/Groq/Azure OpenAI (raw) support only parallel actions and need an external planner for dependent multi-step goals.
3. An external planner can be built with prompt engineering alone (few-shot JSON plan generation + a `for-each`-style iteration construct + local execution) — no special framework is required, though LangChain/SK offer ready-made alternatives.
4. Plan execution happens locally, independent of the LLM's own tool-use support — this means planners can add sequential capability to models that can't natively call tools at all.
5. Even reasoning-integrated models (OpenAI's o1-preview/"Strawberry") can still get complex problems wrong — model-level reasoning reduces but does not eliminate the need for external evaluation and feedback.
6. You can generate your own corrective feedback by asking the LLM to self-critique once given the correct answer, then reuse that feedback in future prompts/instructions.
7. Match the intensity of planning, reasoning, evaluation, and feedback to the application type — controlled/low-risk applications (customer service bots) need the least; complex, high-autonomy applications (autonomous agents, research) need the most, often with evaluation/feedback external and after the fact rather than in-loop.

## Connects To
- **Ch1**: closes the loop on the five-component agent model — planning/feedback was the last component introduced there and is now fully implemented.
- **Ch5, Ch7**: directly builds on parallel action/tool-calling infrastructure, showing its limits and how planning extends it.
- **Ch6**: agentic behavior trees are another structured-planning approach; this chapter's JSON planner is a lighter-weight, prompt-only alternative.
- **Ch10**: reuses the same time-travel reasoning problem to show that even reasoning-native models (o1-preview) don't solve the evaluation/feedback need — planning and reasoning remain distinct, complementary components.
