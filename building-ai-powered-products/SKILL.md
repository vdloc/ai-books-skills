---
name: building-ai-powered-products
description: "Knowledge base from \"Building AI-Powered Products: The Essential Guide to AI and GenAI Product Management\" by Dr. Marily Nika. Use when applying the AI Product Development Lifecycle (AIPDL), scoring AI features with RICE, deciding build-vs-buy or fine-tuning/RAG/grounding, setting AI product OKRs, designing AI agents, or referencing its frameworks."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Building AI-Powered Products
**Author**: Dr. Marily Nika | **Pages**: ~183 (Appendix/Index excluded per request) | **Chapters**: 8 | **Generated**: 2026-09-16

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `RICE`, `build vs buy`, `AI agents`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

**The AI Product Development Lifecycle (AIPDL)** is the book's spine: five iterative stages — Ideation → Opportunity → Concept/Prototype → Testing & Analysis → Rollout — moving a business problem to a shipped, monitored AI solution. Every other framework in the book slots into one of these stages. Expect to loop back, especially after a Testing & Analysis "No-Go."

**Before scoping any AI feature**, run the "AI might not be the answer" checklist: don't use AI if a simpler approach works as well, if you can't get good data, if you can't commit to indefinite maintenance, or if the cost doesn't justify the ROI. A legitimate "no AI needed" is a valid strategic outcome.

**0-to-1 vs. 1-to-n** is the first fork in almost every AI product decision: applying an emerging technology to a brand-new product (0-to-1, high uncertainty, market-discovery-heavy) vs. enhancing an established product (1-to-n, clearer market fit, pain-point-mining-heavy).

**RICE + AI Investment**: prioritize candidate features with Reach × Impact × Confidence / Effort, dividing further by an AI Investment factor (model/data complexity) when features are AI-specific. A low Confidence score signals unfinished Opportunity-stage validation, not a weak idea.

**Product-market fit is a strict three-pillar AND**: business viability, technical feasibility, and user desirability must *all* pass — a product can be technically brilliant and desirable and still fail PMF on business viability alone (e.g. a technically-feasible, user-desired AI mood-detection product killed by privacy/regulatory risk).

**AI MVP ≠ prototype**: an AI MVP must use live data and real integrations and add value from day one (hardcoding non-critical parts is fine); a prototype only needs to demonstrate feasibility with mock data.

**The AI lifecycle (nested inside Concept/Prototype)**: Project scoping → Data collection → Model training → Validation & testing (loop until Minimum Viable Quality, MVQ) → Deployment. Human-in-the-loop threads through every stage, not just one.

**Trade-offs are sliders, not binaries.** Use the Trade Space six-step method (identify factors → rank priorities → map interdependencies → visualize → test scenarios → iterate) instead of forcing multidimensional AI decisions (accuracy/speed, complexity/simplicity, data quality/quantity, generalization/specificity, privacy/personalization, ethics/business goals, explainability/performance) into false either/ors.

**Build vs. buy**: default to build when AI is core to your value proposition and a long-term strategic asset; default to buy when it's a supporting feature and speed matters; default to **hybrid** (build the differentiator, buy the commodity) when the signal is mixed — this is the common real-world answer. Score across 7 factors: core competency, resources/expertise, time to market, long-term strategy, cost, risk, data privacy, competitive landscape.

**Sustaining vs. disruptive innovation** (Christensen's Innovator's Dilemma, applied to AI): classify every AI bet before setting its success bar. Disruptive bets look weak on today's metrics — that's the expected pattern, not a failure signal. Don't judge a disruptive bet by sustaining-innovation KPIs.

**Fine-tuning vs. RAG vs. grounding**: fine-tune for well-defined, precision-critical, relatively static tasks (expensive, high accuracy); RAG for information that changes faster than a retrain cycle (news, trends); ground (prompt engineering) for cheap, fast, lightweight behavior tuning.

**Synthetic vs. real data**: synthetic for sensitive/rare/expensive-to-collect data; real when user behavior or contextual nuance is central; hybrid (synthetic to bootstrap, real to refine) is the common production pattern — always validate synthetic data against real samples.

