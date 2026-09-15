# Patterns

## GAN/VAE Implementation Pipeline
**When to use**: any image-enhancement, denoising, inpainting, or style-transfer project.
**How**: Data Preparation (gather + clean + label images) → Model Selection (GAN for realism, VAE for structured latent-space reconstruction) → Training → Evaluation (PSNR, SSIM) → Deployment.
**Trade-offs**: GANs give sharper, more realistic output but unstable training; VAEs are more stable but blurrier/lower-resolution.
*(Ch1)*

## Domain-Specific LLM Fine-Tuning Pipeline
**When to use**: applying GPT-style generation to a narrow technical domain (e.g., EV-charging literature synthesis).
**How**: Data Collection → Pre-processing → Model Selection → Fine-tuning → Text Generation → Quality Assessment (coherence, relevance, factual accuracy against source literature) → Output Analysis → Iterative Improvement → Documentation.
**Trade-offs**: fine-tuning improves domain fidelity but requires a mandatory factual-accuracy check step before the output can inform real decisions.
*(Ch5)*

## IDS Technique Selection (IoT)
**When to use**: choosing an intrusion-detection approach for a resource/data-constrained IoT deployment.
**How**: match technique family (ML — RF/DT/NB/SVM/etc.; DL — RNN/LSTM/GRU/CNN; Generative-AI — GAN/TL/EL for synthetic attack data; or non-ML — heuristic/flow/entropy/protocol/behavioral/fog-based) to device compute budget, available labeled data, and specific attack class.
**Trade-offs**: simple ML (NB) is cheap but can badly underperform (38-40% accuracy in-book vs 99%+ for RF/DT/DL) — algorithm choice within a category matters more than category choice.
*(Ch3)*

## Hybrid CNN+Dense Dual-Branch Classifier
**When to use**: structured/tabular classification tasks (e.g., network traffic) where both spatial-sequence patterns and simple feature relationships carry signal.
**How**: Input → parallel CNN branch (Conv1D→Pool→Conv1D→Pool→Dense→Dense→Flatten) and Dense branch (Dense→Dense→Dense→Dense) → Connector Layer merges both → final Dense → Output.
**Trade-offs**: outperformed general LLMs (GPT-3.5/GPT-4o) by ~46-52 percentage points of accuracy on IoT intrusion detection in-book — purpose-built architectures beat general-purpose LLMs on structured data tasks.
*(Ch6)*

## AI-Driven Predictive Maintenance Stack
**When to use**: any sensor-instrumented critical infrastructure (grid, industrial equipment, EV charging stations) moving from calendar-based to condition-based maintenance.
**How**: Condition Monitoring (continuous sensor streaming) → Early Detection (ML flags deviations from learned "normal") → Predictive Analytics (forecast failure/degradation timing) → Proactive Maintenance Scheduling.
**Trade-offs**: requires upfront sensor integration and historical-data investment; pays off by avoiding unplanned downtime versus reactive maintenance.
*(Ch7)*

## Constrained Energy Storage Optimization
**When to use**: scheduling charge/discharge cycles for grid-connected battery/storage systems.
**How**: maximize economic objective (sell-high/buy-low) as a function of charge/discharge power, subject to hard constraints — max capacity (`E_max`), charge efficiency (`η_charge`), discharge efficiency (`η_discharge`).
**Trade-offs**: ignoring the efficiency/capacity constraints breaks physical feasibility even if the economic objective looks optimized — constraints must be enforced, not treated as soft preferences.
*(Ch7)*

## GAN Face Reconstruction with Identity Preservation
**When to use**: restoring/inpainting occluded or damaged facial images where the output must still verify as the same identity (not just look plausible).
**How**: Generator reconstructs occluded region → discriminator(s) (often dual: global + local) enforce realism → identity-preserving loss term (e.g., pre-trained VGG features, or three-way identity competition as in FaceID-GAN) ensures output matches original identity.
**Trade-offs**: without the identity-loss term, GAN inpainting can produce a visually convincing but wrong-identity face — a critical failure for recognition/verification use cases.
*(Ch8)*

## Transfer-Learning Backbone Segmentation
**When to use**: pixel-level segmentation tasks (medical imaging, organ boundaries) with limited training data or compute budget.
**How**: replace a U-Net's encoder with a pre-trained ImageNet backbone (VGGNet, ResNet-152, or MobileNetV2) as feature extractor, keep U-Net-style decoder to reconstruct the segmentation mask.
**Trade-offs**: addresses UNet's overfitting/data-volume/compute challenges; but no single backbone wins on every sample/metric — validate multiple backbones on your own data (book found MobileNetV2 usually best, but not always).
*(Ch9)*

## ChatGPT-IoT Deployment Checklist
**When to use**: pre-launch audit for any conversational-AI-on-IoT-device project.
**How**: verify Edge Computing/Latency handling → Model Optimization (quantization/pruning/distillation for edge hardware) → Data Privacy/Security (encryption, access control) → Scalability/Load Balancing → Continuous Monitoring → Interoperability (MQTT/CoAP, documented APIs/SDKs) → User Experience/Feedback Loop.
**Trade-offs**: skipping any one dimension (e.g., deploying full-size models without optimization) causes production failure even if conversational quality is excellent in testing.
*(Ch10)*
