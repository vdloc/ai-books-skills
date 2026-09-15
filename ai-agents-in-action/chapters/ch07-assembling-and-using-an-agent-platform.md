# Chapter 7: Assembling and Using an Agent Platform

## Core Idea
Nexus — the author's own open-source teaching platform built on Streamlit — assembles the book's agent components (profile/persona, actions/tools, and eventually knowledge/memory and planning/feedback) into a single pluggable system, demonstrating with real code how a `BaseAgent` abstraction, a YAML-based profile system, and a decorator-based action system fit together into a working, extensible agent host.

## Frameworks Introduced
- **Nexus Agent Platform**: a Streamlit-based agent host with pluggable profiles (YAML), actions (decorated Python functions), and swappable agent engines.
  - When to use: as a learning/reference architecture for assembling your own agent platform, or directly as a lightweight platform for prototyping agents with configurable persona + tools.
  - How: drop a YAML profile into `nexus_profiles/`, drop `@agent_action`-decorated functions into `nexus_actions/`, subclass `BaseAgent` in `nexus_agents/` for a new model/engine — Nexus auto-discovers all three via its plugin/folder-scanning system.
- **Streamlit Chat App Pattern**: session-state-driven, rerun-on-interaction chat UI.
  - When to use: rapid prototyping of any LLM chat/dashboard interface in pure Python.
  - How: use `st.session_state` for anything that must persist across Streamlit's full-script reruns (model choice, message history); render history in a loop with `st.chat_message`; capture input via `if prompt := st.chat_input(...)`; for perceived responsiveness use `stream=True` on the API call plus `st.write_stream(stream)` instead of blocking on the full completion.
- **`BaseAgent` Abstraction**: an engine-agnostic interface (`get_response`, `get_semantic_response`, `get_response_stream`, `load_chat_history`, `load_actions`) that concrete engines (e.g., `OpenAIAgent`) implement.
  - When to use: whenever you want a platform to support multiple LLM providers/toolkits (OpenAI, SK, Claude, Gemini) behind one consistent agent interface.
  - How: subclass `BaseAgent`, implement the abstract methods against your chosen provider's SDK, and drop the module into the engine plugin folder for auto-discovery.
- **Decorator-Based Action Definition (`@agent_action`)**: turn any Python function — native (real code) or semantic (docstring-only prompt) — into an LLM-callable tool automatically.
  - When to use: the fastest way to add a tool to an agent without hand-writing an OpenAI tool JSON schema.
  - How: decorate a function with `@agent_action`; its docstring becomes the tool `description` (for semantic functions, the docstring *is* the prompt template with `{{placeholders}}`, and the function body is `pass`); its parameters/type hints are introspected into the OpenAI tool JSON schema automatically.

