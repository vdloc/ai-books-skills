# Chapter 2: Generative AI in Business Unlocking Value

## Core Idea
GenAI's business value clusters into five capability areas — conversational chat, insights/data analysis, content generation, multimodal functionality, and code generation — and its impact varies systematically by industry and business function.

## Frameworks Introduced
- **Five GenAI Application Categories**: general-purpose chat, insights discovery, content drafting, multimodal functionality, code generation.
  - When to use: as an intake taxonomy — when scoping a new enterprise GenAI use case, classify it into one of these five to reuse the relevant pattern library and known failure modes.
- **Value-Driver Mapping (sector × function × use case)**: McKinsey-style matrix mapping industry + business function to a specific GenAI use case and its description (e.g., Banking/Customer Ops → GenAI-trained IVR bot).
  - How: for any new engagement, look up the client's sector and function to find analogous proven use cases before designing from scratch.

## Key Concepts
- **Enterprise semantic search**: LLM + internal document repositories combined to answer natural-language employee/customer queries directly instead of manual portal search.
- **GenAI-driven personalization**: using behavior/browsing history to generate tailored recommendations (healthcare wellness plans, banking advice, retail products).
- **GenAI-powered regulatory compliance**: dynamic, context-aware compliance checking vs. traditional static rule checklists.
- **Multimodal compliance check**: extending text compliance checks to images (e.g., scanning product packaging for missing FDA allergen warnings).
- **COTS delivery accelerator**: GenAI automating configuration/customization of commercial off-the-shelf systems (SAP, ServiceNow, Oracle).

## Mental Models
- **"Create → Act" as the maturity axis**: GenAI creates content and analyzes data; agentic AI (Ch 4) acts autonomously on it. Use this to explain to stakeholders why a chatbot alone isn't "agentic AI."
- When evaluating a candidate use case, ask **"Is this the LLM answering, analyzing, drafting, generating cross-modal content, or coding?"** — the answer determines which known failure modes and evaluation approach (Ch 6) apply.

## Anti-patterns
- **Static, checklist-based compliance systems** — the book contrasts these unfavorably with GenAI's adaptive, context-aware compliance review, since checklists don't adapt to changing regulation.
- **Manual, SME-dependent legacy code understanding** — creates bottlenecks and knowledge-loss risk; GenAI-assisted parsing reduces this dependency but doesn't eliminate the need for review.

## Reference Tables
| Sector | Function | Use case | Description |
|---|---|---|---|
| Banking | R&D/Software Eng | Legacy code conversion | Optimize migration of legacy frameworks |
| Banking | Customer Ops | Customer emergency IVR | GenAI bot trained on proprietary policy/research data for always-on technical support |
| Consumer/Retail | Marketing & Sales | Marketing content creation | Copywriting, brainstorming, content analysis at scale |
| Pharma | R&D/Software Eng | Drug discovery | Accelerates candidate protein/molecule selection |

## Worked Example
**Morgan Stanley's AI assistant**: built on GPT-4 to help tens of thousands of wealth managers rapidly search and synthesize the firm's internal knowledge base — the point being enterprise-wide knowledge search as a productivity multiplier, not a novelty chatbot.

**Stitch Fix style visualization**: uses DALL·E text-to-image generation so stylists can visualize a described clothing preference (color/fabric/style) and then match it against real inventory — an example of multimodal GenAI directly accelerating a human decision (product matching), not replacing the human.

## Key Takeaways
1. Classify any proposed GenAI project into one of the five capability categories before scoping — it reveals which known patterns and risks apply.
2. Multimodal GenAI extends value beyond text: compliance checks, marketing visuals, and insurance-claim scene reconstruction from text descriptions are concrete, already-deployed examples.
3. Real deployments (Morgan Stanley, Stitch Fix) succeed by augmenting an existing human workflow, not replacing it wholesale.
4. GenAI's value is *industry-specific* — retail gets the most value from marketing/customer interaction, high-tech from software development acceleration.

## Connects To
- **Ch 1**: builds directly on the LLM capability/limitation foundation from the prior chapter.
- **Ch 3**: the "create vs. act" distinction here sets up the design-pattern taxonomy for building these applications reliably.
- **Ch 4**: agentic AI is explicitly framed as "GenAI that can also act," the natural next step after this chapter's create-only capabilities.
