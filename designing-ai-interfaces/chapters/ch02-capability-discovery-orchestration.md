# Chapter 2: Capability, Discovery, and Orchestration

## Core Idea
AI features fail (like Google Wave) not from lack of capability but from a capability-first mindset — successful AI products design outward from **intent** (what the user wants), through **discovery** (how they learn what's possible) and **orchestration** (the config layer connecting intent to capability), rather than starting from "what can the model do."

## Frameworks Introduced
- **Intent / Capability / Discovery / Orchestration model**: the chapter's core vocabulary.
  - When to use: framing any AI feature kickoff conversation, to force "what does the user want" before "what can we build."
  - How: Intent = the user's goal; Capability = what the model can actually do; Discovery = what the user knows is possible and how they learn it; Orchestration = the UX/systems layer connecting intent to capability productively.
- **Discovery 2×2 matrix (User Intent × System Initiative)**: classifies which discovery pattern fits a feature.
  - When to use: deciding *how* to surface a new AI feature.
  - How: axes are "does the user already know they want this?" (intent) and "does the tool act proactively or wait?" (initiative). Low intent + low initiative → feature emerges socially/organically. Low intent + high initiative → needs strong context cues to avoid feeling intrusive. High intent + low initiative → make discovery/activation frictionless (the user is already looking). High intent + high initiative → just remove friction and accelerate.
- **Model → Tool → Agent capability stack** (Maslow-style hierarchy): models are the reactive foundation (respond to a prompt, no goal-pursuit); tools extend the model into the world (search, read, query — user still directs each step); agents add planning autonomy on top of both (interpret a goal, break into subtasks, choose tools, adapt on failure).
  - When to use: deciding what kind of interface affordances a feature needs — model-tier needs input/output feedback; tool-tier needs permissioning/confirmation/visibility; agent-tier needs pause/redirect controls and trust-building around unsupervised action.
- **Three Orchestration principles**: intentional (present choices with clear stakes at meaningful moments), transparent (surface what changed — model switch, new permission, new plugin), recognizable (users should notice change and be able to reorient after time away; mental model stays stable until user changes it).

## Key Concepts
- **Pretraining vs. fine-tuning**: pretraining = next-word prediction over massive general text; fine-tuning = smaller targeted dataset shaping instruction-following, often via RLHF (reinforcement learning from human feedback).
- **Evals (MMLU, GSM8K, MT Bench)**: MMLU = 57-topic multiple-choice general knowledge; GSM8K = grade-school multi-step math word problems (tests structured reasoning); MT Bench = human-judged comparison on realistic tasks (catches tone/nuance automated tests miss).
- **Vibe coding** (Karpathy, Feb 2025): AI-assisted dev where you accept generated code without review, debug by pasting errors back to the LLM — fine for throwaway prototypes, risky for production.
- **Momentum behavior** (Nielsen Norman Group): users stick to a chosen workflow path even when better options exist, because weak interface signaling failed to call attention to alternatives at the right moment.
- **Algorithmic transference**: users carry assumptions/frustrations from earlier AI interactions into new ones, even when no longer applicable — worsened by how fast AI capabilities change (the "plastic state").
- **Hick's Law**: decision time increases with number/complexity of choices — cited re: the ~120-second onboarding window before users abandon (2017 Clutch study: 72% cite <1 min setup as a retention factor).

## Anti-patterns
- **Capability-first product development** ("what features should we build" instead of "what are users trying to accomplish") — the Google Wave failure mode: powerful tech, no legible entry point.
- **Invisible automatic model routing with no signal**: if the system silently swaps models per-query, users can't explain variance in speed/quality/capability — always show a persistent badge naming the active model and its benefit.
- **Binary all-or-nothing file/data permissions**: forces users into a choice that doesn't match how they actually think about data sensitivity; use granular scopes (this conversation / this project / everything).
- **Metering with no upfront disclosure**: if quota exhaustion isn't visible before it happens, users get penalized twice — once by the interruption, again by having to reconstruct lost context (also: preserve session state on quota-out so users can resume).
- **Confusing "agent" with "chatbot" in product language**: ambiguous internally and externally; prefer "agentic" to signal planning capability distinct from a conversational bot.

## Reference Tables

| Layer | User controls each step? | Design focus |
|---|---|---|
| Model | Yes — every turn | Input/output formatting, generation feedback |
| Tool | Yes — user initiates, tool executes | Permissioning, confirmation, result visibility |
| Agent | No — agent plans across multiple steps | Show current action, allow pause/redirect, build trust in unsupervised action |

| Metering pattern | Example | Trade-off |
|---|---|---|
| Token/credit conversion to a friendly unit | Visual Electric's "volts" | Softens cost anxiety, still needs upfront per-query disclosure |
| Session-count caps | ChatGPT deep-research session limits | Simple to communicate, needs visible remaining-count |
| Silent backend metering | No visible unit at all | Lowest friction UI, highest risk of surprise interruption |

## Worked Example
A single prompt — *"I have a client meeting tomorrow with Salesforce about integrating our CRM data. Can you help me prepare?"* — walked through all three capability layers: **Model only** → generic advice (review background, talking points, security/API questions). **Model + tools** → looks up Salesforce's latest announcements, pulls calendar details, finds internal case studies, checks attendee list, tailors content — user still guides each step. **Agent** → blocks calendar time, drafts a presentation, compiles stakeholder list with technical roles, creates review reminders, generates follow-up email variants for different outcomes, and keeps refining as new information arrives. The lesson: the *same user request* demands three entirely different interfaces depending on which capability layer answers it — decide the layer before designing the UI.

## Key Takeaways
1. Diagnose feature ideas by asking "what does the user want to accomplish" before "what can the model do" — capability-first thinking produces Google-Wave-style orphaned features.
2. Use the Discovery 2×2 (intent × initiative) to pick a surfacing pattern instead of defaulting to a sparkle-emoji icon for every feature.
3. Model/Tool/Agent are a dependency stack, not synonyms — each layer needs different UI (feedback vs. permissioning vs. pause/redirect trust controls).
4. Silent automatic routing (models or tools) erodes trust — always expose what changed via a visible, low-friction signal.
5. Metering and permissions should be granular, revocable, and disclosed *before* the user hits a limit, with session state preserved across interruptions.
6. Momentum behavior means users rarely reconsider workflows on their own — proactive, well-timed prompts matter more than passive discoverability for driving adoption of better paths.

## Connects To
- **Ch 1**: builds directly on the ICO framework — this chapter covers everything that must exist *before* Input.
- **Ch 6**: agent design (introduced briefly here) gets full treatment — planning patterns, multiagent coordination, shared control.
- **External**: Model Context Protocol (MCP) as the emerging standard for tool-capability description; Yang et al. (CMU HCII, 2020) on designer difficulty grasping AI capability bounds.
