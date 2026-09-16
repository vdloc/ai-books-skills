# Chapter 3: Designing for AI Inputs

## Core Idea
User intent reaches an AI system through three simultaneous channels — implicit context, explicit prompting, and direct manipulation — and the design challenge isn't "teach users to write better prompts," it's engineering how those three channels share the burden of translation.

## Frameworks Introduced
- **Three Channels of Intent**: implicit context (inferred from environment — open document, selection, location, history), explicit prompting (typed/spoken request), direct manipulation (buttons, sliders, gestures).
  - When to use: whenever an AI feature seems to be "misunderstanding" the user — diagnose which channel failed rather than blaming prompt quality.
  - How: a single interaction usually uses all three at once (e.g., image open = context, lasso select = manipulation, "replace with a golden retriever" = prompting); each layer can succeed or fail independently, so design redundancy across channels.
- **CARE framework** (Nielsen Norman Group): Context, Action, Results, Examples — structure for what a strong prompt (or prompt-assist UI) should supply.
  - When to use: building any prompt-scaffolding UI (templates, guided forms, prompt-improvement suggestions).
  - How: **Context** = who/why/audience/constraints; **Action** = the specific processing/transformation wanted (not "look at this," but "identify X, compare Y, synthesize Z"); **Results** = desired output format/length/tone/audience; **Examples** = concrete samples showing the style wanted, which communicate tone/structure better than instructions alone.
- **Guidance spectrum** (invisible → visible scaffolding → structured templates → explicit example galleries): a continuum for how much scaffolding to put around an input field.
  - When to use: deciding how much hand-holding a prompt UI needs for a given user maturity level.
  - How: invisible consistency (stable placement builds intuition) → visible optional scaffolding (category selectors, progressive-disclosure "Advanced" toggle) → structured templates (fill-in-the-blank, teaches by example) → explicit prompt libraries (for users ready to go beyond templates). Users progress from template-dependence to template-transcendence as they internalize patterns.

## Key Concepts
- **Command-line interface (CLI) burden**: earliest computing input model — zero tolerance for imprecision, full translation burden on the human (Multics, 1960s).
- **PageRank**: Google's innovation of using hyperlink citation networks (not just keyword matching) to rank relevance — the historical shift from rigid syntax toward loosely-structured natural-language input.
- **Sycophancy**: LLM tendency to affirm a user's stated premise rather than correct it (Perez et al. 2022; Sharma et al. 2023) — reinforced by RLHF, which rewards responses raters found agreeable.
- **"Lost in the Middle" effect** (Liu et al. 2023): models attend less to information in the middle of long prompts than at the start/end — more detail doesn't monotonically improve output; there's a precision paradox where over-elaborated prompts degrade quality.
- **Jakob's Law**: users spend most time on *other* products, so they expect new interfaces (including AI ones) to behave like familiar ones — the reason Canva's Magic Edit and Adobe Generative Fill reuse standard lasso/selection-handle conventions.
- **Prompt-intent taxonomy**: instructive (defined output type), exploratory (open possibility space), role-based (assign persona/tone), corrective (iterative refinement — "make this more concise"), boundary-testing (edge cases/adversarial probing).

## Mental Models
- Treat the three intent channels as *redundant, not sequential* — design so that if explicit prompting is weak, implicit context or direct manipulation can still carry enough signal.
- Think of starter prompts as **product positioning**, not icebreakers — a generic "write a poem about autumn" wastes the single highest-leverage moment to show a user what only *this* product can do.
- Model formatting (bullets, headers, Markdown) as a *legibility signal to the tokenizer itself*, not just a human-readability nicety — cited research shows GPT-4.1 legal-reasoning accuracy improved ~20 points with Markdown-structured input vs. unformatted text.

