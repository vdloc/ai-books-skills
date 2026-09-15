# Chapter 4: Exploring Multi-Agent Systems

## Core Idea
Multi-agent systems split work across role-specialized agents that converse to complete and critique tasks — AutoGen favors flexible, conversation-driven proxy/group-chat patterns good for research and iteration, while CrewAI favors structured, role/task-defined crews (sequential or hierarchical) better suited to enterprise-style, controlled workflows; both need observability (e.g., AgentOps) because token costs and repetitive-thought loops spiral quickly.

## Frameworks Introduced
- **AutoGen (Microsoft)**: conversable agents that communicate via natural language, typically a `UserProxyAgent` <-> `ConversableAgent`/`AssistantAgent` pair, extendable to nested chats or `GroupChat`+`GroupChatManager`.
  - When to use: exploratory or research-style multi-agent coding tasks where flexible, implicit task delegation through conversation is enough.
  - How: `UserProxyAgent` executes code and gives/collects feedback each iteration until a termination condition (e.g., message ending in "TERMINATE"); add a critic agent via `register_nested_chats` for review loops; use `GroupChat`/`GroupChatManager` when nested/sequential hand-offs lose information (the "telephone game" problem).
- **AutoGen Studio**: a web UI/dev environment for AutoGen with a Skills builder (add Python functions like `describe_image` as tools) and a Playground for running tasks.
  - When to use: prototyping multi-agent tasks and skills without writing orchestration code first.
  - How: Build tab -> Skills -> New Skill (paste Python function) -> attach skill to an agent workflow -> run task in Playground.
- **CrewAI**: role-based agent "crews" with explicit `Agent` (role, goal, backstory, memory, delegation) and `Task` (description, expected_output, agent) objects, run via `Crew(agents, tasks, process=Process.sequential|hierarchical)`.
  - When to use: when you want explicit, controllable task/role structure (enterprise-style) rather than open-ended conversation; hierarchical processing when a manager agent should coordinate/delegate.
  - How: define each agent's role/goal/backstory and `allow_delegation`; define one `Task` per agent with an explicit `expected_output`; assemble into a `Crew` and call `crew.kickoff()`. Add a `manager_llm` for hierarchical process.
- **AgentOps observability**: a dedicated agent-tracing platform (works with CrewAI, extensible to others) tracking duration, prompt/completion tokens, cost, and a "Repeat Thoughts" plot.
  - When to use: any nontrivial multi-agent run — cost and repetition can spiral invisibly otherwise.
  - How: `pip install agentops` (or `crewai[agentops]`), get an API key, add one init line to the script, then inspect the dashboard for cost and repeated-thought loops.

## Key Concepts
- **Proxy communication**: one agent (the "waiter") relays user requests to worker agents ("kitchen") and returns results — AutoGen's basic pattern.
- **Conversable agent**: an AutoGen agent whose sole interface is natural-language chat.
- **Nested chat**: a sub-conversation (e.g., engineer <-> critic) triggered within a larger agent interaction, registered via `register_nested_chats`.
- **Group chat**: a shared conversation all agents see, coordinated by a `GroupChatManager`, avoiding the information-loss ("telephone game") problem of nested/sequential hand-offs.
- **Caching (AutoGen)**: `Cache.disk(cache_seed=...)` persists conversation progress to resume interrupted long-running tasks and reduce repeated token spend.
- **Sequential vs hierarchical processing (CrewAI)**: sequential iterates agents/tasks in order; hierarchical adds a manager LLM that coordinates/delegates among agents.
- **Repeat Thoughts**: an AgentOps metric showing when agents loop on the same reasoning without converging — a sign the task/role/process design needs adjusting.
- **Skills/actions over agent-written code**: the chapter's stated lesson — code agents write can break or go stale, so prefer giving agents callable skills/tools over having them generate throwaway code to "solve" problems.

## Mental Models
- Use AutoGen when the task benefits from flexible, implicit, conversational delegation (fast prototyping, research); use CrewAI when you need explicit role/task contracts and predictable structure (production-leaning, enterprise).
- Use hierarchical CrewAI processing only when a manager genuinely adds coordination value — the chapter shows it can double cost without improving output.
- Treat repetitive agent "thoughts" as a design smell, not a token-budget problem to just throw more calls at — fix roles/tasks/process instead.

