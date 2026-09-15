# Chapter 5: Strategic Thinking in AI

## Core Idea
Strategic AI thinking means running a sequence of explicit checks — is AI even the right tool, is the innovation sustaining or disruptive, should you build or buy, should you use synthetic or real data, should you fine-tune/RAG/ground — instead of defaulting to "add AI" and hoping the tactics sort themselves out later.

## Frameworks Introduced
- **"AI Might Not Always Be the Answer" checklist**: a deliberate off-ramp before committing to an AI solution.
  - When to use: at the very start of any AI feature proposal, before Ideation.
  - How: don't use AI when (1) a simpler/cheaper alternative solves it equally well, (2) you can't get good enough data, (3) your org isn't ready to productionize (infra for scale/security/latency/monitoring), (4) cost outweighs justified ROI, (5) you can't commit to indefinite maintenance/iteration.
- **The Innovator's Dilemma applied to AI** (Clayton Christensen's framework): decide whether your AI feature is sustaining or disruptive innovation before setting expectations for it.
  - When to use: scoping any new AI initiative against your existing product line.
  - How: **Sustaining innovation** — incremental improvement to an existing product meeting current customers' current needs (e.g. AI-driven predictive analytics added to an existing product). **Disruptive innovation** — initially inferior/niche, targets an unmet or future need, redefines the market over time (smartphone cameras vs. point-and-shoot, as the book's example). Expect disruptive AI bets to look weak on today's metrics — that's not a failure signal, it's the pattern.
- **Build-vs-Buy Decision Matrix** (Table 5-1): the strategic-level version of Chapter 3's build/buy factor list, now organized as a lookup table.
  - When to use: deciding whether to build an AI capability in-house or buy/license a pretrained solution.
  - How: score across 7 rows — core competency, resources/expertise, time to market, long-term strategy, cost, risk/uncertainty, data privacy/ethics, competitive landscape — each cell states which side (build/buy) wins for that factor. AI central to your value prop + long-term strategic asset → build. AI supporting/secondary + speed matters → buy. Consider a **hybrid approach**: build the differentiating core (e.g. proprietary recommendation engine) in-house, buy commodity capability (e.g. a pretrained LLM for NLP) — Nika's explicit recommended default when the two don't cleanly separate.
- **Synthetic vs. Real-World Data decision rule**: when to simulate training data vs. collect it live.
  - When to use: at the data-strategy stage, especially when real data is scarce, sensitive, or dangerous to collect.
  - How: use synthetic data when handling sensitive info (healthcare, finance), testing rare/critical scenarios (self-driving accidents), moving fast in early development, or when real-data collection is prohibitively expensive. Stick to real data when user behavior/preferences are central, cultural/contextual nuance matters, or decisions are high-stakes and directly affect users. Hybrid is common and recommended (Tesla trains on both) — start synthetic, layer in real data as you gather it; always validate synthetic data's statistical properties against real samples.
- **Fine-tuning vs. RAG vs. Grounding decision framework** (Table 5-2): the three ways to adapt a base/pretrained model to your product, compared on latency, data needs, accuracy, and scalability.
  - When to use: choosing how to specialize an LLM or pretrained model for your product's use case.
  - How: **Fine-tuning** — retrain a pretrained model further on your specific labeled dataset; high latency to set up, needs large labeled data, high accuracy, resource-intensive; best for well-defined, precision-critical tasks (e.g. content-moderation hate-speech detection). **RAG** — retrieval mechanism pulls from a live corpus without retraining the model; can be latency-optimized, needs a large retrieval corpus (not labeled training data), variable accuracy depending on retrieved data, scales with corpus size; best for dynamic/frequently-changing information (news, trending topics). **Grounding** — prompt-engineering context to steer a base model; low latency, minimal new data, moderate accuracy, scales easily; best for rapid iteration and lightweight behavior adjustments (chatbot tone/flow).
- **Product Review Types** (Table 5-3): match the review format to the decision you actually need from leadership.
  - When to use: scheduling any leadership/stakeholder checkpoint.
  - How: **Decision review** — go/no-go, present options with pros/cons/trade-offs, outcome = clear decision + action items. **Discussion review** — open-ended early-stage brainstorming, outcome = gathered feedback. **Alignment review** — cross-functional vision/goals/timeline alignment, outcome = surfaced misalignments resolved. **Status update** — KPIs/milestones/roadblocks, outcome = stakeholders informed.

