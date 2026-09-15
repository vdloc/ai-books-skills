# Patterns — Building AI-Powered Products

## The AI Product Development Lifecycle (AIPDL)
**When to use**: as the default project structure for any AI feature, 0-to-1 or 1-to-n.
**How**: cycle through five stages — Ideation (identify AI features via superpower mapping + RICE prioritization), Opportunity (validate against the three-pillar product-market-fit test), Concept/Prototype (build an AI MVP, not a prototype — includes the five-stage AI lifecycle for the model itself), Testing & Analysis (structured feedback, culminating in a Go/No-Go decision), Rollout (launch + ongoing monitoring/retraining).
**Trade-offs**: iterative and cyclical by design — expect to loop back after a "No-Go," which costs time but avoids shipping unvalidated bets.

## RICE Feature Prioritization (+ AI Investment)
**When to use**: ranking candidate AI features once you have more ideas than resources.
**How**: score Reach (users/time window), Impact (1-3 scale), Confidence (% certainty — often lower for AI due to data/feasibility uncertainty), Effort (person-months including data+training+integration). RICE = R×I×C/E. Optionally divide by an AI Investment factor for model/data complexity: R×I×C/(E×A).
**Trade-offs**: objective and defensible, but Confidence scores are inherently softer for novel AI features than for established feature types — don't over-trust precision on early-stage estimates.

## Product-Market Fit Three-Pillar Test
**When to use**: deciding whether to proceed past a hypothesis at the AIPDL's Opportunity stage.
**How**: check business viability (revenue/ROI/compliance), technical feasibility (data/compute/talent available), and user desirability (validated pain point) — all three as independent gates, not an average.
**Trade-offs**: a product can pass 2 of 3 pillars strongly and still fail (e.g. technically feasible + desirable AI mood-detection product killed by weak business viability due to privacy risk). Forces explicit external validation, not just internal technical confidence.

## AI MVP (vs. Prototype)
**When to use**: deciding what to actually ship at the Concept/Prototype stage.
**How**: build something that (1) may hardcode non-critical parts for speed, (2) demonstrates low-effort integration compatibility via API, (3) showcases domain expertise on a small high-quality dataset, (4) adds real value from day one with a feedback loop for future learning — using live data and real systems, not mocks.
**Trade-offs**: faster and more honest signal than a prototype, but requires more upfront integration work than a pure demo.

## The Trade Space (Six-Step Method)
**When to use**: navigating any decision with more than two competing, interdependent factors (build/buy, on-device/cloud, accuracy/speed).
**How**: (1) identify key factors via cross-functional input, (2) rank priorities — non-negotiable vs. compromisable, (3) map interdependencies between factors, (4) visualize as a matrix/graph, (5) test scenarios by simulating decisions, (6) iterate as the project evolves.
**Trade-offs**: more setup effort than a gut call, but prevents force-fitting multidimensional decisions into false binaries.

## Build-vs-Buy Decision Matrix
**When to use**: deciding whether to build an AI capability in-house or license/buy it.
**How**: score across core competency, resources/expertise, time to market, long-term strategy, cost, risk/uncertainty, data privacy/ethics, competitive landscape. Default to build when AI is central to the value proposition and a long-term strategic asset; default to buy when AI is supporting/secondary and speed matters. Default to hybrid (build the differentiator, buy the commodity) when the signal is mixed.
**Trade-offs**: building maximizes control/customization/data-privacy but costs more upfront and takes longer; buying is faster and lower-risk but creates vendor dependency and less differentiation.

## Innovator's Dilemma Classification
**When to use**: setting expectations for a new AI bet against your existing product line.
**How**: classify as sustaining (incremental, meets current users' current needs) or disruptive (initially inferior/niche, targets a future/unmet need, redefines the market over time). Judge each against the metrics appropriate to its category — don't apply sustaining-innovation KPIs to a disruptive bet.
**Trade-offs**: disruptive bets look weak by conventional metrics early on — this is expected, not a red flag, but makes internal buy-in harder to secure.

## Fine-Tuning vs. RAG vs. Grounding
**When to use**: adapting a pretrained/foundation model to your product's specific needs.
**How**: fine-tuning for well-defined, precision-critical, relatively static tasks (large labeled dataset, high setup cost, high accuracy). RAG for information that changes faster than retraining cycles (news, trends) — retrieval from a live corpus, no retraining needed. Grounding for cheap, fast, lightweight behavior adjustments via prompt engineering — no new data needed.
**Trade-offs**: fine-tuning is the most accurate but most expensive/slowest to update; grounding is the cheapest/fastest but least precise; RAG sits in between, trading some accuracy for currency.

## Synthetic vs. Real-World Data Strategy
**When to use**: deciding how to source training data, especially for sensitive or rare-event domains.
**How**: use synthetic data for sensitive info (healthcare, finance), rare/critical scenarios (self-driving accidents), early fast-moving development, or when real collection is prohibitively expensive. Use real data when user behavior/preferences are central, cultural/contextual nuance matters, or decisions are high-stakes/user-facing. Hybrid (synthetic to bootstrap, real to refine) is common — validate synthetic data's statistical properties against real samples regularly.
**Trade-offs**: synthetic data protects privacy and covers rare events cheaply, but risks missing the "messy, wonderful complexity" of real user behavior — never treat it as a full substitute for user-behavior-critical products.

## AI Product Metric Blend + OKR Framework
**When to use**: defining what success means for an AI feature, and setting quarterly goals.
**How**: blend product health (engagement, satisfaction, adoption, retention, revenue), system health (uptime, latency, scalability, error rate), and AI proxy metrics (accuracy, precision, recall, loss). Build each OKR with: objective, specific features, exactly one North Star metric, at least one KPI from each of the three buckets, and a guardrail metric capping acceptable side effects.
**Trade-offs**: more setup effort than a single metric, but prevents optimizing one dimension (e.g. model accuracy) at the expense of another (e.g. user retention).

## Cross-Functional Stakeholder Mapping
**When to use**: scoping who needs to be involved before a launch or major feature decision.
**How**: name every function explicitly — AI/ML teams (scientists, red/blue security, MLOps), operations (program managers, DataOps), engineering (developers, testers, data engineers, TPMs), UX (researchers, designers, content), business (PMMs, sales, partnerships), third-party (vendors, consultants), GRC (legal, privacy, compliance), leadership (C-suite, investors).
**Trade-offs**: smaller orgs collapse roles across fewer people, but skipping a *function* entirely (e.g. no security red-team pass) is a gap, not a valid simplification.

## Agent Design Decision Sequence
**When to use**: designing any agentic (autonomous, learning, proactive) product feature.
**How**: work through six decisions in order — (1) task-specific vs. general-purpose (and if task-specific: reflex/goal-based/utility-based), (2) activation (proactive vs. reactive), (3) autonomy level (explicitly scoped, often progressive), (4) feedback/learning mechanism (explicit or implicit), (5) UI pattern matched to activation type (side panel, floating bubble, chat interface, integrated UI, pop-up, collaborative browser), (6) scalability/integration/privacy (load, languages, GDPR/CCPA, API/CRM compatibility).
**Trade-offs**: more structured than "just add an LLM," but the alternative (skipping these decisions) tends to produce either an over-autonomous agent users don't trust or an under-autonomous one that adds no value over a chatbot.
