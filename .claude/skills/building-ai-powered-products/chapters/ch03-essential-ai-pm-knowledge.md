# Chapter 3: Essential AI PM Knowledge

## Core Idea
An AI PM's job is to translate between four skill buckets — core PM craft, engineering foundations, leadership/collaboration, and AI-lifecycle/operational awareness — and the chapter's real payload is the trade-off vocabulary and the AI lifecycle's five technical stages (scoping → data collection → model training → validation/testing → deployment) that every AI PM must be fluent in without needing to code.

## Frameworks Introduced
- **The Trade Space (six-step guide)**: Nika's method for navigating AI's competing constraints as a continuous space, not a binary choice.
  - When to use: whenever a decision (build vs. buy, on-device vs. cloud, accuracy vs. speed) has more than two clean options.
  - How: (1) Identify key factors (cost, time, expertise, risk, at minimum) by talking to stakeholders/scientists/engineers, (2) Rank priorities — what's non-negotiable vs. compromisable, (3) Map interdependencies between trade-offs, (4) Visualize the trade space as a matrix/graph, (5) Test scenarios by simulating decisions, (6) Iterate — revisit regularly as the project evolves.
- **Seven Named AI-Specific Trade-offs**: the vocabulary for framing any AI product decision to stakeholders.
  - When to use: writing an executive summary or product review for a contested AI decision.
  - How: Accuracy vs. speed; Complexity vs. simplicity; Data quality vs. quantity; Generalization vs. specificity; User privacy vs. personalization; Ethical considerations vs. business goals; Explainability vs. performance. Each names two sides + the stakes of over-indexing on either.
- **Build vs. Buy — eight decision factors**: a structured checklist instead of a gut call.
  - When to use: any "should we build this AI capability or license it" decision.
  - How: score across cost-benefit ratio, expertise/talent availability, time to market, risk/uncertainty, data privacy & ethics control, scalability & maintenance, competitive landscape, and alignment with core business goals — the more of these that favor building, the stronger the case for in-house.
- **The AI Lifecycle (five technical stages)**: the model-training mini-lifecycle first mentioned in Ch. 2's MVP discussion, expanded here — nested inside the AIPDL's Concept/Prototype stage.
  - When to use: as your mental checklist whenever data scientists say "we're working on the model."
  - How: (1) Project scoping — translate the PRD into technical boundaries and explicit out-of-scope items; (2) Data collection — source from internal DBs, third-party APIs, user-generated content, public repos, sensor/IoT data, data vendors, or (last resort) synthetic data, then clean/label/preprocess; (3) Model training — feed data through an algorithm to produce a model, iterating on algorithm choice and hyperparameters; (4) Validation & testing — check generalization on held-out data, loop back to training until the model hits **Minimum Viable Quality (MVQ)**; (5) Deployment — move the validated model into production and wire it into the product's live infrastructure.