## Key Concepts
- **Profile/persona (Nexus)**: a YAML file (e.g., `fiona.yaml`) defining an agent's system prompt/personality; auto-discovered from `nexus_profiles/`.
- **Agent engine**: the concrete implementation powering a profile against a specific LLM/toolkit (currently only `OpenAIAgent` in the book's version of Nexus).
- **Native vs. semantic function (Nexus)**: native = real executable code; semantic = a docstring-as-prompt-template function whose body is `pass` — mirrors Semantic Kernel's native/semantic function split from chapter 5, but with less ceremony.
- **Parallel function/tool calling**: the OpenAI agent engine executes all tool calls returned by one LLM turn together (no ordering), then makes a second call with all results appended — ordered/sequenced tool execution is deferred to planners in chapter 11.
- **Streamlit session state**: per-browser-session, in-memory state that resets on browser close; not a substitute for a real database for persistent history.
- **Plugin/folder-scanning discovery**: Nexus's mechanism for finding profiles, actions, and agent engines — simply place a correctly structured file in the right folder.

## Mental Models
- Treat "profile + actions + (later) memory + (later) planning" as the concrete implementation checklist for any agent platform you build — Nexus is a working reference for the five-component model from chapter 1.
- Use Streamlit's rerun-on-interaction model correctly: anything that must survive a rerun (history, config) belongs in `st.session_state`, not local Python variables.
- Prefer streaming (`st.write_stream`) over spinners for any user-facing chat UI — it meaningfully improves perceived responsiveness at near-zero extra code.

## Anti-patterns
- **Giving an agent unrestricted access to all available actions**: the chapter notes that, as of writing, Nexus profiles do not yet restrict which actions an agent can select from — an agent with tools it doesn't need increases hallucination/misuse risk (echoing the Manager Assistant caution in ch6).
- **Relying on Streamlit session state as durable storage**: session state disappears when the browser session ends — use a real database for anything that must persist (Nexus's chat system explicitly layers a database underneath for this reason).
- **Hand-writing OpenAI tool JSON schemas when a decorator can generate them**: manual schema authoring (as done raw in ch5) is more error-prone and slower to iterate than function-introspection-based generation.

## Code Examples
```python
from nexus.nexus_base.action_manager import agent_action

@agent_action
def get_current_weather(location, unit="fahrenheit"):
    """Get the current weather in a given location"""
    return f"The current weather in {location} is 0 {unit}."

@agent_action
def recommend(topic):
    """
    System: Provide a recommendation for a given {{topic}}.
    Use your best judgment to provide a recommendation.
    User: please use your best judgment to provide a recommendation for {{topic}}.
    """
    pass
```
- **What it demonstrates**: Nexus's unified native/semantic action pattern — the first function executes real code and returns a string result; the second has no code body at all (`pass`) and instead uses its docstring as a semantic (prompt-template) function; both are auto-converted into valid OpenAI tool specifications purely from the decorator plus function signature/docstring.

## Reference Tables
| Nexus component | Folder | Auto-discovery trigger |
|---|---|---|
| Profile/persona | `nexus_base/nexus_profiles/` | Any valid YAML file |
| Action/tool | `nexus_base/nexus_actions/` | Any function decorated with `@agent_action` |
| Agent engine | `nexus_agents/` | Any class subclassing `BaseAgent` |

| Chat UI approach | User experience | Code complexity |
|---|---|---|
| Blocking call + `st.spinner` | Wait, then full response appears | Minimal |
| `stream=True` + `st.write_stream` | Tokens appear progressively | Minimal (near-identical code) |

## Worked Example
The chapter walks through creating a new persona end to end: (1) author `fiona.yaml` in `nexus_profiles/` giving the agent an ogre-inspired persona; (2) launch Nexus in debug mode via a Streamlit `launch.json` config; (3) start a new chat thread, select the `OpenAIAgent` engine and the new "Fiona" persona; (4) ask her to spell "clock" and observe the system prompt driving an in-character (ogre-speak) reply — demonstrating that the profile YAML alone, with no code changes, fully determines the agent's voice. A second worked example wires up the `recommend` (semantic) and `get_current_weather` (native) actions together with the terse "Olly" persona, showing the OpenAI agent engine detect two parallel `tool_calls` in one LLM turn, execute both, append both results as `tool`-role messages, and produce one synthesized terse reply referencing both.

## Key Takeaways
1. Nexus operationalizes the five-component agent model (ch1) as a real, extensible codebase: profiles, actions, and (in later chapters) memory and planning are all pluggable via folder-based discovery.
2. Streamlit's full-script-rerun model requires disciplined use of `session_state` for anything that must persist across interactions — get this wrong and state silently vanishes.
3. Streaming responses (`stream=True` + `st.write_stream`) is a nearly free upgrade to perceived agent responsiveness.
4. A `BaseAgent` abstraction (get_response / get_semantic_response / get_response_stream / load_chat_history / load_actions) is the right seam for supporting multiple LLM providers/toolkits behind one platform.
5. Decorator-based action definition (`@agent_action`) collapses the OpenAI tool-schema boilerplate from chapter 5 into a single decorator plus docstring, for both native and semantic functions.
6. The OpenAI agent engine executes tool calls in parallel/unordered within a turn; sequenced/ordered tool execution requires a planner (deferred to ch11).
7. Action access is currently unrestricted per profile in Nexus — a known gap worth guarding against by convention (goal-specific action sets) until the platform enforces it.

## Connects To
- **Ch1**: Nexus is the concrete implementation of the five-component agent model introduced there.
- **Ch5**: Nexus's native/semantic action split directly mirrors Semantic Kernel's native/semantic function distinction, simplified via decorators.
- **Ch8**: Nexus's knowledge/memory component (mentioned but not yet implemented here) is built out using RAG in the next chapter.
- **Ch11**: parallel (unordered) tool calling here is contrasted with the ordered/sequenced tool execution planners introduce later.
