# Chapter 2: Interpretable and Trustworthy Sleep Pattern Analysis for Sleep Disorders Using Explainable AI (XAI) Techniques

## Core Idea
A black-box DL classifier for sleep-disorder diagnosis only becomes clinically usable once wrapped with XAI (SHAP, LIME, attention maps, saliency maps) — accuracy alone doesn't earn clinician trust in life-critical applications.

## Frameworks Introduced
- **XAI-wrapped clinical pipeline**: Data Collection/Preprocessing → Feature Extraction → XAI Integration (SHAP/LIME/Grad-CAM) → Model Interpretation & Evaluation.
  - When to use: any healthcare ML deployment where a clinician must sign off on individual predictions, not just aggregate accuracy.
  - How: train the classifier normally, then run post-hoc explainers (SHAP for global+local feature importance, LIME for fast local surrogate explanations, Grad-CAM for image/attention regions) and report fidelity + interpretability-size + unambiguity alongside accuracy.
- **Beta metrics for XAI evaluation**: fidelity (how well the explainer's local model matches the base model's predictions), interpretability size (number of features the explanation relies on), unambiguity (clarity of activations) — a 3-metric rubric for comparing explainers, not just comparing classifiers.
  - When to use: choosing between SHAP and LIME (or any two explainers) for a deployment — don't just eyeball plots, score fidelity/size/unambiguity.

## Key Concepts
- **SHAP (SHAPley Additive exPlanations)**: game-theoretic, model-agnostic method giving unified additive feature-importance values; supports both global and local interpretability.
- **LIME (Local Interpretable Model-agnostic Explanations)**: perturbs input around one instance, fits a simple local surrogate model, and reports per-feature contribution for that single prediction only (local, not global).
- **Grad-CAM**: gradient-weighted class activation mapping — visualizes which image regions drove a CNN's prediction.
- **Fidelity**: metric for how faithfully an explanation method's surrogate matches the underlying black-box model's actual predictions.
- **Unambiguity**: metric for how clear/non-overlapping an explanation's activations are.
- **BETA / CART**: intrinsically interpretable models (as opposed to post-hoc explainers) capable of producing global explanations directly.

## Mental Models
- Treat interpretability as a deployment gate, not an afterthought: "accuracy 98%" is not sufficient for a life-critical prediction — pair every deployed classifier with at least one post-hoc explainer.
- LIME vs. SHAP is a speed/fidelity trade-off: LIME is faster (chapter reports it "provide[d] explanations much quicker") but SHAP tends toward higher, more consistent fidelity (fidelity=1.0 across models in Table 2.1) — pick LIME for rapid iteration/debugging, SHAP when the explanation itself must be defensible (e.g., an audit trail).
- Use intrinsically interpretable models (BETA, CART) when you need a *global* explanation of model logic, not just per-instance justifications — a fundamentally different capability from post-hoc local explainers like LIME.

## Anti-patterns
- **Deploying a black-box classifier in a clinical setting without any explainer**: clinicians will not adopt a diagnosis tool they can't validate against their own reasoning — the chapter frames this explicitly as the adoption barrier XAI exists to remove.
- **Treating LIME's local explanation as if it generalizes**: LIME explanations are valid only for the single perturbed instance; extrapolating a LIME explanation to describe overall model behavior misrepresents what the method computes (use SHAP's global mode or an intrinsically interpretable model for that).

## Reference Tables
| Model | Fidelity | Interpretability Size (avg features) | Unambiguity |
|---|---|---|---|
| SHAP | 1.0 | 1364 | 0.1098 |
| LIME | 1.0 | 1364 | 0.0200 |
| BETA | 0.9123 | 1240 | 0.44706 |
| CART | 1.0 | 1240 | 0.001754 |

## Worked Example
The chapter's own pipeline: sleep data (374 patients — sleep duration, quality, physical activity, stress level, BMI, blood pressure, heart rate, daily steps) is fed to five classifiers (XGBoost, CatBoost, Logistic Regression, SVM, CART), each hitting 91-94% test accuracy. Rather than stopping at accuracy, every classifier is run back through both SHAP and LIME: SHAP surfaces "occupation" as the leading insomnia factor for the XGBoost model; LIME produces per-instance positive/negative-influence plots for each classifier. The BETA-metrics table then scores each explainer/model pair on fidelity, feature count, and unambiguity — concluding LIME and SHAP reach identical fidelity (1.0) but SHAP is slower and LIME faster, so the recommended production setup pairs a boosting classifier with LIME for real-time explanation and SHAP for periodic audit-quality review.

## Key Takeaways
1. In healthcare ML, always report an explainability metric (fidelity/unambiguity/interpretability-size) alongside accuracy — accuracy alone won't clear a clinical adoption bar.
2. SHAP gives consistent, high-fidelity, model-agnostic global+local explanations at higher compute cost; LIME is faster but strictly local — pick based on whether you need an audit trail (SHAP) or fast iteration (LIME).
3. Ensemble/boosting classifiers (XGBoost, CatBoost, Gradient Boosting) outperformed single models (Logistic Regression, SVM) on this task — pair boosting classifiers with post-hoc XAI rather than defaulting to intrinsically interpretable-but-weaker models.
4. Attention maps and saliency maps are complementary to SHAP/LIME for text/image models — they visualize *where* the model looked, while SHAP/LIME quantify *how much* each feature mattered.

## Connects To
- **Ch8 (LIME)**: this chapter is LIME's applied case study; Ch8 covers the underlying mechanics in more depth.
- **Ch9 (SHAP)**: same relationship — this chapter applies SHAP, Ch9 explains the method.
- **Ch14**: extends interpretable AI into healthcare more broadly (diagnostic frameworks beyond sleep disorders).