- **Four ML Learning Methods**: the quadrant map (Figure 3-5 in the book) an AI PM uses to match a use case to the right underlying technique.
  - When to use: scoping a feature with data scientists — lets you ask "should this be supervised or unsupervised?" instead of just "make it smart."
  - How: **Supervised learning** (labeled data; classification/regression — fraud detection, medical diagnostics, forecasting). **Self-supervised learning** (model generates its own labels; powers LLMs/transformers — chatbots, content synthesis). **Unsupervised learning** (no labels; clustering/dimensionality reduction — anomaly detection, customer segmentation). **Reinforcement learning** (agent learns via reward/penalty; control/optimization — financial trading, robotics, Netflix's multi-armed-bandit recommendations).
- **FATE Framework (Fairness, Accountability, Transparency, Ethics)**: Nika's go-to ethical lens for Responsible AI practices, paired with the AI Ethics Canvas.
  - When to use: at every AI lifecycle stage, not just before launch — ask "who will this impact?" and "what potential harms might arise?" up front.
  - How: audit dataset fairness, test model outputs for bias, run scenario analysis for unintended uses, mitigate via diverse datasets and edge-case stress-testing, and instrument ongoing bias/impact monitoring post-launch.

## Key Concepts
- **Algorithm vs. Model**: an algorithm is the general rule set (e.g. decision trees, regression); a model is a specific trained instance of an algorithm on a specific dataset — the same algorithm produces different models on different data.
- **Minimum Viable Quality (MVQ)**: the quality threshold (set by the PM, varies by use case — e.g. NPS/CSAT for a recommender vs. 95% accuracy for a medical diagnostic tool) at which the AI lifecycle stops iterating and the product ships.
- **Human-in-the-loop (lifecycle-wide)**: not a single stage but a thread through all five — humans label training data, evaluate validation results, and provide post-deployment feedback that feeds retraining.
- **MLOps**: the team/discipline typically responsible for large-scale data-collection pipelines feeding model training.
- **Explainable AI (XAI) techniques**: feature-importance scores, visualization (decision trees, heatmaps), and counterfactual explanations ("if your income were $5,000 higher...") — used both for user trust and for engineers debugging biased predictions.
- **Data preprocessing**: the cleaning + labeling + structuring step between raw data collection and model training; treated as a continuous, evolving process, not a one-time task.

## Mental Models
- Use the trade space as a literal visual (a matrix/graph per decision) whenever a trade-off has more than two obvious sides — don't try to hold it in your head.
- Treat "algorithm vs. model" as your test for whether you're communicating precisely with data scientists — conflating the two is a tell that you haven't internalized the distinction.
- Default to asking "what's the MVQ for this specific use case?" before a launch debate starts — a chatbot's MVQ (user satisfaction) and a medical tool's MVQ (95%+ accuracy) are not comparable, and conflating them produces bad launch decisions.

## Anti-patterns
- **Treating trade-offs as binary A-or-B choices**: real AI decisions are multidimensional; force-fitting them into a single either/or loses the nuance the trade-space model is built to capture.
- **Skipping the "out of scope" declaration during project scoping**: invites scope creep because cross-functional partners don't share an explicit boundary.
- **Using synthetic data as a first resort**: the book explicitly flags synthetic data generation as suboptimal, to be used only when real data is scarce or too sensitive to collect.
- **Treating XAI as a launch-gate checkbox**: explainability serves both users (trust) and engineers (debugging) — deferring it until pre-launch compliance review misses its debugging value during model training.

## Worked Example
**Build-vs-buy trade space: on-device vs. cloud processing** (Table 3-1, reproduced compactly): an executive-summary-style comparison across five factors —

| Factor | On-device | Cloud |
|---|---|---|
| User Experience | + low latency, offline-capable / – limited by device hardware | + supports advanced models, scales easily / – network-dependent |
| Ethics & Privacy | + data stays local / – device loss/hack risk | + centralized auditing / – aggregation increases misuse risk |
| Compliance | + easier GDPR/HIPAA (data stays local) / – hardware-dependent variance | + simplifies global compliance / – cross-border data-flow restrictions |
| Resource Constraints | + lower ongoing cost / – high upfront hardware cost | + pay-as-you-go / – ongoing infra opex |
| Technology Constraints | + network-independent / – needs lightweight optimized models | + supports cutting-edge, compute-heavy models / – network failure point |

Pattern to reuse: build this table for any build-vs-buy or architecture decision, then close with an explicit recommendation + justification — this is the structure the book recommends for a product-review executive summary.

## Key Takeaways
1. Use the seven named trade-offs (accuracy/speed, complexity/simplicity, data quality/quantity, generalization/specificity, privacy/personalization, ethics/business goals, explainability/performance) as your default vocabulary for framing AI product decisions to stakeholders.
2. Build a trade space (six-step method) instead of forcing multidimensional AI decisions into binary choices.
3. Know the five AI lifecycle stages (scoping, data collection, model training, validation/testing, deployment) well enough to ask data scientists informed questions — you don't need to code, but you need lifecycle fluency.
4. Set an explicit, use-case-specific MVQ before debating whether a model is "good enough" to launch — a chatbot and a medical diagnostic tool have very different bars.
5. Match feature ideas to the right learning method (supervised / self-supervised / unsupervised / reinforcement) before scoping a solution with your data science team.
6. Embed FATE (fairness, accountability, transparency, ethics) checks at every lifecycle stage, not just pre-launch — bias mitigation via diverse datasets and edge-case testing is cheaper early than after deployment.
7. Prefer real data sources (internal DBs, APIs, public repos, user-generated content) over synthetic data generation, which the book treats as a last resort.

## Connects To
- **Ch 2**: The AI lifecycle detailed here is the technical core of the AIPDL's Concept/Prototype stage and the AI MVP's model-training component.
- **Ch 4**: The build process tools (MLflow, Zapier) and estimation frameworks here inform the day-to-day workflow covered next.
- **Ch 5**: Build-vs-buy and trade-space thinking feed directly into Chapter 5's strategic road-mapping decisions.