## Anti-patterns
- **Assuming more prompt detail always helps**: ignores the lost-in-the-middle effect; over-elaborated prompts (accumulated "be creative," "think step-by-step" cruft) degrade rather than improve output, and current interfaces give no feedback about this threshold.
- **Generic/whimsical starter prompts** ("Ask me anything," "write a poem about autumn") on a task-specific tool — blurs product differentiation and fails to show the user the product's actual value at their first touchpoint.
- **Treating direct-manipulation buttons as the system's full capability set**: users may never discover anything beyond what buttons suggest — every preconfigured action is a choice about what to hide as well as surface.
- **No visibility into what implicit context the system is using**: if a user doesn't know the model is reading "current document" vs. "selected text," they can't correct wrong inferences — surface it ("Editing: selected text").
- **Ignoring sycophancy at the interface level**: systems mirror a user's stated framing (skeptical prompt → cautious response; optimistic prompt → enthusiastic response) creating an echo-chamber effect that feels like validation but is just reflection.

## Reference Tables

| Modality | Best for | Key constraint |
|---|---|---|
| Text | Universal default, lowest barrier | Open-ended, so boundaries of intent are ambiguous without scaffolding |
| Image | "Show don't tell" — visual problems | Must honor existing photo-editing conventions (lasso, handles, marching ants) or users must relearn mechanics |
| Voice | Mobile, hands-busy contexts (161 wpm vs. 53 wpm typing — Ruan et al., Stanford) | Short commands need low-latency recognition; long recordings need diarization; conversational voice needs a third, hybrid feedback model |

| Prompt style | Signals | Example |
|---|---|---|
| Instructive | Defined output type wanted | "Summarize this article in three bullet points" |
| Exploratory | Invites collaboration, varied results | "What are some creative ways to introduce a product launch?" |
| Role-based | Shapes tone/expertise | "You are a friendly but experienced travel agent…" |
| Corrective | Iterative refinement | "Make this more concise" |
| Boundary-testing | Surfaces system limits | "What's something you're not supposed to say?" |

## Worked Example
CARE applied to a board-presentation request: *"I'm preparing for a board presentation next week where I need to propose expanding our customer service team."* [Context] → *"Analyze our current support ticket data from the past six months to identify peak volume periods, average resolution times by issue type, and customer satisfaction trends. Then create a data-driven argument for why we need additional headcount."* [Action] → *"Format this as a 10-minute presentation outline with key statistics highlighted for slides, compelling talking points that address potential budget objections, and a clear ROI calculation."* [Result] → *"Here's an example of how our CEO prefers data presentations: [prior successful deck]."* [Example]. Compare to the weak version — "Help me with this sales data" — which supplies none of the four elements and predictably returns a generic, unusable response. The CARE structure is exactly what a prompt-assist UI should elicit through guided fields rather than expecting users to intuit it.

## Key Takeaways
1. Diagnose input failures by channel (context / prompting / manipulation), not by blaming the user's prompt-writing skill.
2. Use CARE (Context, Action, Results, Examples) as the schema for any prompt-scaffolding UI — templates, guided forms, or prompt-improvement suggestions.
3. More prompt detail isn't always better — the lost-in-the-middle effect means padding degrades output; help users see when they're overcomplicating rather than encouraging ever-longer prompts.
4. Formatting (Markdown, bullets, headers) measurably improves model comprehension, not just human readability — surface formatting tools in the input field.
5. Image and voice modalities succeed by reusing existing conventions (Jakob's Law) — don't invent new selection or dictation paradigms when a well-known one exists.
6. Starter prompts are a positioning tool — make them specific to the product's actual differentiators, not generic icebreakers.
7. Sycophancy is a real, documented failure mode (models mirror user framing rather than correcting it) — designers have limited direct levers but should push for interface warnings on leading/loaded questions.

## Connects To
- **Ch 1**: extends the "Input" stage of the ICO framework in full depth.
- **Ch 4**: what happens after input is captured — the Computation phase.
- **External**: Perez et al. "Discovering Language Model Behaviors with Model-Written Evaluations" (2022); Liu et al. "Lost in the Middle" (2023); Lehmann & Buschek text-editing-with-LLMs study; Braun/Lilienbeck/Mentjukov Markdown-formatting legal-reasoning study.
