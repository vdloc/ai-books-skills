# Chapter 4: The AI PM's Day-to-Day

## Core Idea
An AI PM's actual day-to-day is defined by two axes: where you sit on the AI PM career ladder (execution → AI/ML PM → strategic leadership), and how well you orchestrate the unusually wide set of cross-functional stakeholders an AI product requires (AI/ML teams, operations, engineering, UX, business, GRC, leadership) — the AI PM is "the glue," not a builder.

## Frameworks Introduced
- **The AI PM Career Ladder (levels 4–9+)**: Nika's illustrative map of how AI PM responsibilities shift with seniority — meant as a mirror, not a rigid hierarchy.
  - When to use: self-assessing career stage, or scoping what a given AI PM role actually expects of you.
  - How: **Execution level (4–6)** — day-to-day shipping, monitoring model performance and data quality, working directly with ML engineers/data scientists, owning OKRs. **AI/ML PM (5–7)** — defining product requirements, prioritizing roadmap, setting multiyear AI product vision, aligning data/infrastructure strategy with business goals. **Strategic leadership (8+)** — aligning AI products with overall business strategy, managing a portfolio, cross-functional governance/ethics/compliance; **9+** (head of AI Product, chief AI officer) — company-wide AI vision, multibillion-dollar investment decisions, org-wide responsible-AI governance.
- **Cross-Functional Stakeholder Map**: the full roster of teams an AI PM must actively coordinate, beyond what a traditional PM manages — used here via a worked Alexa example.
  - When to use: onboarding onto a new AI product, or scoping who needs to be in the room for a launch decision.
  - How: **AI/ML teams** (ML scientists build/train models; red/blue teams handle security; MLOps deploys and monitors models in production). **Operations** (program managers coordinate timelines/dependencies; DataOps collects/cleans/ensures compliant data). **Engineering** (developers integrate models into product architecture; testers validate real-world behavior; data engineers maintain pipelines; TPMs coordinate engineering execution). **UX** (user researchers surface pain points; designers translate AI functionality into intuitive interfaces; content specialists write user-facing AI copy). **Business** (PMMs own go-to-market messaging; sales relays customer feedback; partnership managers build external alliances). **Third-party stakeholders** (vendors/OEMs, consultants/research institutions). **GRC** (legal, privacy, compliance specialists — GDPR/CCPA). **Leadership** (C-suite sets strategic goals; investors provide funding and ROI scrutiny).

## Key Concepts
- **OKRs (Objectives and Key Results)**: the goal-setting mechanism execution-level AI PMs use day-to-day; expanded fully in Chapter 6.
- **Red/blue teams**: security function within AI/ML teams — red teams simulate attacks, blue teams defend, jointly hardening the AI product against real-world threats.
- **MLOps**: the operational discipline responsible for deploying and continuously monitoring ML models in production at scale.
- **DataOps**: the team ensuring training/deployment data is clean, integrated, and compliant (e.g. GDPR) — distinct from MLOps, which handles the models themselves.
- **TPM (Technical Program Manager)**: coordinates engineering timelines and resource allocation across teams building the AI product, distinct from the AI PM's product-vision role.

## Mental Models
- Use the ladder to diagnose scope creep: if you're at execution level but being asked to set multiyear vision (level 6-7 work), that's a signal your role or org structure needs clarifying.
- Treat the cross-functional map as a per-project checklist, not a permanent org chart — smaller companies collapse many of these roles into fewer people, but the *functions* (model building, security, data ops, UX, compliance, business alignment) still need covering.
- Model industry seniority patterns (Nika's Meta/Google observation): as companies scale AI orgs, the ladder tends to *compress* — fewer senior strategic roles, more consolidated execution-level ownership.

## Anti-patterns
- **Treating the career ladder as a strict hierarchy**: the book explicitly warns these levels are illustrative — real roles often blend multiple levels, especially at smaller companies.
- **Skipping GRC/legal engagement until pre-launch**: privacy, compliance, and legal stakeholders (GDPR, CCPA) should be part of ongoing cross-functional collaboration, not a late-stage gate.
- **AI PM trying to be the ML scientist**: the chapter's stakeholder map is explicit that ML scientists build/train models — the AI PM's job is translation and orchestration, not building the model themselves.

## Worked Example
**Cross-functional stakeholder mapping for an Amazon Alexa feature update** (the chapter's running example): an AI PM enhancing Alexa's daily-task-management ability must actively engage all seven stakeholder groups — ML scientists/red-blue teams/MLOps for the voice-recognition and NLU models; program managers/DataOps for pipeline and data compliance; developers/testers/data engineers/TPMs for integration into Alexa's architecture; user researchers/designers/content specialists for the interaction UX; PMMs/sales/partnership managers for go-to-market; legal/privacy/compliance for regulatory adherence; and C-suite/investors for strategic buy-in. The lesson: name every group explicitly before scoping a launch — smaller orgs collapse roles, but skipping a *function* (e.g. no explicit security red-team pass) is a gap, not a simplification.

## Key Takeaways
1. Identify your position on the AI PM career ladder (execution / AI-ML PM / strategic leadership) to calibrate what's actually expected of your role versus what you're being asked to do.
2. Build your stakeholder map before scoping a launch — AI products require materially more cross-functional coordination (ML scientists, red/blue security teams, MLOps, DataOps, GRC) than traditional software products.
3. Treat MLOps and DataOps as distinct functions: one operationalizes models, the other operationalizes data — both are prerequisites for reliable deployment.
4. Engage GRC (legal, privacy, compliance) stakeholders continuously, not as a pre-launch checkbox — their role is baked into the AI lifecycle, not appended to it.
5. Expect the AI PM career ladder to compress at scale (per Nika's Meta/Google observation) — fewer senior strategic roles concentrating vision, more consolidated execution-level ownership underneath.

## Connects To
- **Ch 3**: The five-stage AI lifecycle (scoping through deployment) maps directly onto which stakeholder group is most active at each stage.
- **Ch 5**: Strategic road-mapping and product reviews (mentioned here as "Getting Buy-in from Leadership") are covered in full next.
- **Ch 6**: OKRs, introduced here as an execution-level day-to-day tool, get a full framework treatment.
