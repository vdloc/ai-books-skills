# Chapter 4: Computation: Designing for the Processing and Generation Phase

## Core Idea
Computation is the invisible middle stage between input and output — tokenization, routing, and token-by-token generation — and because AI errors often don't look like errors at all (a hallucination arrives "wrapped in perfect grammar... delivered with algorithmic confidence"), designers must design both for the failures the system can detect and the ones it can't.

## Frameworks Introduced
- **Three-step computation pipeline**: Input processing/preparation (tokenization, converting language to numeric vectors) → Routing (deciding which models/tools/retrieval paths handle the request) → Generation/inference (token-by-token or diffusion-step production of output).
  - When to use: whenever debugging "why did the AI give that answer" — pinpoint which of the three stages is responsible before designing a fix.
- **Three latency priorities**: "Make it fast where it must be" (real-time control, transactions), "make it engaging where it can be" (AI art, chatbots), "make it clear where it needs to be" (search, payments, learning tools).
  - When to use: allocating design effort across an AI product's different task types.
  - How: match the latency-mitigation technique to the task category, not a one-size-fits-all spinner.
- **Three-tier progress communication**: Tier 1 (Overview — minimal, always-visible status, e.g. stage + time elapsed), Tier 2 (Detail — clickable breakdown of subtasks and why they were chosen, for verification/trust), Tier 3 (Full record — complete retraceable log, for debugging/compliance/curiosity).
  - When to use: designing status UI for any multistep or agentic process.
  - How: notifications sit above all three as an abstract "still working" signal; each tier should be opt-in-able so users choose their own attention level.
