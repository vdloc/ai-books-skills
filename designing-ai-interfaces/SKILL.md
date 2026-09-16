---
name: designing-ai-interfaces
description: "Knowledge base from \"Designing AI Interfaces: Design Principles for Creative and Autonomous AI\" by Louise Macfadyen. Use when designing UX for LLM-powered products or AI agents, structuring inputs/outputs for generative AI, designing latency/progress/error states for AI computation, building agentic (plan/act/adapt) interfaces, or referencing the book's frameworks and patterns."
---

# Designing AI Interfaces: Design Principles for Creative and Autonomous AI

**Author**: Louise Macfadyen | **Pages**: ~180 | **Chapters**: 6 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference across any AI UX task.
- **With a topic** — ask about `latency`, `agentic design`, `hallucinations`, `grounding`, `permissions`, or another indexed topic; the relevant chapter is read on demand.
- **With a chapter** — ask for `ch04` (or by name) to load that specific chapter in full.
- **Browse** — ask "what chapters do you have?" to see the full index below.

When a question touches a topic not covered in Core Frameworks, read the relevant chapter file before answering — chapters carry the worked examples and reference tables that make answers concrete rather than generic.

---

## Core Frameworks & Mental Models

**Input-Computation-Output (ICO)**: the book's organizing structure for every AI interaction. Input = where human intention enters (implicit context, explicit prompting, direct manipulation). Computation = the opaque middle stage (tokenization, routing, generation) where meaning is made but invisible to the user. Output = the delivered, formatted result — never "the answer" itself, but a designed artifact requiring interpretation. Diagnose any AI UX failure by first asking which of the three stages broke.

**Three Channels of Intent** (Ch 3): implicit context (inferred from environment), explicit prompting (typed/spoken request), direct manipulation (buttons/sliders/gestures). A single interaction typically uses all three redundantly — design so a weak signal in one channel is compensated by another, rather than relying on prompt quality alone.

**CARE framework** (Ch 3, Nielsen Norman Group): Context, Action, Results, Examples — the schema for what makes a prompt (or a prompt-assist UI) effective. Weak prompts omit these; strong ones supply them explicitly.

**Five Output Design Principles** (Ch 5): outputs should be Clear (instantly understandable), Verifiable (independently checkable, not just confidence-scored), Grounded (context/assumptions visible), Actionable (onward-task-oriented — the output is a staging area for what happens next), Adjustable (user-editable, not restart-from-scratch).

**Verifiability test** (Ch 5): never surface a raw model confidence percentage — it's next-token probability, not epistemic certainty, and looks deceptively like a legible statistic (e.g., a spell-checker's 93%) when it isn't. Instead ask "can the user independently validate this?" Low-stakes/subjective → confidence is irrelevant. High-stakes/objective and hard to verify → require a secondary human verification layer, traceable sources, or hedging language.

