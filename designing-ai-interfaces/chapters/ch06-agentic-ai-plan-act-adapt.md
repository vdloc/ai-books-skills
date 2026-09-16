# Chapter 6: Agentic AI: Designing for Systems That Plan, Act, and Adapt

## Core Idea
Clippy (1997) had the right vision — proactive computer assistance — but failed because it lacked transparency, couldn't adapt, and acted on its own timing rather than the user's; today's agentic systems finally have the technical capability Clippy lacked, but only succeed if their interfaces reveal plans, prioritize what matters, and design for shared control rather than silent autonomy.

## Frameworks Introduced
- **Five agentic design patterns**: Reflection (self-evaluation/error correction), Tool Use (extending agents with external capabilities), Planning (breaking goals into steps), Multiagent Collaboration (distributing work among specialists), ReAct (Reason+Act — adaptive interleaving of reasoning and action).
  - When to use: as the engineering vocabulary for what an agentic system is actually doing — not mutually exclusive; effective systems combine several (e.g., ReAct for exploration + Reflection to check each step + Tool Use to act + Multiagent for specialized subtasks).
- **Three New Principles for Agentic Interface Design**: **Reveal the Plan** (show interpretation before execution, while it's still cheap to correct), **Prioritize What Matters Most** (layered information — most users need orientation, not the full record), **Design for Shared Control** (explicit boundaries on autonomous vs. approval-required actions, interruptibility, rollback, and human-judgment decision points).
  - When to use: the master checklist for any agentic product review — "is our plan visible before cost is spent? Is information layered by attention level? Can the user always interrupt, redirect, or reclaim control?"
- **Processing sub-stages** (Decomposition → Sequencing → Delegation): decomposition = breaking a goal into actionable parts and showing them as a plan; sequencing = the dependency order between tasks (what runs first/parallel); delegation = the point where description becomes execution, often handing off to sub-agents or external tools.
  - When to use: designing the moment between a user's request and an agent beginning autonomous work — each sub-stage is a distinct opportunity for a user checkpoint.
- **Checkpoint system** (checkpoints, rollback, intermediate outputs, permissions, edits/errors, sources): the full toolkit for making continuous agentic computation inspectable and revisable, modeled on Git-style version history.

## Key Concepts
- **Reflection pattern in practice**: generate → critique (self- or separate critic-agent) → regenerate loop; valuable for high-stakes work (financial calcs, code, compliance) but adds latency that must be explained, and raises the design question of how much of the critique process to expose (most systems show only the final refined output + optional access to what was checked/changed).
- **Tool Use permission tiers** (Claude Code example): read-only tools (no approval needed) vs. Bash/shell commands (approval required, "don't ask again" persists per-project) vs. file modification (approval required, persists until session end) — a concrete tiered-risk permission model.
- **Planning granularity**: too high-level = no real insight into what the agent will do; too fine-grained = overwhelming detail. Right level scales with task duration — a <1-minute task needs ~3 steps; multi-minute analysis benefits from phased disclosure (broad phases now, substeps revealed as each phase begins).
- **Conditional/hierarchical/adaptive planning**: hierarchical = steps that expand into substeps on demand ("Step 1: Diagnose" → "1a: Check logs, 1b: Run diagnostics"); adaptive = explicit branch points stated up front ("After Step 2, I'll either X or Y depending on data quality").
- **Multiagent conflict resolution modes**: manager-agent arbitration, voting/consensus among agents, escalation to a human — the design opportunity in conflicts is surfacing both perspectives to the user as a decision point ("Cost agent says 10 hours; historical data agent says 15 — which estimate should I use?") rather than silently picking one.
- **Browser/Computer Use visualization**: showing a live remote-desktop/browser view teaches capability through direct observation (users see an agent navigate a familiar spreadsheet UI) but has real limits — no temporal navigation (can't see what's done vs. remaining), high bandwidth demands, impractical for long tasks. Best combined with, not substituted for, layered textual progress.

## Mental Models
- The Clippy failure mode generalizes: proactive help that (1) can't tell a genuinely-stuck user from an expert who dismissed the same suggestion 50 times, (2) interrupts on the system's timing not the user's, and (3) lacks the capability to actually finish the task, will always erode trust regardless of good intentions.
- Treat a visible plan as "a contract" — when a user approves (explicitly or by not objecting) a stated scope of work, it prevents the agent from silently doing much more or much less than expected.
- Distinguish "capability" (what agentic patterns technically do) from "experience" (what the user actually perceives) — users never think "ah, this is using ReAct"; they experience a surface layer of plans, progress, and checkpoints that must translate the engineering pattern into legible collaboration.

## Anti-patterns
- **No visible plan before agent execution begins**: the "trust gap" — users can't tell whether the agent understood correctly until work is already done, when correction is expensive.
- **Showing every reflection/critique cycle by default**: overwhelming and can expose uncertainty in ways that undermine confidence — hide by default, make available via progressive disclosure (e.g., collapsed Chain-of-Thought).
- **Requesting granular permission at every single step of a large task**: inefficient friction — offer persistent/"don't ask again" permission scoped appropriately (per-project, per-session) instead of re-asking constantly.
- **A ReAct agent claiming a fixed step count** ("Step 3 of 5") when the loop has no predetermined number of iterations — show iteration count or elapsed time/effort instead of a false determinate progress bar.
- **Screen/browser-visualization as the sole progress mechanism for long tasks**: no way to estimate remaining work, can't step away and return meaningfully — pair with layered status, don't rely on it alone.
- **Rollback that deletes the current state**: restoring a prior checkpoint should create a new branch/snapshot, never destroy in-progress work.

## Reference Tables

| Agentic pattern | Core problem it solves | Primary interface implication |
|---|---|---|
| Reflection | Confident mistakes (hallucinated code/facts) | Explain added latency; decide what critique detail to expose |
| Tool Use | Limited knowledge / can't act in the world | Permission model (tiered risk), visible tool-access indicators |
| Planning | Multistep task complexity, losing track | Reveal plan pre-execution; scale detail to task duration |
| Multiagent Collaboration | Different subtasks need different expertise | Status per-agent only when useful; surface conflicts as decision points |
| ReAct | Path forward unclear from the start | Show iteration/reasoning trace; signal exploratory vs. direct-answer mode |

| Checkpoint element | Purpose |
|---|---|
| Rollback | Git-style version restore without deleting current state |
| Intermediate outputs | Provisional partial results (draft charts/text) users can accept or revise before continuing |
| Permissions | Explicit, consistent, revocable access requests tied to specific capabilities |
| Edits/errors | Pause-and-edit without losing completed work; errors shown in the normal checkpoint rhythm, not as system interruption |
| Sources | Provenance list (what was consulted, when, link to verify) attached to the relevant checkpoint |

## Worked Example
Replit's build flow demonstrates all three New Principles at once: when a user asks it to build something, Replit first shows a **plan** (files to create, functionality, work order) — a few seconds of review preventing minutes of misdirected effort ("Wait, I meant a web app, not a CLI tool" is caught before any code exists). During execution, it shows an "In progress tasks" component — simple cards/rows, expandable for dependencies — which is the **prioritization** principle in action (minimal overview by default, detail on demand). And its checkpoint system lets users pause, roll back to a prior saved state, and resume without losing completed work — the **shared control** principle. The three principles aren't separable in a good implementation: a visible plan is useless if buried in noise, and shared control requires both visibility (from prioritization) and a mechanism to act on it (from checkpoints).

## Key Takeaways
1. Name which of the five agentic patterns (Reflection, Tool Use, Planning, Multiagent, ReAct) is active before designing its interface — each has a distinct default failure mode and mitigation.
2. Reveal plans before execution begins, especially for complex/costly/hard-to-undo tasks — correction is cheap before work starts, expensive after.
3. Layer information (notifications → overview → detail → full record) rather than exposing everything or hiding everything — match exposure to task stakes and user attention level.
4. Tiered, persistent permissions (not per-step re-asking) balance user control against friction — model risk explicitly (read-only vs. destructive vs. requires-approval).
5. Checkpoints + rollback + intermediate outputs turn continuous autonomous computation into something inspectable and reversible — never let restoring a prior state destroy current progress.
6. Multiagent conflicts are a design opportunity, not just an engineering problem — surface disagreement as a decision point for human judgment rather than silently resolving it.
7. Screen/browser visualization teaches capability effectively but is a poor sole progress mechanism for long tasks — combine with layered status.

## Connects To
- **Ch 2**: extends the Model → Tool → Agent capability stack introduced there into full agentic design detail.
- **Ch 4**: the three-tier progress communication model (overview/detail/full record) is reused and extended here for agentic/autonomous contexts.
- **Ch 5**: canvas/versioning concepts (Claude's artifact versions) directly parallel the rollback/checkpoint system here.
- **External**: Clippy/Microsoft Bob as the chapter's organizing historical case; Claude Code's tiered permission framework and clarification-via-tabbing UX as recurring concrete examples.