**The AI product metric blend**: no single metric captures success. Blend product health (engagement, satisfaction, adoption, retention, revenue — the AI PM's direct responsibility), system health (uptime, latency, scalability, error rate), and AI proxy metrics (accuracy, precision, recall, loss functions). Every OKR gets exactly one North Star metric, at least one KPI from each bucket, and a guardrail metric capping acceptable side effects.

**AI agents are not chatbots.** An agent must be autonomous, learn from experience, and act proactively — ChatGPT itself is explicitly *not* an agent by this test. Design an agent through six sequential decisions: task-specific vs. general-purpose → activation (proactive/reactive) → autonomy level (progressive, explicitly scoped) → feedback/learning mechanism → UI pattern (side panel, floating bubble, chat interface, integrated UI, pop-up, collaborative browser — matched to activation type) → scalability/integration/privacy.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-role-of-ai-product-managers.md) | The Role of AI Product Managers | Four Types of AI, Seven Superpowers, 0-to-1 vs 1-to-n, AI PM Skill Set, Three AI PM Categories |
| [ch02](chapters/ch02-ai-product-development-lifecycle.md) | The AI Product Development Lifecycle | AIPDL, RICE, Product-Market Fit (3 pillars), AI MVP vs Prototype |
| [ch03](chapters/ch03-essential-ai-pm-knowledge.md) | Essential AI PM Knowledge | Trade Space, 7 AI Trade-offs, Build-vs-Buy factors, AI Lifecycle (5 stages), 4 Learning Methods, FATE |
| [ch04](chapters/ch04-ai-pms-day-to-day.md) | The AI PM's Day-to-Day | AI PM Career Ladder, Cross-Functional Stakeholder Map |
| [ch05](chapters/ch05-strategic-thinking-in-ai.md) | Strategic Thinking in AI | "AI Might Not Be the Answer," Innovator's Dilemma, Build-vs-Buy Matrix, Synthetic vs Real Data, Fine-tuning/RAG/Grounding, Product Review Types |
| [ch06](chapters/ch06-setting-goals-measuring-success.md) | Setting Goals and Measuring Success | AI Product Metric Blend, OKR Framework, Confusion Matrix |
| [ch07](chapters/ch07-ai-tools-for-product-managers.md) | AI Tools for Product Managers | AI Product Mgmt vs AI for PMs, AIPDL-Stage Tool Map |
| [ch08](chapters/ch08-building-ai-agents.md) | Building AI Agents | Agent Definition (Poole & Mackworth), Chatbot vs Agent vs Multi-Agent, Agent Type Matrix, 6 Design Decisions, UI Patterns |

## Topic Index

- **Agents (AI)** → ch01, ch08
- **AI Investment (RICE)** → ch02
- **AI Lifecycle (5 stages)** → ch03
- **AI Metric Blend** → ch06
- **AI Might Not Be the Answer** → ch05
- **AI Proxy Metrics** → ch06
- **AIPDL** → ch02, ch03
- **Autonomy (agent)** → ch08
- **Build vs. Buy** → ch03, ch05
- **Career Ladder** → ch04
- **Confusion Matrix** → ch06
- **Cross-Functional Stakeholders** → ch04
- **Disruptive Innovation** → ch05
- **Fine-tuning / RAG / Grounding** → ch05
- **FATE Framework** → ch03
- **Four Types of AI** → ch01
- **Go/No-Go Decision** → ch02
- **Innovator's Dilemma** → ch05
- **Learning Methods (ML)** → ch03
- **MVQ (Minimum Viable Quality)** → ch03
- **North Star Metric** → ch06
- **OKRs** → ch04, ch06
- **Product Health Metrics** → ch06
- **Product-Market Fit** → ch02
- **Product Reviews** → ch05
- **RICE Framework** → ch02
- **Seven Superpowers of AI/GenAI** → ch01
- **Synthetic vs Real Data** → ch05
- **System Health Metrics** → ch06
- **Trade Space** → ch03
- **UI Patterns (agents)** → ch08
- **0-to-1 vs 1-to-n** → ch01, ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and decision frameworks
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision rules

---

## Scope & Limits

This skill covers the book's 8 main chapters only. Per the requesting user's instruction, the Appendix (product review/worksheet templates) and Index were deliberately excluded from extraction. For hands-on implementation in your own product org, combine with project-specific tools. For topics beyond this book, check related skills or ask the agent directly.