- **Error-design principles** (adapted from HCI/Alan Cooper's *About Face*): make errors understandable (plain language, explain why not just that), recoverable (preserve work, no restart-from-scratch), prevent foreseeable errors via defaults/validation, degrade gracefully (partial functionality over total block), surface system status clearly (say if it's temporary/persistent, don't leave users guessing).
  - Critical AI-era addition: these principles only cover *detectable* errors. AI introduces **undetectable errors** (hallucinations) where the system itself doesn't know it's wrong — this shifts error-detection burden onto the user, requiring designed-in friction that prompts verification rather than error messages that assume the system knows something failed.

## Key Concepts
- **Tokenization**: breaking text into subword units mapped to numeric token IDs (e.g., "Summarize" → ["Summ", "arize"]) — done this way (not whole-word) so novel/rare words remain processable.
- **Latent space / vector embedding**: high-dimensional space (e.g., 3,072 dimensions) where semantically similar tokens/images sit close together — analogous to RGB color space but with vastly more axes of meaning.
- **Temperature**: sampling-randomness parameter — low temperature = deterministic/conservative/repetitive; high temperature = more varied/creative but more error-prone. Controls expressive range, not correctness.
- **Diffusion generation**: image models start from pure noise and iteratively refine toward the prompt — fundamentally different generation mechanic from LLMs' token-by-token improvisation, which affects how latency should be communicated (diffusion models can estimate duration more reliably).
- **Nielsen's latency thresholds**: 0.1s = feels instant (no feedback needed); 1s = flow stays uninterrupted (feedback optional); 10s = attention limit (progress indicators become critical beyond this).
- **Optimistic UI**: showing a confirmation before server response is finalized, used in transactional flows to maintain confidence.
- **Miller's Law**: humans hold ~7 items in working memory — cited as the reason learning-focused chatbot interactions need low latency and high wait-time transparency (competing cognitive load has less room for uncertainty).

## Mental Models
- Treat latency itself as a communication channel (Watzlawick's axiom: "it's impossible not to communicate") — silence during a wait *says something* (usually: something's wrong), so design the wait deliberately rather than treating it as dead time.
- The Apollo 11 "1202 alarm" is the chapter's central metaphor: a system working exactly as designed can look identical to failure when its internal logic is invisible — closing that gap (not just avoiding actual errors) is the design job.
- Segment latency design by *task cognitive profile*, not by absolute duration — a 2-minute wait during creative exploration feels fine; a 2-second wait during active learning can break flow, because working-memory load differs.

## Anti-patterns
- **Static spinners for long AI generation tasks**: negatively associated with perceived long wait — prefer progress bars, skeleton screens, or streaming when duration is even roughly estimable.
- **Large centered "big loader" latency for chat when reasoning/first-tokens are already visible**: redundant and theatrical once the interface already shows activity (streaming text, tool-call citations) — a subtler "still working" signal suffices.
- **Assuming AI "try again" behaves like form-validation retry**: unlike a corrected form field, retrying a well-formed AI prompt can yield an entirely different (better/worse/just different) output with no red-outline diagnostic — design explicit edit/retry/regenerate affordances instead of implying a fixable "error."
- **Treating hallucination-shaped output as a normal detectable error**: it isn't — the model doesn't know it's wrong, so no error message will fire; the design lever is friction that invites verification, not error recovery UI.
- **Losing all context on a midstream generation failure**: unlike deterministic form autosave, a cut-off generation can't be perfectly "resumed" — but prompt/prior messages/partial output must still be preserved so the user can build forward rather than restart from zero. Replit's Git-like "checkpoint" pattern is the model to follow.

## Reference Tables

| Task category | Latency expectation | Key technique |
|---|---|---|
| Information retrieval | Medium/slow tolerated if flow preserved | Progressive disclosure, predictive search/autosuggest |
| Real-time control | Instant | Immediate visual feedback, skeleton loaders, live cursors/avatars for connectivity |
| Transactional | Fast + confirmed | Optimistic UI, staged feedback ("Step 1 of 3: verifying card") |
| Conversational agents | ~2–3 seconds feels natural | Typing indicators, streamed partial responses, confirmation echoes |
| Creative generation | Tolerant if progress is visible | Progress bars over spinners, sequential status messages |
| Autonomous/agentic tasks | Tolerant if transparent | Periodic check-ins, incremental delivery, opt-out of constant approval |

| Loading indicator | Best for |
|---|---|
| Spinner (indeterminate) | Short, uncertain-duration tasks |
| Progress bar (determinate) | Estimable duration; risk if stalls/inaccurate |
| Skeleton screen | Predictable, structured content layouts |
| Real-time token streaming | LLM text generation — reduces perceived latency most directly |

## Worked Example
A user uploads a PDF and types "Summarize this document and make a draft presentation." Tracing the pipeline: **Input processing** — PDF parsed, text extracted, tokenized. **Routing** — request split across a document parser, a summarization model, a slide-generation system, and possibly a separate model for charts/graphics; the router also decides whether any sub-step needs external retrieval (e.g., today's date-sensitive facts) vs. relying on training knowledge. **Generation** — the summarization model produces text token-by-token (temperature governs how conservative vs. creative the phrasing is), while the slide generator may use a diffusion-style visual model for any generated imagery. Each hop is a separate potential failure point invisible to the user — which is exactly why Tier 1/2/3 progress communication (per-stage status → clickable breakdown → full retraceable log) matters more here than in a single-model chat reply.

## Key Takeaways
1. Decompose "why did the AI give that answer" into processing / routing / generation before designing any fix or explanation.
2. Match latency-mitigation technique to task type (real-time control needs near-zero delay; creative generation tolerates delay if progress feels meaningful) — don't apply one loading pattern everywhere.
3. Use the three-tier progress model (overview / detail / full record) so different users can self-select their attention level on multistep or agentic tasks.
4. Apply the five classic error-design principles (understandable, recoverable, preventable, graceful degradation, clear status) — but recognize they only cover errors the *system* can detect.
5. Hallucinations are undetectable-by-design errors — the mitigation is designed friction that invites verification, not error messaging (the model doesn't know it's wrong).
6. Preserve context on midstream failure (checkpoint patterns) since generative output can't be deterministically restored the way a crashed form submission can.

## Connects To
- **Ch 3**: this is the middle ICO stage, following directly from how inputs were captured.
- **Ch 5**: output design picks up immediately where generation ends — how results are formatted, verified, and made actionable.
- **Ch 6**: agentic/autonomous latency design (briefly introduced here) is expanded fully — planning, checkpoints, shared control.
