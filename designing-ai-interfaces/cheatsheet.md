# Cheatsheet

## Decision rules

- **When a feature idea starts from "what can the model do"** → stop, reframe as "what is the user trying to accomplish" (Google Wave failure mode). Capability-first features orphan users.
- **When deciding whether to show a confidence score** → never show a bare percentage. Ask "can the user independently verify this?" If no + high-stakes/objective → route to human review or citations. If yes or low-stakes/subjective → confidence is irrelevant, skip it.
- **When an output depends on a hidden assumption (jurisdiction, date, model version, mode)** → always ground it visibly. Don't let users discover the assumption was wrong (the Vegas-wedding-by-Elvis case: correct-sounding, wrong-jurisdiction legal answer).
- **When a task exceeds ~10 seconds** → progress indicators become mandatory (Nielsen's threshold). Below 1 second, no feedback needed. Below 0.1 second, feels instant.
- **When an agent is about to take a costly, hard-to-undo, or complex multistep action** → reveal the plan before execution. Correction before work starts is cheap; after, it's expensive.
- **When a tool/agent action is destructive or irreversible** → require explicit approval every time. When it's read-only → never ask. When it's reversible-but-consequential → allow persistent "don't ask again" scoped to session/project.
- **When more than ~3 sentences of context could help a prompt** → structure it with CARE (Context, Action, Results, Examples) rather than dumping unstructured detail — but stop before "lost in the middle" territory; don't pad prompts hoping more detail always helps.
- **When output is meant to be edited/revisited across sessions** → put it on a canvas, not in chat. When it's single-shot Q&A → chat is sufficient.
- **When a ReAct-style agent loop has no fixed number of steps** → never fake a determinate progress bar ("Step 3 of 5"). Show iteration count or elapsed time instead.
- **When restoring a prior checkpoint/rollback** → never delete the current state. Always branch/snapshot.
- **When an AI-mimicking-human interface interacts in a commercial/political context** → disclosure is legally required in some jurisdictions (CA SB 1001, EU AI Act) — never skip it, regardless of how fluent the interaction feels.

## Trade-off matrices

| Decision | Favor A when... | Favor B when... |
|---|---|---|
| Visible plan (A) vs. silent execution (B) | Task is complex, costly, or hard to undo | Task is quick, low-stakes, routine |
| Persistent permission (A) vs. per-step confirmation (B) | High-volume routine agentic tasks, trusted user | New user, destructive/irreversible action type |
| Structured template output (A) vs. flexible/loose (B) | Task needs scannability, consistency, high stakes | Creative/subjective task where mechanical feel hurts trust |
| Screen/browser visualization (A) vs. layered text status (B) | Teaching capability, building initial trust, audit requirement | Long-running task where user needs to step away and return |
| Canvas (A) vs. chat (B) | Iterative, multi-session, multimodal work | Single-shot Q&A, ephemeral exchange |

## Thresholds & defaults

- **0.1s** — feels instantaneous, no feedback needed.
- **1s** — flow stays uninterrupted; feedback optional.
- **10s** — attention limit; progress indicators become critical.
- **~120 seconds** — onboarding abandonment threshold (72% of users cite <1 min setup as a retention factor; Clutch 2017).
- **~2-3 seconds** — feels like natural conversational rhythm for chatbot replies (too fast = robotic, too slow = breaks the dialogue illusion).
- **7 items** — Miller's Law, working-memory capacity; a rough ceiling for how much a user can hold in mind during learning-focused tasks that also involve AI latency.
- **~20 percentage points** — GPT-4.1 legal-reasoning accuracy improvement from Markdown-structured vs. unformatted input (Braun/Lilienbeck/Mentjukov study) — formatting is a legibility signal to the model, not just to humans.
- **26% / 9%** — OpenAI's 2023 AI-text classifier's true-positive / false-positive rate before being shut down — a concrete data point for why detection-based approaches (vs. generation-time watermarking) remain unreliable.

## Tells & smells

- **"The output sounds too perfect / mechanical"** → likely over-rigid output templating suppressing natural model variation; loosen structural constraints.
- **User keeps clicking "try again" hoping for a fix** → sign the interface implied there was something "wrong" to correct, when generative retry just produces a different, not necessarily better, result. Offer explicit edit/regenerate framing instead.
- **A feature nobody discovers despite being genuinely useful** → likely mismatched discovery posture (check the Discovery 2×2 — is initiative/intent misjudged?).
- **Users express frustration despite "average" latency numbers** → check if delays are unexplained; users tolerate long waits when they understand why, and get frustrated at short unexplained ones (Watzlawick: silence itself communicates).
- **A model's confident, fluent, well-formatted answer turns out to be wrong** → classic hallucination signature — grammatically perfect, formatted correctly, delivered with unwarranted confidence. This is the error class the system itself cannot detect; only designed friction (verification prompts, sourcing) catches it.
- **Multiple agents in a pipeline produce silently contradictory results** → a conflict-resolution gap; surface the disagreement to the user as a decision point rather than letting one agent's output silently win.
- **A new user needs approval for everything, an expert user is annoyed by constant confirmation** → your permission/autonomy boundary isn't adjustable per user trust level; make shared-control boundaries explicit and tunable.
