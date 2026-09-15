# Chapter 7: Advanced Use Cases and Industry Applications

## Core Idea
The same core Bedrock building blocks (RAG, guardrails, agents, prompt templates) recur across every industry vertical — e-commerce, finance, media/entertainment — the differentiator is not new technology per vertical but disciplined engineering practice: prompt versioning, full invocation auditability, and architecture deep-dives that treat each use case as a production system, not a demo.

## Frameworks Introduced
- **Prompt Versioning with Semantic Versioning (MAJOR.MINOR.PATCH)**: treating prompts as versioned artifacts, not mutable strings.
  - When to use: any production prompt that will be iterated on after initial deployment (i.e. almost all of them).
  - How: apply semver-style discipline — MAJOR version bump for changes that alter the prompt's fundamental behavior/output contract, MINOR for backward-compatible enhancements (added instructions/examples), PATCH for wording/formatting fixes with no behavior change. Store versioned prompts in Git (or a `PromptTemplateID` + `PromptTemplateVersion` registry) so any production output can be traced back to the exact prompt version that generated it.
- **Audit Log Schema for LLM Invocations**: the book's concrete schema for full reproducibility/auditability of every Bedrock call.
  - When to use: any regulated or business-critical use case (the chapter's Finance deep-dive especially) where "what exactly did the model see and say" must be reconstructable after the fact.
  - How: log, per invocation — `InvocationID` (UUID), `TimestampUTC` (ISO 8601), `UserID`/`SystemID`, `ModelID`, `PromptTemplateID` + `PromptTemplateVersion`, `InputVariables`, `FinalPrompt` (the fully rendered prompt sent to the model), `InferenceParameters`, `RawModelResponse`, `GuardrailEvaluation` result, `InvocationLatency`, and a `TraceID` linking the call to its parent pipeline/request. This schema is what makes an AI decision defensible in a compliance review (e.g. explaining a fraud-flag decision).

## Key Concepts
- **Reference Architecture for a Conversational Commerce Engine** (E-commerce deep dive): a RAG-based chatbot with access to order data — grounds responses in FAQs, shipping/delivery info, product warranty details, and return/exchange policies via RAG, plus live order-status lookups via a tool/function-calling layer, rather than relying on the base model's general knowledge.
- **Event-Driven Fraud Explanation Workflow** (Finance deep-dive): a flagged transaction event triggers a workflow where the system retrieves full context of the flagged transaction, then invokes Bedrock (with the Audit Log Schema logging every step) to generate a human-readable explanation of *why* the transaction was flagged — turning an opaque fraud-detection score into an auditable, explainable narrative.
- **Narrative Agent Architecture with Multi-Agent Collaboration** (Media/Entertainment deep-dive): a multi-agent system where specialized agents (e.g. one for plot structure, one for dialogue, one for continuity-checking) collaborate on generating long-form narrative content, coordinated rather than relying on a single model call to do everything at once.
- **Media Enrichment Pipeline**: automates tasks that were historically manual/labor-intensive in media production — synopsis generation from raw footage, multi-language script translation/transcreation, and interactive storytelling — addressing the industry's traditional bottlenecks in localization and post-production.
- **Transcreation** (vs. literal translation): adapting content across languages/cultures for tone, idiom, and audience resonance rather than word-for-word translation — the chapter gives explicit best-practice guidance for transcreation prompts, distinct from ordinary translation prompts.

## Mental Models
- Treat **every advanced use case as "RAG/guardrails/agents + an audit trail"**, not as needing fundamentally new Bedrock capabilities — the chapter's three industry deep-dives are variations on the same architectural primitives from Ch1-Ch6, applied with domain-specific data and compliance requirements.
- Use **semantic versioning for prompts the same way you'd version an API** — a prompt change that alters output structure/behavior is a breaking change for anything downstream consuming that output, and should be versioned as such.
- In regulated domains (finance especially), design for **explainability as a first-class output**, not an add-on — the Fraud Explanation Workflow generates the explanation as part of the primary response, not as a separate debugging step.

## Anti-patterns
- **Treating prompts as mutable, unversioned strings in production**: makes it impossible to reproduce or debug why a past output was generated the way it was — the book's audit schema and semver framework exist specifically to close this gap.
- **Building a single-agent "do everything" system for complex creative generation**: the Narrative Agent example shows multi-agent decomposition (plot, dialogue, continuity) producing more coherent long-form output than one model call handling the entire task.
- **Logging only the final model output, not the full invocation context**: without `FinalPrompt`, `InputVariables`, and `GuardrailEvaluation` in the log, an auditor (or engineer debugging a bad output) can't reconstruct what actually happened — partial logging defeats the purpose of an audit trail.

## Reference Tables
**Industry Deep-Dive Architectures**
| Industry | Core Pattern | Key Bedrock Components |
|---|---|---|
| E-commerce | RAG-based chatbot with order-data access | Knowledge Bases (FAQs/policies) + tool-calling for live order lookups |
| Finance | Event-driven fraud explanation workflow | Retrieve flagged-transaction context → Bedrock invocation → audit-logged explanation |
| Media & Entertainment | Narrative agent with multi-agent collaboration; media enrichment pipeline | Multi-agent orchestration, transcreation prompts, synopsis generation |

**Audit Log Schema (fields)**: InvocationID, TimestampUTC, UserID/SystemID, ModelID, PromptTemplateID, PromptTemplateVersion, InputVariables, FinalPrompt, InferenceParameters, RawModelResponse, GuardrailEvaluation, InvocationLatency, TraceID.

## Worked Example
The Finance deep-dive's fraud explanation workflow: a transaction-monitoring system flags a transaction → an event triggers a Lambda function that retrieves the full context of the flagged transaction (amount, merchant, account history) → the function constructs a `FinalPrompt` from a versioned `PromptTemplateID`, invokes Bedrock, and captures the full Audit Log Schema record (including `GuardrailEvaluation`) → the generated human-readable explanation is returned to the compliance/fraud team alongside the complete audit trail, satisfying the "why was this flagged" reproducibility requirement that a raw ML fraud score alone can't provide.

## Key Takeaways
1. Apply semantic versioning (MAJOR.MINOR.PATCH) to prompts in production — treat prompt changes with the same discipline as API versioning.
2. Log the full Audit Log Schema (not just the output) for any regulated or business-critical Bedrock invocation, to support reproducibility and compliance review.
3. Decompose complex generative tasks (long-form narrative, multi-step creative work) into multi-agent architectures rather than one large single-call prompt.
4. Use transcreation-specific prompting (not plain translation prompts) when localizing content across languages/cultures.
5. Every industry vertical's "advanced" use case is a recombination of Ch1-Ch6's primitives (RAG, guardrails, agents, audit logging) applied to domain-specific data and compliance needs — look for the underlying pattern before assuming a new capability is needed.

## Connects To
- **Ch4**: the RAG pipeline built there underlies the E-commerce conversational commerce engine here.
- **Ch5/Ch6**: event-driven integration and observability patterns underpin the Fraud Explanation Workflow's auditability.
- **Ch8**: multi-agent collaboration (Narrative Agent) here is a direct precursor to Ch8's full treatment of autonomous AI agents.