**Five Agentic Design Patterns** (Ch 6): Reflection (self-critique loop — catches confident mistakes before the user sees them), Tool Use (extends the agent from advisor to operator via APIs/databases), Planning (breaks complex goals into structured, reviewable steps), Multiagent Collaboration (distributes work among specialists via an orchestrator), ReAct (Reason+Act — adaptive iteration when the path isn't clear up front). Not mutually exclusive; strong systems combine several.

**Three New Principles for Agentic Interface Design** (Ch 6) — the master checklist for any agentic product:
1. **Reveal the Plan** — show interpretation before execution begins, while correction is still cheap.
2. **Prioritize What Matters Most** — layer information (notifications → overview → detail → full record); don't show everything or hide everything.
3. **Design for Shared Control** — explicit, adjustable boundaries on autonomous-vs-approval-required actions; interruptibility; rollback without data loss; surface genuinely human-judgment decisions as collaboration, not system failure.

**Three-Tier Progress Communication** (Ch 4, extended in Ch 6): Tier 1 Overview (minimal always-visible status) → Tier 2 Detail (clickable breakdown, builds trust/verification) → Tier 3 Full Record (complete retraceable log, for debugging/compliance). Notifications sit above all three as an abstract "still working" signal.

**Discovery 2×2** (Ch 2): classify feature-surfacing choice by user intent (do they know they want this?) × system initiative (does the tool act proactively or wait?) — four distinct design postures result.

**Model → Tool → Agent capability stack** (Ch 2): models are reactive (respond to a prompt, no goal-pursuit); tools extend a model into the world (user still directs each step); agents add autonomous planning on top of both. Each layer needs different interface affordances — feedback vs. permissioning vs. pause/redirect trust controls.

**Grounding vs. Verifiability** (Ch 5): verifiability asks "is this claim checkable"; grounding asks "under what context/assumptions was this produced" (jurisdiction, date, mode, model version). An output can be fluently verifiable-sounding yet ungrounded — wrong for the user's actual situation (the ChatGPT-wrong-state-marriage-law case).

**AI overreliance** (Ch 5, Microsoft Aether research): skill atrophy, automation bias, confirmation bias, and ordering effects — counterintuitively, *more detailed explanations can increase* overreliance rather than improve judgment. Interface countermeasures: cognitive forcing functions, transparent capability communication, progressive disclosure.

**Hallucinations are undetectable-by-design errors** (Ch 4/5): unlike traditional software errors, the system doesn't know it's wrong — burden of detection shifts to the user. The mitigation is designed friction that invites verification, not error-recovery UI (which assumes the system knows something failed).

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-understanding-llms-and-systems.md) | Understanding Large Language Models and Systems | Input-Computation-Output framework, AI Organizational Readiness levels |
| [ch02](chapters/ch02-capability-discovery-orchestration.md) | Capability, Discovery, and Orchestration | Intent/Capability/Discovery/Orchestration, Discovery 2×2, Model→Tool→Agent stack |
| [ch03](chapters/ch03-designing-for-ai-inputs.md) | Designing for AI Inputs | Three Channels of Intent, CARE framework, Guidance spectrum |
| [ch04](chapters/ch04-computation-processing-and-generation.md) | Computation: Designing for the Processing and Generation Phase | Computation pipeline, Latency priorities, Three-tier progress, Error-design principles |
| [ch05](chapters/ch05-output-delivery-and-presentation.md) | Output: Designing the Delivery and Presentation of LLM Responses | Five output principles, Verifiability test, Canvas vs. chat, Watermarking |
| [ch06](chapters/ch06-agentic-ai-plan-act-adapt.md) | Agentic AI: Designing for Systems That Plan, Act, and Adapt | Five agentic patterns, Three New Principles, Checkpoint system |

## Topic Index

- **Agentic design patterns** → ch06
- **AI disclosure / watermarking / SynthID** → ch05
- **AI organizational readiness** → ch01
- **CARE framework** → ch03
- **Canvas (editable workspace)** → ch05
- **Capability / discoverability / orchestration** → ch02
- **CDSS (clinical decision support systems)** → ch05
- **Checkpoints, rollback, permissions (agentic)** → ch06
- **Confidence indicators / verifiability** → ch05
- **Direct manipulation** → ch03
- **Discovery 2×2 matrix** → ch02
- **Error design (recoverability, degradation)** → ch04
- **Grounding** → ch05
- **Hallucinations** → ch01, ch04, ch05
- **Human-in-the-loop (HITL)** → ch05
- **Implicit context / explicit prompting** → ch03
- **Input-Computation-Output (ICO) framework** → ch01
- **Latency design / loading indicators** → ch04
- **Model/Tool/Agent capability stack** → ch02
- **Momentum behavior** → ch02
- **Multiagent collaboration / conflict resolution** → ch06
- **Overreliance (AI)** → ch05
- **Planning pattern (agentic)** → ch06
- **Progress communication (three-tier)** → ch04, ch06
- **ReAct pattern** → ch06
- **Reflection pattern** → ch06
- **Sycophancy** → ch03
- **Temperature (generation parameter)** → ch04, ch05
- **Tokenization / latent space** → ch04
- **Tool Use pattern** → ch06
- **Vibe coding** → ch02, ch05

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the book content only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this book, check related skills or ask the agent directly.
