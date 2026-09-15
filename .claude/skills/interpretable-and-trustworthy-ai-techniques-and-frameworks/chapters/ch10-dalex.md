# Chapter 10: DALEX (Model Agnostic Exploration, Explanation and Learning Implementation in Interpretable AI)

## Core Idea
DALEX treats interpretability not as one explanation but as an iterative, multi-level exploration ("the XAI/EMA pyramid") that moves from a single prediction, up through variable attribution, sensitivity, and local fit quality, to whole-model diagnostics — and it deliberately avoids claiming any single explanation is "the" answer.

## Frameworks Introduced
- **XAI/EMA Pyramid**: single-prediction level (SHAP, LIME, Break Down — "why did the model predict X for this instance") → variable-sensitivity level (Ceteris Paribus/predict_profile — "how does changing one feature change this prediction") → local-fit-quality level (residual diagnostics — "is the model unusually wrong for cases like this one") → whole-model level (accuracy metrics: F1/MSE/AUC, then permutation-based variable importance, then Partial/Accumulated Dependence Profiles).
  - When to use: as a drill-down map — start at whichever level matches your question (one player's value vs. whole-model behavior) and move up/down the pyramid as follow-up questions arise; don't stop at the first explanation level reached.
- **Permutation-based variable importance**: model-agnostic importance measure — compute baseline loss L₀, then for each feature, permute (shuffle) that column, recompute the loss L*ⱼ on the permuted data, and score importance as vipⱼ = L*ⱼ − L₀ (or the ratio L*ⱼ/L₀). A feature that matters a lot produces a large loss increase when shuffled.
  - When to use: for global, model-agnostic feature importance that works identically across linear models, random forests, and neural networks (unlike model-specific importance measures such as regression coefficients or impurity decrease).
- **Aspect-based importance**: permute multiple related features together (e.g. "fare" and "class," both proxies for socioeconomic status) to measure their *combined* importance rather than each in isolation — useful when features are conceptually entangled.

## Key Concepts
- **explain() wrapper**: DALEX's universal adapter — wraps any trained model (regardless of framework: scikit-learn, H2O, TensorFlow, randomForest, GBM) into a consistent interface so the same explainer functions work across model types.
- **Break Down**: DALEX's default local attribution method, alternative to Shapley values, decomposing one prediction into additive feature contributions.
- **Ceteris Paribus profile**: shows how a single prediction would change if one feature varied while all others stayed fixed ("all else equal") — DALEX's instance-level sensitivity tool.
- **Partial Dependence Profile**: the model-level (not instance-level) counterpart of Ceteris Paribus — shows how the model's average prediction changes as one feature varies, across the whole dataset.
- **DALEX family**: DALEXtra (connectors to scikit-learn/H2O/mlr/caret/tensorflow/keras), modelStudio (interactive dashboard), modelDown (static HTML report), iBreakDown (specialized local attribution auditor).

## Mental Models
- Treat interpretability as a process, not a deliverable — DALEX's own philosophy is explicit: "We are not searching for a single explanation that will provide all the answers... our comprehension of the models will grow with each subsequent stage." Design your own model audits the same way — iterative, multi-level, never "done" after one SHAP plot.
- Use model-level (model_parts/model_profile) vs. instance-level (predict_parts/predict_profile) functions as a matched pair — DALEX names them so the same underlying method applies at both scopes; if you find yourself asking a global question with an instance-level function (or vice versa), you have the wrong tool.

## Anti-patterns
- **Treating DALEX (or any interpretability tool) as a bias-fixing tool**: the chapter is explicit that "DALEX by itself does not ensure that the simulation will be totally free of prejudice" — it surfaces bias signals, it doesn't remove bias; pair with dedicated fairness-mitigation techniques (Ch3).
- **Reporting a single permutation-importance run as definitive**: permutation involves randomness, so results vary run to run — the chapter recommends repeating the process to quantify uncertainty around the importance values, not trusting one pass.
- **Picking one loss function L() and treating the resulting importance ranking as absolute**: the chapter notes "no one 'absolute' metric exists" — the choice of loss function (RMSE, cross-entropy, AUC-based) shapes which features rank as important.

## Worked Example
The chapter's FIFA 19 walkthrough demonstrates the full pyramid on a GBM model predicting player transfer value. The model predicts Cristiano Ronaldo's value at ~€48M (`predict(fifa_exp, cr7)`). Moving down the pyramid: `predict_parts(fifa_exp, cr7)` (Break Down) shows which features drove that number; switching `type='shap'` on the same call cross-checks with Shapley values to see if feature attribution agrees across methods. `predict_profile(fifa_exp, cr7)` (Ceteris Paribus) reveals that near-perfect BallControl substantially raises the prediction and that the model penalizes players over 30. A residual-distribution check shows CR7-like players have *larger* residuals than "average" players — a local-fit-quality flag the chapter says should make you "more wary" of the prediction's precision. Finally `model_parts()` (global permutation importance) shows Reactions is the least important feature overall, while BallControl and Age dominate — cross-validating the instance-level finding at the whole-model level. This is the pyramid used end-to-end: prediction → attribution → sensitivity → local fit → global importance.

## Key Takeaways
1. Don't stop at one explanation level — the XAI/EMA pyramid is a drill-down map from single-prediction to whole-model; follow-up questions belong at a different pyramid level, not a re-run of the same method.
2. Cross-check attribution methods (Break Down vs. Shapley) on the same instance — divergence is itself informative about model behavior.
3. Always check local fit quality (residuals for similar cases), not just the point prediction — a confident-looking prediction can sit in a region of unusually high model error.
4. Use permutation-based importance for model-agnostic global rankings, but repeat it to quantify randomness-driven variance, and be explicit about which loss function you chose.
5. DALEX surfaces bias/fairness signals but does not fix them — treat its output as diagnostic input to a separate fairness-mitigation process (Ch3).

## Connects To
- **Ch8, Ch9**: DALEX's Break Down and SHAP-mode functions directly overlap with LIME (Ch8) and SHAP (Ch9) — DALEX packages both as interchangeable attribution methods within one pyramid.
- **Ch3**: DALEX's explicit "does not ensure bias-free models" caveat points directly to Ch3's dedicated bias-mitigation framework.
