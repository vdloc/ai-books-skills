# Chapter 1: Understanding Large Language Models and Systems

## Core Idea
Users bring inaccurate mental models (human-like understanding, factuality, consistency, complete knowledge, unbiased output) to AI products because LLMs are fundamentally statistical, not cognitive — designers must actively manage the gap between what users assume and what the system actually does.

## Frameworks Introduced
- **Input-Computation-Output (ICO) framework**: the three-stage structure the whole book is organized around.
  - When to use: as the default lens for diagnosing *any* AI UX problem — "is the input unclear, is computation opaque, or is the output unusable?"
  - How: **Input** — where human intention enters (explicit text, or implicit/inferred context); **Computation** — the opaque black box (interpretation, retrieval, reasoning, tool orchestration, guardrails); **Output** — the formatted result returned to the user (text/image/chart/audio, plus citations and interactivity). Before Input sits *configuration* (accounts, defaults); after Output sits *interpretation/action* (user verifies, edits, integrates) — the loop restarts.
- **AI Organizational Readiness levels** (adapted from the SAE autonomous-driving 0–5 scale): a 3-level maturity model for how AI knowledge flows inside a company.
  - When to use: diagnosing why a design team keeps getting blindsided by model changes, or auditing whether an org is ready to ship AI features responsibly.
  - How: **Level 1 (Individual exploration)** — no shared owner for model behavior, evals, or responsible-AI; features arrive top-down. **Level 2 (Shared understanding, manual maintenance)** — PMs/engineers steward knowledge, evals have an owner, but distribution is manual and ad hoc. **Level 3 (Integrated AI organization)** — information flows reliably, evaluation and responsible-AI work has a stable home, capability shareouts run on a steady cadence, features originate from validated user research rather than hype.

## Key Concepts
- **Hallucination**: fluent, confident, generated content unsupported by real data.
- **ELIZA effect**: the tendency to project humanlike understanding onto text that only simulates it.
- **Transformer/attention**: 2017 architecture letting a model weigh every word's relevance to every other word in parallel (vs. sequential processing).
- **Constitutional AI**: training method (used by Anthropic) that uses a written set of principles instead of only human feedback to shape model behavior.
- **Stochastic parrot** (Bender et al. 2021): critique that scale creates the illusion of understanding while merely recombining training-data patterns, amplifying bias and concentrating power.
- **AI winter**: funding collapse (1970s) after rule-based symbolic AI failed to scale to real language complexity.
- **AGI (artificial general intelligence)**: media/marketing framing that inflates user expectations disconnected from real model capability.

## Mental Models
- Think of a user's prior software experience (search engines, chatbots, sci-fi AI) as a *lens they involuntarily apply* to your product — your job is to correct the lens, not just build features.
- Use the **three intersecting pressures** (business urgency to ship, hype-driven expectations, absence of regulatory standards) as the root-cause explanation whenever a shipped AI feature "solves a problem the user doesn't have."
- Treat **personas** in AI products not as marketing artifacts but as tools for anticipating *misinterpretation* — what will this type of user assume the system can do, and where will that assumption break.

## Anti-patterns
- **Interface hides complexity entirely**: clean, minimal, conversational UI reinforces the illusion of a thinking being and obscures the probabilistic reality — users stop fact-checking.
- **One-size-fits-all interface for open-ended generative use cases**: flattens genuinely different user goals (learning vs. writing vs. ideation) into one experience, causing disorientation.
- **No designated owner for model-capability updates**: at Level 1 maturity, capability shifts (sometimes weekly) reach no one reliably, so design decisions are made on stale assumptions.
- **Treating ethics as purely an engineering problem**: designers who disclaim all responsibility for hallucinations/bias miss their unique leverage point — proximity to the user experience.

## Worked Example
Google's "Startographer" project (author's own case): to explain how an NLP embedding model worked, the team built a game where users typed a word to navigate toward a target word based on embedding-space proximity (e.g., typing "lifeform" to approach a planet labeled "biology," scoring 90% association). The design challenge wasn't the game mechanic — it was translating an abstract technical capability (vector-space proximity) into something a non-technical user could feel and understand intuitively. Lesson generalized: when a model's real capability is illegible to users, look for a *spatial or comparative metaphor* the interaction itself can teach, rather than an explanation the user must read.

## Key Takeaways
1. Every AI UX failure decomposes into an input, computation, or output problem — use ICO as the first diagnostic question.
2. Users' five default misassumptions (human-like understanding, factuality, consistency, complete knowledge, unbiased output) are predictable — design defenses against each rather than treating confusion as user error.
3. Designers don't need ML-researcher depth — enough technical fluency to explain behavior, anticipate failure, and set expectations is sufficient; depth accrues through building/prototyping with real models, not theory.
4. Ethical responsibility for AI splits three ways (social/interpersonal, safety/security, environmental) — designers own the interface-level mitigations in each, not the underlying systemic fixes.
5. Organizational maturity (the 3-level model) is often the actual constraint on good AI design, not individual designer skill — diagnose the org before diagnosing the interface.

## Connects To
- **Ch 2**: capability/discovery/orchestration — what must exist before a user ever types.
- **Ch 5**: revisits the "search for outcomes, not text" principle introduced here when discussing output actionability.
- **External**: DeepMind's 2021 "ethical and social risks of harm from language models" taxonomy; Bender et al. "On the Dangers of Stochastic Parrots"; SAE J3016 autonomous-driving levels (the direct model for the org-readiness framework).
