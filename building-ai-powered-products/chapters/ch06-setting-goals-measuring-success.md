# Chapter 6: Setting Goals and Measuring Success

## Core Idea
No single metric captures AI product success — you need the "AI product metric blend" of product health, system health, and AI proxy metrics working together, then translate that blend into OKRs where each objective is anchored by exactly one North Star metric plus supporting KPIs from every bucket.

## Frameworks Introduced
- **The AI Product Metric Blend (three buckets)**: Nika's foundational model for evaluating any AI feature's success — no bucket alone is sufficient.
  - When to use: any time you're deciding whether an AI feature is "working" or ready to launch.
  - How: **Product health metrics** (your direct responsibility) — engagement, user satisfaction (NPS/surveys), adoption, conversion, retention/churn, financial metrics (revenue, ROI). **System health metrics** (technical, usually not directly owned but must be tracked) — uptime, latency, scalability, error rate. **AI proxy metrics** (model-specific, also usually not directly owned) — model quality metrics (accuracy, precision, sensitivity, recall, ROC curve), objective/loss functions (e.g. MSE), confusion matrices.
- **A Framework for Crafting AI Product OKRs**: Nika's structured template requiring representation from all three metric buckets in every OKR.
  - When to use: quarterly goal-setting for any AI feature or product.
  - How: fill in six components — (1) **Objective**: user-focused, explicit "who is it for" statement of desired outcome; (2) **Specific features**: what you'll ship to achieve it; (3) **North Star (KPI)**: the single primary success metric — exactly one per OKR, though supporting metrics provide context; (4) **Product health metrics (KPIs)**: retention, satisfaction, adoption; (5) **Guardrail metrics (KPIs)**: adverse side effects to monitor and cap (e.g. "don't let listening time drop more than 5%"); (6) **System health metrics (KPIs)**: uptime, latency; (7) **AI proxy metrics (KPIs)**: model accuracy/precision/recall. Rule: include at least one metric from each of the three buckets in every OKR — this is what keeps goals from being lopsided toward only what's easy to measure.
- **Confusion Matrix**: the standard 2×2 evaluation table for any binary classification model — the foundation under precision/recall/sensitivity.
  - When to use: evaluating any classifier (spam detection, fraud detection, content moderation).
  - How: four cells — True Positive (correctly flagged), True Negative (correctly cleared), False Positive (incorrectly flagged, Type I error), False Negative (incorrectly cleared, Type II error). Precision = TP / (TP + FP) — how trustworthy are positive predictions. Recall/Sensitivity = TP / (TP + FN) — how many real positives did you catch. High precision minimizes false positives; high recall minimizes false negatives — these trade off against each other and the right balance depends on which error is more costly for your use case (e.g. medical diagnosis wants high recall even at some precision cost).

## Key Concepts
- **North Star metric**: the single KPI that captures the core value an AI product delivers — every OKR gets exactly one, even though it may have many supporting KPIs.
- **Guardrail metric**: a KPI that caps an acceptable side effect of optimizing the North Star (e.g. don't let overall listening time drop while boosting playlist engagement) — prevents over-optimizing one metric at the expense of overall product health.
- **Proxy metric**: any metric that approximates the true goal but isn't the goal itself — named "proxy" because model accuracy, precision, etc. measure the model's behavior, not the user's actual satisfaction.
- **Evals**: the evaluation runs comparing a live model against offline candidate versions, used iteratively post-deployment to decide whether a new model is a significant-enough improvement to launch.
- **Loss function / MSE (mean squared error)**: the training-time proxy metric that measures how far predictions are from actual values, squared to penalize larger errors more — minimized during model training to improve accuracy.
- **Churn**: the inverse of retention; understanding why users leave is treated as equally important to understanding why they stay.

## Mental Models
- Think of the three metric buckets as a three-legged stool: leaning on only one (e.g. optimizing AI proxy metrics like precision while ignoring product health) produces a technically excellent model that still fails as a product.
- Use guardrail metrics as your explicit answer to "what am I willing to sacrifice, and how much" — write the cap into the OKR itself rather than discovering the trade-off after shipping.
- Treat the confusion matrix's precision/recall trade-off as context-dependent, not a universal target — ask "which error type is more costly here" before deciding which to optimize.

## Anti-patterns
- **Relying on a single metric to judge AI feature success**: explicitly called out as insufficient — always blend product health, system health, and AI proxy metrics.
- **OKRs with no guardrail metric**: optimizing a North Star metric without a cap on side effects risks winning the metric while damaging the broader product (e.g. boosting engagement with recommended playlists while overall listening time silently drops).
- **Confusing model quality with product success**: a model can hit strong precision/recall numbers while the feature it powers still fails on product health metrics — proxy metrics are proxies, not the goal itself.
- **Setting more than one North Star metric per OKR**: dilutes focus; the framework is explicit that each OKR gets exactly one.

## Worked Example
**Full OKR built with the framework, for a streaming music service's recommendation system** (Table 6-2, reproduced):

| Component | Example |
|---|---|
| Objective | Enhance the user experience by providing more personalized music recommendations |
| Specific feature | Introduce three new personalization algorithms based on user behavior, mood, and music trends |
| North Star metric | Increase user engagement with recommended playlists by 25% |
| Product health metric | Reduce users skipping songs in AI-generated playlists by 20% |
| Guardrail metric | Overall listening time does not decrease by more than 5% |
| System health metric | Maintain 99% uptime, playlist loading under 1 second |
| AI proxy metric | Increase recommendation algorithm precision by 15% |

Notice all three buckets are represented, there's exactly one North Star, and the guardrail metric explicitly protects against the North Star being gamed at the expense of overall product health.

**Confusion matrix walkthrough (spam classifier)**: of 100 actual spam emails, the algorithm correctly flags 80 (true positives) and misses 20 (false negatives) → 80% recall. Precision would separately ask: of everything the algorithm *flagged* as spam, what fraction was actually spam (few false positives = high precision).

## Key Takeaways
1. Never judge AI feature success on a single metric — blend product health, system health, and AI proxy metrics into a holistic view.
2. Give every OKR exactly one North Star metric, and always include a guardrail metric to cap acceptable side effects of chasing it.
3. Ensure every OKR draws at least one KPI from each of the three metric buckets — this is the mechanical check that prevents lopsided goal-setting.
4. Distinguish precision from recall explicitly, and choose which to prioritize based on the cost of each error type for your specific use case (e.g. medical diagnosis favors recall).
5. Treat AI proxy metrics (accuracy, precision, MSE) as means, not ends — a model can excel on proxy metrics while the product still fails on health metrics like retention or satisfaction.
6. Use "evals" (live vs. offline model comparisons) as your standard decision gate for whether a retrained model is worth launching, tying back into the AIPDL's iterative validation loop.

## Connects To
- **Ch 3**: Model quality metrics (accuracy, precision, recall) extend the validation/testing stage of the AI lifecycle covered there.
- **Ch 2**: The North Star / OKR structure here is what ultimately measures whether the RICE-prioritized features from Ideation delivered their promised impact.
- **Ch 5**: Status-update product reviews (Ch. 5) are where these OKRs and their KPI trends get presented to leadership.
