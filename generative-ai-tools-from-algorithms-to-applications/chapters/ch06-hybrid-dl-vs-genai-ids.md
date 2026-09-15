# Chapter 6: A Hybrid Deep Learning-Based vs Generative AI for Intrusion Detection System in IoT

*Authors: Monika Vishwakarma and Suparn Padma Patra*

## Core Idea
A purpose-built hybrid CNN+Dense (+LSTM) deep-learning model dramatically outperforms general-purpose LLMs (GPT-3.5, GPT-4o) at IoT intrusion detection — ~99.5% vs. 47-53% accuracy — demonstrating that state-of-the-art conversational LLMs are not automatically the right tool for structured, non-text classification tasks.

## Frameworks Introduced
- **Dual-Branch Hybrid CNN+Dense Architecture**: an Input Layer feeds two parallel branches — a CNN branch (Conv1D-32 → MaxPool1D → Conv1D-64 → MaxPool1D → Dense-128 → Dense-64 → Flatten) for spatial feature extraction, and a Dense branch (Dense-32 → Dense-64 → Dense-128 → Dense-64) for direct feature learning — merged by a Connector Layer into a final Dense-32 → Output Layer.
  - When to use: tasks where both spatial/sequential patterns (CNN's strength) and simple feature relationships (Dense's strength) both carry signal — e.g., network traffic classification.
  - How: train branches jointly; the connector layer concatenates the CNN's flattened output with the dense branch's final representation before the shared output head.
- **LLM-as-Classifier Evaluation Workflow**: client script queries an LLM API (GPT-3.5-turbo, GPT-4o) with the classification task → compare LLM-predicted labels against ground truth → compute accuracy/precision/recall/F1.
  - When to use: benchmarking whether a general-purpose LLM can substitute for a purpose-built classifier — the chapter's own result argues it generally cannot, for structured numeric/traffic data.

## Key Concepts
- **CICIoT2022 dataset**: network traffic from 60 real IoT devices (sensors, cameras, smart locks, plugs) across power-on, idle, and active-use states — used as the benchmark here.
- **Confusion matrix (multi-class)**: diagonal = correct classifications, off-diagonal = misclassifications; used to diagnose *which* attack classes (BruteForce, HTTPFlood, TCPFlood, UDPFlood, Normal) a model confuses.
- **GPT-4o**: OpenAI's GPT-4 variant specialized for complex queries and multi-turn reasoning — chapter notes it still underperforms on non-text, structured intrusion-detection data despite general capability gains.
- **Overfitting signal**: chapter flags a spike in validation loss during the hybrid model's 50-epoch training as a possible overfitting sign, even though overall accuracy/loss curves looked healthy.

## Mental Models
- **Capability ≠ transferability**: an LLM being state-of-the-art at text tasks (reasoning, multi-turn dialogue) says nothing about its fitness for structured/numeric classification — always benchmark against a purpose-built model before assuming an LLM generalizes to your task.
- Read a confusion matrix per-class, not just as an aggregate accuracy score: GPT-3.5's aggregate 53% accuracy hid a specific, actionable failure — BruteForce was frequently misclassified as Normal traffic, a dangerous false-negative pattern for a security tool.

## Anti-patterns
- **Defaulting to "use an LLM" for any AI task because it's trendy/general-purpose**: this chapter is a direct, quantified counter-example — hybrid CNN+Dense (99.5%) beat GPT-4o (47%) and GPT-3.5 (53%) by a wide margin on the *same* intrusion-detection task.
- **Reporting only aggregate accuracy for a security classifier**: precision/recall per attack class matters more — a model that's "53% accurate" overall but frequently mislabels BruteForce as Normal traffic is dangerously worse than the headline number suggests.

## Reference Tables

**Hybrid model vs. LLMs on CICIoT2022 (chapter's own results):**

| Model | Accuracy | Precision | Recall |
|---|---|---|---|
| Proposed Hybrid (CNN+Dense+LSTM) | ~99.50% | ~1.00 (most classes) | ~1.00 (most classes) |
| GPT-4o | 47% | 74% | 47% |
| GPT-3.5-turbo | 53% | 63% | 53% |

**Architecture branch comparison:**

| Branch | Layers | Role |
|---|---|---|
| CNN | Conv1D(32)→MaxPool→Conv1D(64)→MaxPool→Dense(128)→Dense(64)→Flatten | Spatial feature extraction |
| Dense | Dense(32)→Dense(64)→Dense(128)→Dense(64) | Direct feature learning |
| Merge | Connector Layer → Dense(32) → Output | Combines both representations for final prediction |

## Worked Example
**Benchmarking GPT-4o and GPT-3.5 as IoT intrusion classifiers.** The authors query the OpenAI API with network-traffic records from CICIoT2022 via a Python client script, asking each LLM to classify traffic as Normal or one of four attack types (BruteForce, HTTPFlood, TCPFlood, UDPFlood), then score predictions against ground-truth labels on 5,000 test instances. Result: GPT-4o classifies HTTPFlood and UDPFlood reasonably well (strong diagonal values in its confusion matrix) but struggles broadly (47% overall accuracy); GPT-3.5 does reasonably on HTTPFlood/Normal but frequently confuses BruteForce with Normal traffic, dragging its BruteForce precision down sharply. Meanwhile the purpose-built hybrid CNN+Dense model, trained directly on the same traffic features, reaches near-100% accuracy with high precision/recall across almost all classes. The lesson the authors draw: general LLMs are "usually strong for text-based tasks" but show clear "difficulties with this particular network traffic data" — a non-text, structured classification problem where a dedicated architecture wins decisively.

## Key Takeaways
1. Purpose-built hybrid CNN+Dense architectures substantially outperform general LLMs (GPT-3.5/GPT-4o) on structured IoT traffic classification — 99.5% vs. 47-53% accuracy in this study.
2. LLMs' text-domain strengths (reasoning, multi-turn coherence) do not transfer automatically to structured/numeric classification tasks like intrusion detection.
3. Always inspect per-class confusion-matrix behavior, not just aggregate accuracy — GPT-3.5's BruteForce-as-Normal confusion is a specific, security-relevant failure mode hidden by the headline number.
4. A validation-loss spike during otherwise-healthy training is worth flagging as a possible overfitting signal, even when final accuracy looks strong.
5. The dual-branch CNN+Dense merge pattern (spatial branch + direct-feature branch, concatenated before output) is a reusable architecture for tasks with mixed feature types.

## Connects To
- **Ch3**: shares the same IDS-for-IoT problem domain and CICIDS-style benchmark evaluation approach, extending it with a head-to-head LLM comparison.
- **Ch4**: provides deeper background on the GPT-3.5/GPT-4/GPT-4o lineage referenced here as the LLM baselines.
