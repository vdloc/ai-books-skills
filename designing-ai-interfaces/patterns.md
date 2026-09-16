# Patterns & Techniques

## Discovery 2×2 (Intent × Initiative)
**When to use**: deciding how to surface a new AI feature.
**How**: plot on axes of user intent (do they already know they want this?) and system initiative (does the tool act proactively or wait?). Low/low → let it emerge socially. Low intent/high initiative → needs strong context cues, risk of feeling intrusive. High intent/low initiative → make discovery/activation frictionless. High/high → just remove friction and accelerate.
**Trade-offs**: over-relying on ambient/proactive discovery for low-intent users risks feeling intrusive; under-using it for high-initiative-appropriate features leaves genuinely useful capability undiscovered.

## CARE Prompt Scaffolding
**When to use**: building any prompt-assist UI (templates, guided forms, prompt-improvement suggestions).
**How**: elicit Context (who/why/audience/constraints), Action (specific processing wanted, not vague "look at this"), Results (desired format/length/tone/audience), Examples (concrete samples of the target style).
**Trade-offs**: fully guided CARE forms reduce failure but add friction for expert users who'd rather free-type; hybrid interfaces (structured fields + free text) serve both.

## Three-Tier Progress Communication
**When to use**: any multistep or agentic process where users need to track status.
**How**: Notifications (abstract, "still working" — for when attention is elsewhere) → Tier 1 Overview (minimal, stage + time elapsed) → Tier 2 Detail (clickable breakdown, why steps were chosen, for verification) → Tier 3 Full Record (complete retraceable log, for debugging/compliance).
**Trade-offs**: showing full detail by default overwhelms casual users; showing only notifications loses power users who need to verify or debug.

## Reveal-the-Plan
**When to use**: before any agentic system begins multistep or costly execution.
**How**: show interpretation (decomposed steps, brief summary, or outline) after the request but before work starts, while correction is still cheap. Scale visibility to task stakes — trivial tasks need no visible plan; complex/expensive/hard-to-undo ones do.
**Trade-offs**: showing a plan for every trivial request adds ceremony and friction; the sweet spot is calibrating visibility to task complexity/cost.

## Checkpoint / Rollback System
**When to use**: any agentic workflow producing evolving artifacts (documents, code, dashboards).
**How**: pause at meaningful boundaries (plan generated, dataset analyzed, irreversible action pending) to show progress and request approval. Rollback restores a prior checkpoint without deleting current state (creates a new branch/snapshot instead). Intermediate outputs are labeled provisional and linked to the step that produced them.
**Trade-offs**: too many checkpoints slow momentum and feel like constant interruption; too few risk losing user trust when the agent goes off-course with no recovery point.

## Tiered Permission Model
**When to use**: agentic tool use requiring access to files, calendars, external services.
**How**: classify tool risk (read-only / reversible-modification / destructive-or-irreversible) and require approval proportional to risk. Offer persistent "don't ask again" scoped appropriately (per-session, per-project) rather than re-prompting every step.
**Trade-offs**: over-broad persistent grants reduce friction but weaken oversight; per-step confirmation maximizes control but is unsustainable for high-volume agentic tasks.

## Grounding Signals ("Why we suggested this")
**When to use**: whenever output correctness depends on an unstated assumption (jurisdiction, date, mode, model version).
**How**: attach a lightweight, dismissible affordance (inline, hover, or info-icon) disclosing model/version, agent responsible, locale, task mode, and referenced session context.
**Trade-offs**: exposing grounding for every response adds visual noise; reserve prominent grounding for outputs where the hidden assumption materially changes correctness (legal, medical, financial, location-based).

## Structured Output Formatting Contract
**When to use**: any generative text output meant to be scanned, acted on, or trusted (docs, guides, reports).
**How**: give the model an explicit template (Title → Introduction → Sections [H2] → Subsections [H3] → Lists → Notes/Tips → Conclusion → Sources), a tone spec, a completeness heuristic ("prioritize readability over exhaustive detail"), and an explicit missing-data fallback ("output 'not available' rather than inventing").
**Trade-offs**: over-rigid templates suppress model flexibility and can raise hallucination rates on messy inputs, and outputs can start to feel mechanical/scripted — leave room for natural variation where stakes are low.

## Latency-by-Task-Category
**When to use**: designing loading/wait UI for any AI feature.
**How**: match the mitigation technique to task type — real-time control needs near-instant visual feedback + skeleton loaders; information retrieval needs progressive disclosure of early results; transactional needs optimistic UI + staged confirmation; conversational agents need typing indicators + streamed partial responses (~2-3s feels natural); creative generation tolerates delay if progress bars (not static spinners) show meaningful advancement; autonomous/agentic tasks need periodic check-ins and incremental delivery.
**Trade-offs**: applying the same latency pattern everywhere either annoys real-time-control users (too slow-feeling) or over-engineers simple transactional confirmations (too much ceremony).

## Verifiability-First Confidence Design
**When to use**: any output making an objective, actionable, or high-stakes claim.
**How**: ask "can the user independently validate this?" instead of showing a raw model confidence percentage. If yes (or low-stakes/subjective) — leave as is. If no and high-stakes — attach traceable sources, side-by-side comparisons, or route to human review; avoid presenting as authoritative (hedge: "this seems to be true").
**Trade-offs**: adding verification friction to every output slows low-stakes creative/subjective tasks where usefulness (not correctness) is the real benchmark — reserve the heavier verification UI for objective/high-stakes claims.

## Canvas for Iterative Co-creation
**When to use**: outputs meant to be edited, revisited, and built upon across sessions (documents, designs, generated code/images).
**How**: provide an editable, spatially organized, persistent, multimodal workspace with version history (Claude's artifact versions, Runway's node-based interface) distinct from linear ephemeral chat.
**Trade-offs**: canvas overhead (state management, version UI) is unnecessary for single-shot Q&A — reserve for genuinely iterative, multi-session work.

## Reflection with Calibrated Transparency
**When to use**: high-stakes generative tasks (code, financial calculations, compliance content) where self-checking materially improves reliability.
**How**: run a generate → critique (self or separate critic agent) → regenerate loop; expose only the final refined output by default, with an optional summary of what was checked/changed available on demand (progressive disclosure, e.g., collapsed Chain-of-Thought).
**Trade-offs**: reflection adds real latency (a 5s task can become 15s) that must be explained via status messaging, or users will suspect something is wrong.
