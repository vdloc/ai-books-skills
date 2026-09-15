# Patterns

## LIME for fast local explanation
**When to use**: need a quick, instance-specific explanation for any black-box model; speed matters more than mathematical rigor.
**How**: perturb the input around the instance, weight perturbed samples by proximity, fit an interpretable surrogate (linear/tree), read off feature contributions.
**Trade-offs**: fast and model-agnostic, but locally faithful only — extrapolating to global model behavior misrepresents the method; stability across repeated runs is a known weakness. (Ch2, Ch3, Ch8)

## SHAP for defensible, consistent attribution
**When to use**: the explanation must hold up under audit/regulatory review, or you need both local and global views from one method.
**How**: compute Shapley values (average marginal contribution across all feature-subset combinations); sum of contributions equals deviation from average prediction.
**Trade-offs**: theoretically consistent and fair, but computationally expensive at scale; sensitive to feature correlation and choice of background dataset. (Ch2, Ch3, Ch9, Ch15, Ch16)

## SHAP-based feature selection
**When to use**: need to reduce a high-dimensional feature set while keeping the selection criterion itself interpretable (e.g., clinical feature sets).
**How**: train full-feature model → compute SHAP values → rank by mean absolute SHAP value → threshold or recursively eliminate → cross-validate reduced set → domain-expert review.
**Trade-offs**: usually trades a small accuracy loss for large interpretability/efficiency gain — not guaranteed to beat the full-feature model on raw accuracy. (Ch9)

## GAN + explainer for post-hoc trust in generative models
**When to use**: GAN outputs (images, synthetic data) are deployed in a high-stakes domain (healthcare, finance) and stakeholders need to trust them.
**How**: apply latent-space visualization, attention mapping, and adversarial/bias testing to the generator and discriminator; combine with counterfactual reasoning.
**Trade-offs**: GANs have two black boxes (generator + discriminator) whose interaction — not just individual weights — produces output, making this harder than standard DNN interpretability. (Ch6)

## Conditioned GAN synthesis for rare-class augmentation
**When to use**: severe class imbalance in a domain where basic augmentation (rotate/flip) and transfer learning both fail to capture the rare class's distinct morphology (e.g., rare disease imaging).
**How**: use abundant-class data as the main dataset, rare-class examples as conditioning input to a StyleGAN-based generator; apply a composite loss (feature matching + perceptual + content consistency + style + diversity + optional segmentation) rather than a single adversarial loss.
**Trade-offs**: dramatically improves downstream classifier sensitivity, but requires validating on the actual clinical/task metric, not just image-quality scores (FID/IS). (Ch17)

## Human-in-the-Loop (HITL) escalation by stakes
**When to use**: any AI-assisted decision where full automation carries unacceptable risk.
**How**: match oversight level to decision reversibility/stakes — Human Supervision (confirm before action) for routine flags, Human Cooperation (AI assists, human intervenes) for analyst workflows, Human Override (AI informs, human decides) for legal/high-stakes judgment.
**Trade-offs**: reduces bias/error risk but adds latency and cognitive load on human experts; scalability suffers in high-throughput settings (e.g., real-time fraud detection). (Ch3, Ch11, Ch15)

## Risk-based AI auditing
**When to use**: audit resources are constrained and you need to prioritize.
**How**: comprehensive risk assessment (privacy, bias, compliance) → categorize by likelihood × impact → prioritize highest-risk areas → continuously update as system/data evolve.
**Trade-offs**: efficient use of limited resources, but requires ongoing re-assessment — a one-time risk assessment goes stale as models and data drift. (Ch12)

## Privacy technique selection by exposure point
**When to use**: designing a privacy-preserving AI pipeline.
**How**: match the technique to the actual exposure point — DP for aggregate outputs, Federated Learning for training-time data locality, HE for computation on encrypted data, SMPC for cross-party joint computation, k-anonymity/l-diversity for published datasets.
**Trade-offs**: each technique protects a different stage; using the wrong one for your actual exposure point leaves the real gap unprotected. Techniques are commonly combined (e.g., DP + Federated Learning). (Ch13)

## Parallel explanation pipeline (healthcare/high-stakes architecture)
**When to use**: any clinical or safety-critical AI deployment.
**How**: design the explanation-generation pipeline to run parallel to the prediction pipeline from the start (not as a post-hoc retrofit); adapt explanation depth by stakeholder (specialist/generalist/patient/regulator).
**Trade-offs**: adds architectural complexity up front, but avoids latency issues from bolting on explanations after deployment, and ensures every prediction ships with its explanation simultaneously. (Ch14)

## Rashomon-set exploration before committing to a black box
**When to use**: facing an "we need the complex model for accuracy" argument.
**How**: train several model families (boosted trees, SVM, logistic regression, neural net) on the same task; if performance clusters closely, a large Rashomon set likely contains a simpler, comparably accurate, more interpretable model.
**Trade-offs**: requires upfront experimentation cost, but converts an assumed trade-off into an empirically tested one — often revealing the trade-off was unnecessary. (Ch18)

## Intrinsic interpretability over black-box + wrapper
**When to use**: maximal transparency/auditability is the priority and a moderate accuracy cost is acceptable (e.g., regulator-facing credit scoring, clinical protocol decisions).
**How**: default to decision trees, linear/logistic regression, GAMs, rule-based systems, or scoring systems; escalate to black-box + LIME/SHAP only when the accuracy gain clearly justifies the interpretability loss.
**Trade-offs**: simpler models may underperform on complex, non-linear, high-dimensional data — verify via the Rashomon-set check above rather than assuming the trade-off exists. (Ch11, Ch15, Ch16, Ch18)
