# Chapter 3: A Comprehensive Review on Intrusion Detection Systems for IoT — From Theory to Implementation Using ML, DL, and Generative AI Techniques

*Authors: Naveen Saran, Nishtha Kesswani, and Ravi Saharan*

## Core Idea
IoT devices are structurally hard to secure (limited compute, battery, heterogeneous protocols), so effective Intrusion Detection Systems (IDS) must be chosen by matching the ML/DL/generative technique to the specific attack class, dataset, and resource constraint rather than picking one universal model.

## Frameworks Introduced
- **IDS Technique Taxonomy for IoT**: 15 named approaches — ML-based, DL-based, Generative-AI-based, fog-computing-based, heuristic, flow-based, entropy-based, distributed, behavioral, protocol, time-series, statistical, payload-inspection, self-learning, collaborative.
  - When to use: as a menu — the chapter is explicit that "individual IoT deployment, its needs, and the kinds of risks it might encounter determine which strategies are best," implying combination over single-technique dependence.
- **Four-category IoT attack classification**: physical, software, network, encryption attacks — each maps to a different IoT architecture layer (physical/network/application).
  - How: secure each layer independently — secure booting + signed firmware at the physical layer, point-to-point encryption/authentication at the network layer, access control/firewalls/antivirus at the application layer.
- **CICIDS2017 ML/DL Benchmarking Algorithm** (Algorithm 3.1 in source): Load dataset → preprocess (binary + multi-class label encoding, one-hot encoding, drop NA) → EDA (mean/std/median/mode, value counts, visualizations) → split train/test → train ML classifiers (RF, DT, NB) → train DL models (RNN, LSTM, GRU) → evaluate via Accuracy/Precision/Recall/F1 → plot loss/accuracy curves.
  - When to use: template for building and comparing an IoT IDS on any labeled benchmark dataset.

## Key Concepts
- **IDS (Intrusion Detection System)**: physical device or software that identifies unauthorized network/system activity and alerts an administrator.
- **DoS / DDoS**: attacks that deny device/service availability, the latter via a botnet of compromised IoT devices.
- **Man-in-the-Middle**: attacker intercepts and can alter traffic between an IoT device and its intended recipient.
- **Spoofing**: malware creates fake node identities to disrupt device function.
- **GAI-based IDS**: using GANs/Transfer Learning/Ensemble Learning to simulate synthetic attack traffic for training and to adapt detection to evolving traffic patterns, reducing false-positive rate.
- **Fog computing (for IDS)**: pushing detection/processing to edge "fog nodes" near data sources instead of the resource-constrained device itself, reducing IoT-side burden.

## Mental Models
- Treat "which IDS technique" as a constrained-optimization question: dataset availability × device compute budget × attack type you're defending against, not a single best-in-class answer.
- Generative AI's specific edge in IDS is *synthetic adversarial training data* — when real attack examples are scarce (as with rare/novel exploits), GAN-generated attack traffic lets you train and stress-test a detector before the real attack occurs.

## Anti-patterns
- **Trusting accuracy alone**: the chapter's own Table 3.1 shows Naive Bayes at only 38–40% accuracy/F1 on CICIDS2017 vs. 99%+ for RF/DT/RNN/LSTM/GRU on the *same* dataset — a reminder that algorithm choice, not just "ML vs. none," dominates outcome.
- **Relying on a single stale benchmark dataset**: KDD Cup99/DARPA are flagged as having accuracy and duplication problems (NSL-KDD was created specifically to fix KDD99's duplicate-record bias) — validate on a current, representative dataset (CICIDS2017, TON-IoT, BoT-IoT) rather than legacy ones.
- **Ignoring device-layer constraints when picking a technique**: deep, multilayer DL models may be infeasible on-device; fog computing or lightweight ML (DT, NB) may be the only realistic option at the edge.

## Reference Tables

**Benchmark IDS results on CICIDS2017 (chapter's own experiment, Table 3.1):**

| Model | Precision % | Recall % | F1 % | Accuracy % |
|---|---|---|---|---|
| Random Forest (RF) | 99.93 | 99.93 | 99.94 | 99.93 |
| Decision Tree (DT) | 99.94 | 99.94 | 99.95 | 99.95 |
| Naive Bayes (NB) | 38.62 | 38.62 | 38.12 | 40.00 |
| RNN | 99.85 | 99.84 | 99.84 | 99.82 |
| LSTM | 99.90 | 99.90 | 99.89 | 99.89 |
| GRU | 99.87 | 99.87 | 99.87 | 99.87 |

**Public IDS benchmark datasets covered:** DARPA/KDD Cup99 (1998/99, duplication issues), CAIDA (2007, low attack diversity), NSL-KDD (fixes KDD99 duplication), ISCX2012 (real traffic traces), CICIDS2017 (DDoS/Heartbleed/Web Attack/Infiltration/Botnet/Brute Force), UNSW-NB15 (2.5M+ records, 7 attack types), TON-IoT (multi-source IoT/IIoT telemetry), BoT-IoT (realistic botnet traffic), MQTT-IoT-IDS2020 (M2M protocol-specific).

## Worked Example
**Building and comparing an IDS on CICIDS2017 (Algorithm 3.1 walkthrough).** The chapter's own pipeline: (1) load the CICIDS2017 CSV; (2) create both binary and multi-class labels, one-hot encode categoricals, drop NA rows; (3) run EDA — per-feature mean/std/median/mode, value counts, bar/pie visualizations; (4) split into train/test; (5) train three ML models (RF, DT, NB) and evaluate with Accuracy/F1/confusion matrix; (6) separately define and train three DL architectures (RNN, LSTM, GRU) and evaluate with Accuracy/Precision/Recall/F1; (7) plot loss and accuracy curves for each. Result: tree-based ML (RF, DT) and recurrent DL (LSTM particularly) both clear 99.8%+ accuracy, while the simple probabilistic baseline (NB) badly underperforms — demonstrating that within IDS, model family choice matters far more than the ML-vs-DL distinction.

## Key Takeaways
1. No single IDS technique wins universally — match technique to device constraints, attack type, and available labeled data.
2. Generative AI's concrete IDS value-add is synthetic adversarial data generation for training/testing under realistic but controlled conditions, plus improved adaptability to shifting traffic patterns.
3. Dataset choice materially affects validity — avoid legacy datasets (KDD99) with known duplication/accuracy problems in favor of current benchmarks (CICIDS2017, TON-IoT, BoT-IoT).
4. Security must be layered by IoT architecture tier: physical (secure boot, signed firmware), network (encryption/authentication), application (access control, firewalls, integrity checks).
5. Fog computing offloads IDS processing from resource-constrained devices to edge nodes, addressing IoT's core limitation: limited compute/battery.
6. Algorithm choice dominates result quality more than technique category — this chapter's own benchmark shows Naive Bayes near-failing (40% accuracy) while other ML/DL models on the identical dataset exceed 99.8%.

## Connects To
- **Ch6**: extends this chapter's GAN-vs-DL comparison specifically for IoT intrusion detection with a hybrid deep-learning approach.
- **Ch1**: GAN fundamentals (generator/discriminator) introduced there underpin the GAI-based IDS techniques here.
- **Ch10**: shares the IoT/edge deployment context, applied there to conversational AI rather than security.