## Anti-patterns
- **Letting agents write ad hoc code as their primary problem-solving strategy**: such code is brittle, needs maintenance, and can break as libraries change — prefer well-defined skills/tools/actions (theme continued into ch5).
- **Adding a manager/hierarchical layer reflexively**: the chapter's own hierarchical CrewAI experiment cost over double the sequential run with no significant quality gain — validate the need for coordination overhead before adding it.
- **Running agentic code-execution without isolation**: AutoGen explicitly recommends Docker for code-execution agents; running natively can leave orphaned windows/processes (esp. on Windows) or expose the host to unsafe generated code.
- **Ignoring cost/repetition observability**: without a tool like AgentOps, runaway token cost and unproductive repeated-thought loops go unnoticed until the bill or the wasted time arrives.

## Code Examples
```python
from autogen import AssistantAgent, UserProxyAgent, config_list_from_json

config_list = config_list_from_json(env_or_file="OAI_CONFIG_LIST")
user_proxy = UserProxyAgent(
    "user",
    code_execution_config={"work_dir": "working", "use_docker": False, "last_n_messages": 1},
    human_input_mode="ALWAYS",
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
)
engineer = AssistantAgent(
    name="Engineer",
    llm_config={"config_list": config_list},
    system_message="You are a profession Python engineer...write clean, well-structured code...",
)
critic = AssistantAgent(
    name="Reviewer",
    llm_config={"config_list": config_list},
    system_message="You are a code reviewer...output them as a list.",
)

def review_code(recipient, messages, sender, config):
    return f"Review and critque the following code.\n{recipient.chat_messages_for_summary(sender)[-1]['content']}"

user_proxy.register_nested_chats(
    [{"recipient": critic, "message": review_code, "summary_method": "last_msg", "max_turns": 1}],
    trigger=engineer,
)
task = "Write a snake game using Pygame."
res = user_proxy.initiate_chat(recipient=engineer, message=task, max_turns=2, summary_method="last_msg")
```
- **What it demonstrates**: the engineer/critic nested-chat pattern — a second agent persona automatically reviews the first agent's code output before the proxy accepts it, without the user manually relaying feedback.

## Reference Tables
| Platform | Communication model | Structure | Delegation | Best for |
|---|---|---|---|---|
| AutoGen | Free-form natural-language conversation | Implicit, emergent | UserProxy mediates | Research/prototyping, flexible coding tasks |
| AutoGen (group chat) | Shared conversation, manager-coordinated | Implicit but visible to all | GroupChatManager | Long-running/complex tasks needing shared context |
| CrewAI (sequential) | Role + explicit Task contracts | Explicit, ordered | Per-agent `allow_delegation` flag | Predictable, enterprise-style pipelines |
| CrewAI (hierarchical) | Role + Task + manager LLM | Explicit, manager-coordinated | Manager delegates | Only when coordination overhead is justified |

## Worked Example
The chapter builds the same "coding crew" task in both platforms for direct comparison: a user-specified game (e.g., Snake) is built by (AutoGen) an Engineer + Reviewer via nested chat, versus (CrewAI) three roles — Senior Software Engineer (`code_task`), Software QA Engineer (`qa_task`, checks imports/syntax/security), and Chief QA Engineer (`evaluate_task`, verifies the game does its job) — run sequentially via `Crew(agents=[...], tasks=[...], process=Process.sequential)`. Adding a hierarchical `manager_llm` version of the same crew is shown to roughly double AgentOps-measured cost without materially improving output, illustrating the chapter's core caution about coordination overhead.

## Key Takeaways
1. Multi-agent systems add power over single agents mainly through built-in mutual feedback/evaluation (critics, QA roles), not just parallelism.
2. AutoGen's strength is flexible conversational delegation (proxy, nested chat, group chat); CrewAI's strength is explicit, controllable role/task contracts.
3. Use group chat over nested/sequential chat when tasks are long-running or complex enough that information loss ("telephone game") becomes a risk.
4. Prefer giving agents skills/tools over having them freely generate code — generated code is fragile and hard to maintain across library/version drift.
5. Isolate code-execution agents (Docker/WSL) to avoid orphaned processes and unsafe code execution on the host.
6. Always attach observability (e.g., AgentOps) to multi-agent runs — cost and repetitive non-convergent "thoughts" are invisible otherwise and can spiral fast (the chapter's joke crew cost ~$0.50 per joke; hierarchical processing roughly doubled cost with no quality gain).
7. Don't add coordination complexity (hierarchical managers) by default — validate that it earns its overhead before adopting it.

## Connects To
- **Ch1**: formalizes the proxy + worker-agent multi-agent configuration sketched in section 1.1.
- **Ch5**: the "give agents skills/tools, not just code-writing ability" principle is expanded into a full actions/tool-use framework.
- **Ch7**: the Nexus platform later offers another take on assembling and orchestrating multi-agent systems, comparable to AutoGen/CrewAI here.
