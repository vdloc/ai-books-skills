# Glossary

**AC-GAN (Auxiliary Classifier GAN)** — cGAN variant adding a classification task in the discriminator to improve class-conditional generation (Ch6).

**Adaptive Instance Normalization (AdaIN)** — modulates synthesis-network features per layer using an intermediate latent vector, giving StyleGAN control over texture/color/pattern at each resolution (Ch6, Ch17).

**Adversarial attack** — crafted input perturbation causing a model to misclassify; types include evasion, data poisoning, and model inversion (Ch3, Ch13).

**Adversarial training** — hardening a model by training it on adversarially perturbed examples (Ch3, Ch6, Ch13).

**AGI (Artificial General Intelligence)** — AI generalizing across tasks at human cognitive level without significant retraining (Ch1).

**ASI (Artificial Super Intelligence)** — intelligence surpassing humans in every domain, capable of recursive self-improvement (Ch1).

**Break Down** — DALEX's default local prediction-attribution method, an alternative to Shapley values (Ch10).

**Capsule Network** — architecture pooling neurons into "capsules" representing an object part's pose/attributes, embedding compositional structure (Ch4, Ch18).

**Ceteris Paribus profile** — DALEX tool showing how one prediction changes as a single feature varies, all else held constant (Ch10).

**Certified robustness** — formal, mathematically provable guarantee that a model resists perturbations within a defined bound (e.g., randomized smoothing) (Ch3, Ch13).

**CIA Triad (data privacy)** — Confidentiality, Integrity, Availability — the three properties of secure AI data handling (Ch13).

**Clever Hans effect** — a model reaches correct-looking predictions for spurious/wrong reasons (Ch18).

**Cycle consistency loss** — CycleGAN's loss ensuring domain-A→B→A translation reconstructs the original image, enabling unpaired training (Ch6, Ch7).

**Data provenance** — tracking a dataset's origin, modifications, and movement through an AI pipeline (Ch13).

**DALEX** — model-agnostic explanation package organizing interpretability into the "XAI/EMA pyramid," from single-prediction to whole-model diagnostics (Ch10).

**Deepfake** — AI-generated (typically GAN-based) synthetic media realistic enough to convincingly impersonate real people/events (Ch6, Ch7).

**Differential Privacy (DP)** — adds calibrated statistical noise to outputs so individual data points can't be inferred; epsilon (ε) controls the privacy/accuracy trade-off (Ch13).

**Disentanglement** — organizing a neural network's latent space so each dimension/neuron corresponds to a distinct, human-interpretable concept (Ch18).

**Federated Learning** — trains a shared model across decentralized devices/institutions without centralizing raw data (Ch13, Ch14).

**Feature Matching Loss** — GAN loss comparing intermediate discriminator activations (not just pixels) between real and generated data (Ch17).

**Fréchet Inception Distance (FID)** — GAN evaluation metric comparing feature-distribution distance between real and generated images; biased estimator (Ch6, Ch7).

**GAM (Generalized Additive Model)** — flexible extension of linear models using visualizable univariate component functions per feature (Ch18).

**Gram matrix** — captures spatial feature correlations; basis of style-loss computation in GANs (Ch17).

**Grad-CAM** — gradient-weighted class activation mapping; visualizes which image regions drove a CNN's prediction (Ch2, Ch3).

**Homomorphic Encryption (HE)** — enables computation directly on encrypted data without decrypting it (Ch13).

**Inception Score (IS)** — GAN evaluation metric using classifier-prediction entropy; misses intra-class mode collapse (Ch6, Ch7).

**Interpretable ML (vs. XAI)** — models transparent by construction (decision trees, scoring systems) vs. post-hoc explanation of black boxes; historically distinct fields, often conflated (Ch18).

**k-Anonymity / l-Diversity** — anonymization ensuring each record is indistinguishable from ≥k−1 others, with sensitive-attribute diversity within groups (Ch13).

**LIME (Local Interpretable Model-agnostic Explanations)** — perturbs an input, fits a local interpretable surrogate model, explains one prediction at a time (Ch2, Ch3, Ch8, Ch15, Ch16).

**Membership inference attack** — adversary determines whether a specific individual's data was used in training, from model outputs alone (Ch13).

**Minimax game** — GAN's adversarial training objective: generator minimizes, discriminator maximizes the same loss, converging toward Nash equilibrium (Ch6, Ch7).

**Mode collapse ("Helvetica scenario")** — GAN generator produces a narrow subset of outputs instead of full data diversity (Ch6, Ch7).

**Nash equilibrium (GAN)** — training state where the discriminator cannot outperform random guessing on real vs. generated data (Ch6, Ch7).

**Partial Dependence Plot (PDP)** — shows a feature's isolated marginal effect on model predictions, assuming feature independence (Ch10, Ch15, Ch16).

**PatchGAN** — Pix2Pix's discriminator, classifying image patches rather than whole images as real/fake (Ch6).

**Permutation-based variable importance** — model-agnostic importance measure: shuffle one feature, measure the resulting loss increase (Ch10).

**PINN (Physics-Informed Neural Network)** — trained to minimize residuals from governing differential equations plus data loss, enabling physically-constrained, often unsupervised training (Ch18).

**Rashomon set** — the set of models within a loss threshold ε of the best-achievable performance; a large Rashomon set usually contains an interpretable member (Ch18).

**Right to explanation (GDPR)** — legal requirement that individuals be informed of the logic behind automated decisions affecting them (Ch3, Ch11, Ch13, Ch14, Ch15, Ch16).

**SaMD (Software as a Medical Device)** — FDA regulatory category for AI/ML healthcare software, requiring validated performance plus a predetermined change-control plan (Ch14).

**Shapley value** — game-theoretic fair-attribution measure: a feature's average marginal contribution across all possible feature-subset combinations (Ch9, Ch10).

**SHAP (SHapley Additive exPlanations)** — model-agnostic method assigning each feature a fair contribution score to a prediction, supporting both local and global interpretation (Ch2, Ch3, Ch9, Ch15, Ch16).

**SMPC (Secure Multi-Party Computation)** — multiple parties jointly compute a function over combined data without revealing individual inputs to each other (Ch13).

**Style loss** — GAN loss comparing Gram-matrix feature correlations to replicate texture/color patterns (Ch6, Ch17).

**Trustworthy AI** — AI exhibiting reliability + security + fairness + explainability simultaneously (Ch11, Ch12).

**WGAN (Wasserstein GAN)** — replaces standard GAN loss with Wasserstein (Earth Mover) distance for smoother gradients and reduced mode collapse (Ch6, Ch7).

**XAI/EMA Pyramid** — DALEX's hierarchical interpretability structure from single-prediction attribution up to whole-model diagnostics (Ch10).

**Zero-Knowledge Proof (ZKP)** — lets one party prove a statement is true without revealing the underlying data (Ch13).
