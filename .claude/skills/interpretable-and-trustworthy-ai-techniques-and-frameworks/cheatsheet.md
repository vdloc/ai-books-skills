# Cheatsheet

## Explainer selection by data type
| Data type | First choice | Why |
|---|---|---|
| Tabular/structured | SHAP | Consistency, fairness guarantees, handles correlated features better than PDP |
| Text | LIME | Fast local perturbation on token/phrase level |
| Images | Grad-CAM (+ attention maps) | Visual, spatial explanation of CNN focus regions |
| Need both local + global | SHAP or DALEX pyramid | LIME is local-only |
| Need speed over rigor | LIME | Faster approximation, less exact |
| Need audit-defensible explanation | SHAP | Game-theoretic consistency guarantee |

## GAN variant selection
| Symptom | Fix |
|---|---|
| Training unstable / mode collapse | Switch to WGAN / WGAN-GP (Wasserstein loss, gradient penalty) |
| No paired training data | CycleGAN (cycle consistency loss) |
| Need class/attribute control | cGAN / AC-GAN |
| Paired image-to-image translation | Pix2Pix (add InstructPix2Pix for text-guided edits) |
| Need high-res, style-disentangled output | StyleGAN / StyleGAN2 |
| Rare-class data scarcity (medical imaging) | Conditioned StyleGAN + composite loss (SkinGAN pattern) |

## Bias mitigation by intervention point
| You control... | Use |
|---|---|
| Data collection | Pre-processing: reweighting, oversampling, synthetic data, fair representation learning |
| Model training | In-processing: adversarial debiasing, fairness constraints/regularization |
| Only the frozen model's output | Post-processing: calibration, equalized-odds threshold adjustment, re-ranking |

## Privacy technique by exposure point
| Exposure point | Technique |
|---|---|
| Aggregate query/model output | Differential Privacy (tune epsilon) |
| Training across institutions/devices | Federated Learning |
| Computation on untrusted infrastructure | Homomorphic Encryption |
| Joint computation across parties | SMPC |
| Publishing/sharing a dataset | k-anonymity / l-diversity / tokenization |

## Interpretable-model-first decision rule
1. Can a decision tree, linear/logistic regression, GAM, or rule-based system hit acceptable accuracy? → **Use it.** (Ch18 Tenet 3: no general accuracy/interpretability trade-off exists.)
2. Not sure? → **Run a Rashomon-set check**: train several model families; if performance clusters tightly, a simpler interpretable model likely exists in the set.
3. Must use a complex/black-box model? → **Wrap with SHAP/LIME**, but never treat the wrapper's explanation as ground truth (it can be wrong — "double black-box" risk).
4. High-stakes decision (health, legal, safety)? → **Prefer intrinsic interpretability over "explained" black box** (Tenet 5) — post-hoc explanations are unverifiable without model internals.

## HITL oversight level by stakes
| Decision stakes/reversibility | Level |
|---|---|
| Routine, reversible | Human Supervision (AI recommends, human confirms) |
| Moderate, analyst-assisted | Human Cooperation (AI assists, human can intervene) |
| High-stakes, hard to reverse (legal, medical) | Human Override (AI informs, human retains full discretion) |

## GAN evaluation — never trust one metric
| Metric | Blind spot |
|---|---|
| Inception Score (IS) | Misses intra-class mode collapse |
| FID | Biased estimator, depends on chosen CNN |
| KID | Unbiased but high variance |
Always combine ≥2 metrics + human evaluation for high-stakes generative content.

## Fairness metric thresholds to know
- **Statistical parity**: positive outcomes at similar rates across demographic groups.
- **Equalized odds**: equal false-positive/false-negative rates across groups.
- **Disparate impact ratio**: standard fairness-audit metric — investigate a model whenever its bias-mitigation coverage gap is large (chapter cites ~80% of orgs recognize fairness importance vs. only ~20% having implemented measures — don't be in that gap).

## Tells & smells
- "We need the black box for accuracy" **without** having tested a Rashomon-set of simpler models → unverified claim, not an established fact.
- LIME explanation changes noticeably on reruns of the *same* prediction → known stability issue; don't act on a single run for high-stakes decisions.
- SHAP explanation computed with a poorly chosen background dataset → results are sensitive to this choice; document it.
- Model's explanation layer bolted on *after* deployment in a clinical/safety pipeline → architecture smell; explanation generation should run parallel to prediction from day one.
- GAN augmenting a rare class *without* diversity loss → risk of mode collapse defeating the point of augmentation.
