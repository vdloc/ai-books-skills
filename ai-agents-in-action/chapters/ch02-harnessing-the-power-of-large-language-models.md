# Chapter 2: Harnessing the Power of Large Language Models

## Core Idea
Building agents starts with understanding LLMs as generative, chat-completion-style models accessed through a request/response pattern (system/user/assistant roles), and effective agent behavior begins with disciplined prompt engineering before any agent framework is layered on top.

## Frameworks Introduced
- **OpenAI Chat Completions Request Pattern**: `model` + `messages` (role-tagged) + `temperature`.
  - When to use: any time you call an LLM programmatically, whether commercial or local.
  - How: `messages` is a list of `{role, content}` dicts using `system` (rules/persona), `user` (the ask), `assistant` (history/injected examples); `temperature` near 0 for consistency, near 1.0 for variability.
- **OpenAI "Write Clear Instructions" Prompt Engineering Strategy** (6 tactics): detailed queries, adopting personas, using delimiters, specifying steps, providing examples, specifying output length.
  - When to use: as the default first strategy for any prompt before reaching for heavier techniques (RAG, chain-of-thought, etc. — covered later chapters).
  - How: apply one or combine several tactics in the system/user messages; test empirically since prompt engineering is not an exact science.
- **LLM Selection Criteria** (8 factors): model performance, model parameters/size, use case/model type, training input, training method, context token size, model speed/deployment, model cost.
  - When to use: when choosing between commercial (GPT-4 Turbo) and open-source/local models for a given agent.
  - How: weigh task-specific performance and context window needs against cost, privacy, and hardware constraints; chat-completions models are essential for iterative/reasoning agents (vs plain completion/instruct models).

## Key Concepts
- **Generative vs predictive/classification models**: generative models create new content from input; predictive models classify/predict a label.
- **RLHF (reinforcement learning with human feedback)**: the fine-tuning method that turns a base LLM into a chat-completions model good at iteration and reasoning.
- **System/User/Assistant roles**: the three message roles; assistant role can inject fake prior history (few-shot-by-conversation) even without a real conversation.
- **Temperature**: controls stochastic variability of output; 0 = deterministic-ish, 1.0 = most varied.
- **Token accounting**: `prompt_tokens` + `completion_tokens` = `total_tokens`; fewer tokens = cheaper, faster, often more consistent.
- **LM Studio**: a free local tool to download, run, and serve open-source LLMs (chat or server mode) with hardware compatibility checks.
- **Few-shot via examples**: injecting example Q/A pairs (often as fake assistant turns) to steer output format and style.

## Mental Models
- Use chat-completions models (not plain completion/instruct models) whenever the agent needs to iterate, reason, or converse — that's the model class this whole book depends on.
- Use temperature 0 for agent-to-agent or production pipelines needing reproducibility; raise it only for creative/exploratory single-shot use.
- Use local LLMs (via LM Studio) when data privacy, cost, or offline access matter more than raw performance; use commercial APIs (GPT-4 Turbo) as the default learning/production baseline otherwise.

## Anti-patterns
- **Vague, low-detail prompts**: under-specifying task and context degrades response quality — the "detailed queries" tactic exists specifically to counter this.
- **Ignoring token counts**: not tracking prompt/completion tokens leads to unexpectedly slow, expensive, or inconsistent agent behavior, especially at multi-agent scale.
- **Chasing "the best model" reflexively**: picking a model without weighing training input, context size, cost, and use case against the actual task wastes resources — smaller fine-tuned models can beat GPT-4 on narrow domains.

## Code Examples
```python
import os
from openai import OpenAI
from dotenv import load_dotenv
load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')
if not api_key:
    raise ValueError("No API key found. Please check your .env file.")
client = OpenAI(api_key=api_key)

def ask_chatgpt(user_message):
    response = client.chat.completions.create(
        model="gpt-4-1106-preview",
        messages=[{"role": "system", "content": "You are a helpful assistant."},
                  {"role": "user", "content": user_message}],
        temperature=0.7,
    )
    return response.choices[0].message.content

user = "What is the capital of France?"
response = ask_chatgpt(user)
print(response)
```
- **What it demonstrates**: the minimal end-to-end pattern every agent in the book builds on — load key from `.env`, construct client, send a role-tagged message list at a chosen temperature, extract `choices[0].message.content`.

## Reference Tables
| Prompt tactic | Core mechanism | Best for |
|---|---|---|
| Detailed queries | Add specificity/context to the ask | General accuracy improvement |
| Adopting personas | System message defines demographics/role | Consistent voice, "rubber ducking" through a specialist lens |
| Using delimiters | Triple quotes / XML tags isolate content | Separating instructions from data, hierarchical input |
| Specifying steps | Numbered Step 1/Step 2 instructions | Complex multi-stage tasks (summarize -> translate) |
| Providing examples | Few-shot via fake assistant turns | Enforcing output format/style |
| Specifying output length | "in 10 words" / "3 bullet points" | Concise, token-efficient, focused replies (esp. multi-agent chat) |

| LLM selection criterion | Why it matters for agents |
|---|---|
| Model performance | Task-specific benchmark fit (e.g., coding agent needs strong code performance) |
| Model parameters (size) | Inference quality vs hardware requirements |
| Use case (model type) | Chat-completions required for iterative/reasoning agents |
| Training input | Domain fit; smaller fine-tuned models can rival GPT-4 in-domain |
| Training method | Affects generalization/reasoning/planning ability |
| Context token size | Larger needed for multi-agent conversations |
| Model speed/deployment | Critical for real-time user-facing agents |
| Model cost | Tradeoff between self-hosting and commercial API |

## Worked Example
The chapter walks through `prompt_engineering.py`: it loads JSONL prompt files from a `prompts` folder (one file per tactic, e.g. `detailed_queries.jsonl`, `adopting_personas.jsonl`), lets the user pick a file, then loops through each prompt in it, sending each to the configured LLM (OpenAI by default, or a commented-out local LM Studio endpoint) and printing prompt + reply side by side. This lets a reader A/B test, e.g., "What is an agent?" (no detail) vs. "What is a GPT Agent? Please give me 3 examples" (detailed query) against the same model/temperature to see the tactic's effect directly.

## Key Takeaways
1. LLMs are generative transformer models; chat-completions variants (RLHF-tuned) are the class to use for agents because they iterate and reason well.
2. Every agent call reduces to model + role-tagged messages + temperature — master this pattern before adding frameworks.
3. Temperature is your primary lever for consistency (agents/pipelines) vs. variability (creative/exploratory use).
4. Track prompt/completion token counts — they drive cost, latency, and consistency, especially once you scale to multi-agent conversations (ch4).
5. Apply the six "Write Clear Instructions" tactics (detail, persona, delimiters, steps, examples, output length) as your default prompt-engineering toolkit before reaching for RAG or planning machinery.
6. LM Studio makes local/open-source LLMs a practical, privacy-preserving, cost-free alternative — use its compatibility check to match models to hardware.
7. Choosing a model is a multi-factor tradeoff (performance, size, type, training, context, speed, cost) — there's rarely one "best" model, only best-for-this-task.

## Connects To
- **Ch9**: prompt/model evaluation here is formalized later using Microsoft Prompt Flow.
- **Ch4**: token-efficient, concise replies matter even more once multiple agents converse with each other.
- **Ch3**: GPT Assistants build directly on the chat-completions request pattern introduced here, adding persistent threads and tools.
