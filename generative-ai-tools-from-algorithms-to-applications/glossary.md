# Glossary

**ARIMA (Autoregressive Integrated Moving Average)** — time-series load-forecasting model incorporating temporal trends in power usage (Ch7).

**Attention-gated U-Net (Atten-UNet)** — U-Net variant adding an attention-gating mechanism at decoder skip connections to suppress irrelevant/redundant features (Ch9).

**BLEU (Bilingual Evaluation Understudy)** — metric comparing generated text to reference responses for quality scoring (Ch4).

**CICIDS2017 / CICIoT2022** — public IoT/network-traffic intrusion-detection benchmark datasets (Ch3, Ch6).

**Confusion matrix** — table comparing true vs. predicted classification labels; diagonal = correct, off-diagonal = misclassifications (Ch3, Ch6).

**Context window** — the amount of prior conversation/text an LLM can use as context; grew from 512 tokens (GPT-1) to 2192+ (GPT-4 era per source) (Ch4).

**DA-GAN** — GAN framework improving synthesized-face authenticity using unlabeled real images while preserving identity (Ch8).

**Decentralized Energy Management Systems (DEMS)** — blockchain-based peer-to-peer energy trading and decentralized grid decision-making (Ch7).

**Deepfake** — highly realistic synthetic media (often GAN-derived) capable of influencing consumer/public perception (Ch2).

**DeMeshNet** — GAN-based face-restoration model targeting pixel- and feature-level resemblance between original and reconstructed faces (Ch8).

**Digital Twin** — virtual replica of a physical asset/network used for simulation and predictive maintenance (Ch7).

**Dynamic Charging Management (DCM)** — real-time adjustment of EV charge rates based on network demand/grid conditions (Ch7).

**Edge AI / Edge computing** — running AI inference near the data source/device instead of a centralized cloud, reducing latency (Ch7, Ch10).

**Explainable AI (XAI)** — techniques for making AI decision-making transparent/interpretable, cited across grid fault-detection and general AI-adoption contexts (Ch7, and referenced in Ch10's context).

**FaceID-GAN** — GAN framework treating identity-preserving face synthesis as a three-way identity-consistency competition (Ch8).

**Federated learning** — collaborative model training across distributed edge devices without centralizing raw data, preserving privacy (Ch7).

**Fog computing** — extending cloud computing to the network edge so data processing happens near data sources/IoT devices (Ch3).

**GAN (Generative Adversarial Network)** — generator/discriminator architecture trained adversarially to produce realistic synthetic data (Ch1, Ch3, Ch8).

**Identity-preserving loss** — training objective term ensuring a GAN-reconstructed/synthesized face remains recognizable as the original person (Ch8).

**IDS (Intrusion Detection System)** — device/software identifying unauthorized network/system activity (Ch3, Ch6).

**JSC (Jaccard Similarity Index / IoU)** — intersection-over-union metric measuring segmentation mask overlap quality, stricter than raw accuracy (Ch9).

**MobileNetV2** — lightweight CNN architecture using depth-wise and point-wise convolutions to reduce compute cost; strong transfer-learning backbone in this book's lung-segmentation study (Ch9).

**MQTT / CoAP** — standard lightweight IoT communication protocols used for device-to-server interoperability (Ch10).

**Multimodal generation/input** — a single model producing or accepting more than one content type (text, image, video, audio) (Ch2, Ch4).

**Perplexity** — metric for how well a language model predicts a sequence of words; lower = better performance (Ch4).

**PSNR / SSIM** — image-quality metrics (Peak Signal-to-Noise Ratio / Structural Similarity Index) used to evaluate GAN/VAE image-enhancement output (Ch1).

**Random Forest** — ensemble decision-tree ML algorithm used across IDS classification (Ch3) and equipment-health prediction (Ch7).

**Residual connections (ResNet)** — shortcut connections bypassing network layers to address vanishing gradients in deep networks (Ch9).

**RLHF (Reinforcement Learning with Human Feedback)** — training approach using human feedback to align model behavior; introduced with GPT-3.5/InstructGPT (Ch4).

**Synthetic data generation** — using generative models to create realistic training/validation data where real data is scarce (Ch1, Ch3).

**Transfer Learning (TL)** — reusing a pre-trained model's weights as a feature extractor/backbone for a new task, reducing training-data/compute requirements (Ch3, Ch9).

**Transformer architecture** — neural network using self-attention to capture word relationships across a sequence; foundation of GPT models (Ch4).

**U-Net** — encoder-decoder CNN architecture with skip connections, standard for image segmentation tasks (Ch9).

**VAE (Variational Autoencoder)** — autoencoder learning a probabilistic latent space for generation/reconstruction; more stable but blurrier output than GANs (Ch1, Ch8).

**Zero-shot learning** — a model performing a task it wasn't explicitly trained for, using only pre-trained general knowledge (Ch4).