## Key Concepts
- **Disruptive vs. sustaining innovation**: see framework above; the central strategic lens for any new AI bet.
- **Hybrid build/buy**: proprietary core + third-party commodity components — the book's recommended default for most real organizations.
- **Core competency test**: is AI central to your value proposition (→ lean build) or a supporting feature (→ lean buy)?
- **Grounding**: the lightest-weight of the three model-adaptation methods, via prompt engineering rather than training.

## Mental Models
- Run the "AI might not be the answer" checklist as a gate, not a formality — it's designed to produce a legitimate "no AI needed" outcome, and that's a valid strategic conclusion.
- Treat disruptive AI bets like the smartphone-camera analogy: judge them against the future need they target, not against today's incumbent product on today's metrics.
- Use the fine-tuning/RAG/grounding choice as a spectrum of investment vs. flexibility: grounding = cheap and fast to iterate, fine-tuning = expensive but precise, RAG = the middle ground for information that changes faster than a model can be retrained.

## Anti-patterns
- **Adding AI for AI's sake**: explicitly called out — "not every problem requires an AI solution"; always trace back to a problem only AI can solve.
- **Judging a disruptive AI innovation by sustaining-innovation metrics**: dismissing a niche/early AI bet because it underperforms your mature product on today's KPIs misreads the Innovator's Dilemma pattern.
- **Fine-tuning for information that changes daily**: mismatches the tool to the problem — RAG exists specifically for this case, since retraining is far more expensive than retrieval.
- **Skipping the pre-review prep checklist**: showing up to a decision review without a shared PRD/deck beforehand wastes the room's time and undermines the review's purpose.

## Worked Example
**Build-vs-buy applied to a music streaming company** (the chapter's running example): build the proprietary recommendation engine in-house (core differentiator, long-term strategic asset) while buying/using a pretrained LLM (e.g. GPT-3-class) for NLP tasks like understanding song lyrics or generating metadata (commodity capability, not differentiating). Extended into fine-tuning/RAG/grounding: the same company might use **RAG** to recommend music aligned with current social-media trends (e.g. surfacing a song blowing up on TikTok) because that information changes faster than any fine-tuned model could be retrained to reflect it, while using **fine-tuning** for personalized recommendation quality that benefits from deep specialization on user-specific listening behavior.

**Real-world validation**: Salesforce acquired Tableau (2019) rather than building comparable analytics in-house — buying won on time-to-market despite the higher acquisition cost, because building would have ceded ground to competitors SAP and Oracle.

## Key Takeaways
1. Run the "AI might not be the answer" checklist before committing resources — a legitimate "no" is a valid strategic outcome, not a failure to find a use case.
2. Classify every AI bet as sustaining or disruptive innovation before setting success metrics for it — judging a disruptive bet by sustaining-innovation KPIs will kill promising niche investments prematurely.
3. Use the build-vs-buy matrix's 7 factors explicitly, and default to a hybrid approach (build the differentiator, buy the commodity) when the answer isn't clean.
4. Choose synthetic vs. real data based on sensitivity, scenario rarity, and cost — not by default; validate synthetic data against real samples before trusting it.
5. Match the model-adaptation method to the problem: fine-tuning for precision-critical well-defined tasks, RAG for fast-changing information, grounding for cheap rapid iteration.
6. Pick the product review type (decision / discussion / alignment / status) that matches what you actually need from the room, and always send a pre-read PRD/deck plus a post-review summary with clear owners.

## Connects To
- **Ch 3**: Build-vs-buy and trade-space thinking introduced there are formalized here into the full decision matrix (Table 5-1).
- **Ch 2**: The RICE-prioritized features from Ch. 2's Ideation stage are what get evaluated here for sustaining/disruptive classification and build/buy decisions.
- **Ch 6**: The KPIs referenced in Status Update reviews here are defined in full in Chapter 6's goal-setting and metrics framework.
