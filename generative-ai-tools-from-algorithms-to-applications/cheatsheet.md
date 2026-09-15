# Cheatsheet

## Decision rules

- **Choosing GAN vs. VAE**: need sharp/realistic output (images, face reconstruction) → GAN, accept unstable training. Need stable, structured latent-space generation/reconstruction (augmentation, compression) → VAE, accept blurrier output. Need both → Adversarial Autoencoder (AAE) hybrid. *(Ch1, Ch8)*
- **Choosing an IDS technique for IoT**: resource-constrained device + known attack signatures → lightweight ML (DT/RF) or heuristic/flow-based. Rich labeled data + compute available → DL (LSTM/GRU/RNN). Attack examples scarce → GAN-based synthetic data generation to fill the gap. *(Ch3)*
- **LLM vs. purpose-built model for structured/numeric tasks**: if the task is structured/tabular classification (not natural language), default to a purpose-built architecture (CNN/Dense hybrid, Random Forest) — do not assume a state-of-the-art LLM transfers. In-book evidence: hybrid model 99.5% vs. GPT-4o 47% / GPT-3.5 53% on the same IoT-intrusion task. *(Ch6)*
- **Rule-based vs. retrieval vs. generative chatbot**: narrow, must-be-predictable domain → rule-based. Common queries against a fixed answer set → retrieval-based. Open-ended, natural conversation → generative/Transformer-based, accept less predictability. *(Ch4)*
- **Transfer-learning backbone for segmentation**: default to MobileNetV2 for best accuracy/compute balance (book's most consistent winner), but validate ResNet-152 and VGGNet too — no backbone won every sample in-book. *(Ch9)*
- **Deploying an LLM at the IoT edge**: full model too large/slow for edge hardware → apply quantization, pruning, or distillation before deployment; do not deploy unmodified. *(Ch10)*

## Thresholds & defaults (as reported in the source studies — dataset/task-specific, not universal constants)

| Metric | Value | Context |
|---|---|---|
| Hybrid CNN+Dense IDS accuracy | ~99.5% | CICIoT2022, vs. GPT-4o 47% / GPT-3.5 53% (Ch6) |
| Naive Bayes IDS accuracy | 38-40% | CICIDS2017 — cautionary example, not a target (Ch3) |
| RF/DT IDS accuracy | ~99.9% | CICIDS2017 (Ch3) |
| DL training hyperparameters (lung segmentation) | Adam, lr=0.001, batch=8, 100 epochs, BCE loss | JSRT dataset (Ch9) |
| GPT-1 → GPT-3 parameter growth | 117M → 1.5B → 175B | Illustrates scale trajectory, not a target to hit (Ch4) |
| AR dataset real-occlusion accuracy range | 60-100% | Wide variance by method; sunglasses vs. scarf differ (Ch8) |

## Tells & smells

- **A reported accuracy number with no per-class or per-metric breakdown** → suspect a hidden failure mode (e.g., GPT-3.5's 53% aggregate hid frequent BruteForce→Normal misclassification — a dangerous false negative for security). Always ask for the confusion matrix or per-class precision/recall. *(Ch3, Ch6)*
- **"Highly accurate" claims for image segmentation with accuracy but no JSC/IoU reported** → likely inflated by easy background pixels; ask for Jaccard/IoU specifically. *(Ch9)*
- **GAN-based face restoration output that looks realistic but recognition confidence drops** → missing identity-preserving loss term; check whether the training objective included an identity/perception loss. *(Ch8)*
- **A generative-AI system confidently producing domain-specific facts (medical, legal, financial) without a stated verification step** → treat as unverified; the book's own EV-planning pipeline treats "compare against existing literature" as a mandatory pipeline stage, not optional. *(Ch5)*
- **An LLM being proposed for a task that's fundamentally structured/tabular data (not natural language)** → red flag; benchmark against a purpose-built model first. *(Ch6)*
- **A chatbot-IoT deployment plan that only discusses conversational quality** → missing dimensions; check it covers all seven of the deployment checklist (latency, efficiency, privacy, scalability, monitoring, interoperability, UX). *(Ch10)*

## Trade-off matrix: GAN vs. VAE vs. Hybrid (AAE)

| Dimension | GAN | VAE | AAE (hybrid) |
|---|---|---|---|
| Output sharpness | High | Low (blurry) | Medium-high |
| Training stability | Low (adversarial, unstable) | High | Medium |
| Latent space structure | Implicit | Explicit/probabilistic | Explicit + adversarial refinement |
| Best for | Realistic image generation, face reconstruction | Reconstruction, augmentation, stable sampling | When both realism and structured latent space matter |
