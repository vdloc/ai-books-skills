# Chapter 6: Building Autonomous Assistants

## Core Idea
Behavior trees — a decades-old robotics/game-AI control pattern of selector, sequence, condition, action, decorator, and parallel nodes — can control agents too, becoming "agentic behavior trees" (ABTs) where prompts to different assistants stand in for actions/conditions, giving you a scalable, debuggable, modular way to build autonomous multi-step, multi-assistant systems without pure emergent conversation.

## Frameworks Introduced
- **Behavior Tree Node Taxonomy**: Selector/fallback, Sequence, Condition, Action, Decorator, Parallel.
  - When to use: whenever you need explicit, debuggable control flow over agent execution rather than relying purely on emergent conversation.
  - How: Selector tries children until one succeeds (OR); Sequence runs children until one fails (AND); Condition returns success/failure based on a check; Action does the actual work; Decorator wraps/gates a child (can act as a control barrier function); Parallel runs children concurrently with a success threshold. Trees tick top-to-bottom, left-to-right; every node returns SUCCESS or FAILURE (no boolean branching).
- **Agentic Behavior Trees (ABTs)**: behavior trees where action/condition nodes are backed by LLM-prompted assistants instead of deterministic code.
  - When to use: multi-step autonomous workflows (coding-challenge pipelines, content pipelines, debugging loops) that need explicit structure plus LLM flexibility at each node.
  - How: build with `py_trees` + a wrapper (e.g., GPT Assistants Playground's `create_assistant_action[_on_thread]` / `create_assistant_condition[_on_thread]`); each node's assistant returns SUCCESS/FAILURE (or produces a file the next node consumes); `tree.tick()` in a loop with a sleep delay until the root reports SUCCESS.
- **Back Chaining (ABT construction method)**: build a tree by working backward from the goal.
  - When to use: whenever you're designing a new ABT from scratch and don't yet know the node structure.
  - How: (1) identify the goal behavior, (2) determine required actions leading to it, (3) identify conditions each action needs, (4) decide communication mode (siloed threads vs. shared conversation thread vs. mixed), (5) construct the tree bottom-up/reversed into a Sequence (or Selector) of action/condition nodes.
- **Siloed vs. Conversational Assistant Threads**: assistants can share one OpenAI Assistants message thread (conversational) or each get an isolated thread (siloed), or a mix.
  - When to use: silo when you want clean, debuggable, noise-free interactions (e.g., a Verifier working alone); share threads when iterative back-and-forth feedback between two roles (e.g., Hacker + Judge, or Debugger + Verifier) benefits from full shared context.

## Key Concepts
- **Behavior tree**: hierarchical control structure using SUCCESS/FAILURE (not booleans), first used in robotics (Rodney Brooks, 1986), now standard in game AI.
- **Fallback (Selector) node**: returns success on the first successful child; falls back through alternatives otherwise.
- **Control barrier function**: a decorator-style safety gate that blocks/prevents unwanted behaviors — the mechanism for guardrails on autonomous agents.
- **Stochastic behavior tree**: an alternative name for ABTs, acknowledging that LLM-driven nodes introduce randomness/variability that classic deterministic behavior trees don't have.
- **Blackboard pattern**: the traditional behavior-tree mechanism for cross-node communication via a shared key/value store; the chapter deliberately substitutes file-based communication for simplicity/transparency instead.
- **GPT Assistants Playground**: a Gradio-based open-source project (author's own) mimicking/extending the OpenAI Assistants Playground with custom actions, a local code runner, and detailed action/tool logging.
- **Manager Assistant**: a special installed assistant with access to all actions, used to install/manage other predefined assistants (e.g., "Please install the Python Coding Assistant").
- **GPT vs. Assistant (OpenAI terminology)**: a GPT runs inside ChatGPT (billed to the ChatGPT account); an Assistant is consumed via the API only (billed per token/tool usage, e.g., Code Interpreter at $0.03/run) and needs custom code.

## Mental Models
- Use a Selector when you want "try option A, fall back to option B" logic (OR); use a Sequence when every step must succeed for the goal to be met (AND) — this is the same logic as short-circuit boolean evaluation but expressed as success/failure propagation.
- Use back chaining whenever you're unsure how to decompose an autonomous goal into concrete agent steps — start from the goal and ask "what must be true/done right before this succeeds?" repeatedly.
- Combine siloed and conversational patterns: silo when isolation/debuggability matters more than context sharing; converse when iterative feedback quality matters more than noise control.

## Anti-patterns
- **Giving one assistant (e.g., a Manager Assistant) unrestricted access to all actions by default**: increases hallucination/mistake risk — keep assistants goal-specific with minimal action sets.
- **Letting agents freely generate arbitrary code without isolation**: the Playground's local code runner is not sandboxed like Docker — anything beyond simple scripts risks host-level side effects; use AutoGen+Docker (ch4) for anything riskier.
- **Running open-ended autonomous ABTs against public platforms without rate awareness**: the YouTube-to-X posting ABT explicitly warns that posting more than a few times a day risks account blocking — autonomous loops need external-world rate limits baked in.
- **Choosing FSMs or rule-based systems for agentic control at scale**: both are called out as impractical for LLM-powered agents because they don't scale or compose well compared to behavior trees.

## Code Examples
```python
root = py_trees.composites.Sequence("RootSequence", memory=True)
thread = api.create_thread()

debug_code = create_assistant_action_on_thread(
    thread=thread,
    action_name="Debug code",
    assistant_name="Python Debugger",
    assistant_instructions=f"""
        Here is the code with bugs in it:
        {bug_file}
        Run the code to identify the bugs and fix them.
        Be sure to test the code to ensure it runs without errors or throws any exceptions.
    """,
)
root.add_child(debug_code)

verify = create_assistant_condition_on_thread(
    thread=thread,
    condition_name="Verify",
    assistant_name="Python Coding Assistant",
    assistant_instructions="""
        Verify the solution fixes the bug and there are no more issues.
        Reply with SUCCESS if the solution is correct, otherwise return FAILURE.
        If you are happy with the solution, save the code to a file called fixed_bug.py.
    """,
)
root.add_child(verify)

tree = py_trees.trees.BehaviourTree(root)
while True:
    tree.tick()
    if root.status == py_trees.common.Status.SUCCESS:
        break
    time.sleep(20)
```
- **What it demonstrates**: a minimal two-node conversational ABT — a debug-action assistant and a verify-condition assistant share one message thread, and the tree ticks in a loop with a throttling sleep until the condition node returns SUCCESS, at which point the fixed code is saved to disk.

## Reference Tables
| Node type | Role | Success condition |
|---|---|---|
| Selector (fallback) | Try children in order (OR) | First child success, else failure |
| Sequence | Run children in order (AND) | All children succeed |
| Condition | Boolean-like check via success/failure | Condition true |
| Action | Do the actual work | Work completed |
| Decorator | Gate/control a child node | Depends on wrapped logic (can block for safety) |
| Parallel | Run children concurrently | Configurable success threshold met |

| Alternative AI control system | Key shortcoming | Verdict for agentic AI |
|---|---|---|
| Finite state machine (FSM) | Unwieldy at scale | Not practical for agents |
| Decision tree | Overfitting/poor generalization | Can enhance behavior trees |
| Utility-based system | Needs careful utility design | Adoptable within a behavior tree |
| Rule-based system | Cumbersome, rule conflicts | Not practical with LLM agents |
| Planning system | Computationally expensive | Agents already self-implement this (later chapters) |
| Behavioral cloning | Poor generalization to unseen cases | Usable inside a specific task/node |
| Hierarchical Task Network (HTN) | Complex for very large tasks | Good for organizing large agentic systems |
| Blackboard system | Hard to manage cross-subsystem comms | Agent conversation/group chat mimics this |
| Genetic algorithm (GA) | Computationally intensive | Could optimize behavior trees themselves |

## Worked Example
The chapter builds a three-node coding-challenge ABT to solve an Edabit "Plant the Grass" simulation challenge: a Sequence root contains (1) the "Hacker" (Python Coding Assistant) action node that writes `solution.py` to solve the stated challenge, (2) the "Judge" action node (Coding Challenge Judge) that loads `solution.py`, runs it against the provided test cases, and if it passes saves `judged_solution.py` — both sharing one message thread; and (3) a separate "Verifier" condition node (a fresh Python Coding Assistant on its own isolated thread) that independently reloads `judged_solution.py`, reruns the tests, and returns the single word SUCCESS or FAILURE. The tree ticks every 20 seconds until the Verifier reports SUCCESS, demonstrating both the Hacker/Judge shared-thread conversational pattern and the Verifier's deliberately siloed isolation to avoid contaminating the independent check.

## Key Takeaways
1. Behavior trees give agentic systems modularity, scalability, debuggability, and clean separation of decision logic from execution — advantages classic FSMs and rule-based systems lack at agent scale.
2. ABTs = behavior trees where action/condition nodes are LLM-prompted assistants; expect more variability ("stochastic behavior trees") than classic deterministic trees.
3. Use Sequence nodes for AND-style multi-step pipelines (search -> summarize -> write -> post) and Selector nodes for OR-style fallback logic.
4. Build new ABTs via back chaining: start from the goal, work backward through required actions and conditions, then decide communication mode before constructing the tree.
5. Choose siloed threads for clean, isolated verification/testing; choose shared conversational threads when back-and-forth context (e.g., debug <-> verify) improves quality — mixing both patterns is often optimal.
6. File-based communication between ABT nodes is a simpler, more transparent substitute for the traditional blackboard pattern, at the cost of some structure.
7. Autonomous ABTs acting on real external systems (e.g., posting to X, searching YouTube) need explicit safeguards against spam/rate-limiting — this is a concrete, hands-on preview of the "guardrails" concern raised for autonomous agents in chapter 1.

## Connects To
- **Ch1**: implements "planning without feedback" vs. "planning with feedback" concretely — ABTs are a structured planning/feedback mechanism for the autonomous agent type introduced there.
- **Ch4**: contrasts with AutoGen/CrewAI's more emergent, less explicitly-structured multi-agent conversation patterns.
- **Ch11**: back chaining and structured plan construction here foreshadow the fuller planning-and-feedback strategies covered later.
- **Ch3/Ch5**: the GPT Assistants Playground builds directly on the GPT Assistants and Semantic Kernel-style action/tool patterns from earlier chapters, now via the raw OpenAI Assistants API.
