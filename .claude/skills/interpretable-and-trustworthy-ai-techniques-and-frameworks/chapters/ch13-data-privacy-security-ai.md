# Chapter 13: Data Privacy and Security in Artificial Intelligence — Tools, Challenges, and Innovations

## Core Idea
Privacy-preserving AI is built from five composable cryptographic/statistical primitives (Differential Privacy, Federated Learning, Homomorphic Encryption, SMPC, anonymization) that each protect a different point in the data lifecycle — collection, training, computation, aggregation — and are routinely combined (e.g., DP + Federated Learning) rather than used in isolation.

## Frameworks Introduced
- **CIA Triad for AI data privacy**: Confidentiality (encryption, DP, HE prevent unauthorized access) + Integrity (validation protocols, checksums, adversarial-attack resistance keep data accurate/unaltered) + Availability (redundancy, backups, failover keep systems accessible to authorized users).
  - When to use: as the three-dimension checklist for any AI data-security review — a system can fail on any one dimension independently (e.g., perfectly confidential but low-integrity data still produces unreliable predictions).
- **Privacy-preserving technique selection by protection point**:
  - When to use DP: protecting aggregate query/model outputs from revealing individual data points (census data, recommendation systems).
  - When to use Federated Learning: training across distributed devices/institutions without centralizing raw data (mobile keyboards, multi-hospital studies).
  - When to use Homomorphic Encryption: computation must happen on data that stays encrypted throughout (cloud-based ML on untrusted infrastructure).
  - When to use SMPC: multiple parties need a joint computation without revealing their individual inputs to each other (cross-institution fraud detection).
  - When to use anonymization (k-anonymity/l-diversity/tokenization): publishing or sharing a dataset itself, not just model outputs.
- **Adversarial defense stack**: Adversarial Training (expose model to perturbed examples) + Defensive Distillation (soften decision boundaries via probability outputs) + Gradient Masking (obscure gradients from attackers — partial protection only) + Randomization (unpredictability in inputs/processing) + Certified Defenses (mathematical robustness guarantees, e.g. randomized smoothing) — same five-layer pattern as Ch3's robustness framework, applied here specifically to data-security threats.

## Key Concepts
- **Epsilon (ε) in Differential Privacy**: the privacy budget/parameter controlling noise magnitude — lower epsilon = stronger privacy but reduced accuracy; calibrating epsilon is the central DP engineering trade-off.
- **Membership inference attack**: adversary uses model outputs to determine whether a specific individual's data was used in training — a privacy risk distinct from data breaches, since it doesn't require accessing the training set directly.
- **k-Anonymity / l-Diversity**: k-anonymity groups records so each individual is indistinguishable from at least k−1 others; l-diversity extends this by requiring sensitive attributes within each group to be diverse, closing a re-identification gap k-anonymity alone leaves open.
- **Zero-Knowledge Proof (ZKP)**: lets one party prove a statement is true without revealing any underlying data — useful for verifying model computations or predictions during audits without exposing sensitive inputs.
- **Data provenance**: tracking a dataset's origin, modifications, and movement through an AI pipeline (via versioning tools, blockchain, digital signatures/hashing, provenance metadata) — distinct from data integrity, which is about the data being unaltered; provenance is about being able to *prove* that.

## Mental Models
- Treat privacy techniques as protecting different *stages* of the AI data lifecycle, not as interchangeable options: DP protects outputs, Federated Learning protects the training process's data locality, HE protects data *during* computation, SMPC protects data shared *across* parties, anonymization protects a *published dataset*. Picking the wrong one for your actual exposure point (e.g., DP when your risk is cross-party data sharing, where SMPC is the fit) leaves the real gap unprotected.
- Every privacy/security technique in this chapter trades off against utility or compute cost — DP trades privacy for accuracy (via epsilon), HE trades privacy for computational cost, SMPC trades privacy for communication overhead. Budget for the trade-off explicitly rather than treating any technique as a free upgrade.

## Anti-patterns
- **Treating "anonymized" data as permanently safe**: the chapter cites documented research showing partially anonymized/de-identified data can be re-identified by combining it with external datasets (linkage attacks) — anonymization alone is not a guarantee, especially against a motivated adversary with auxiliary data.
- **Using gradient masking as a complete adversarial defense**: the chapter flags this explicitly — gradient masking can produce "obfuscation rather than true robustness if not implemented carefully," and sophisticated attacks can bypass it; pair with adversarial training or certified defenses, don't rely on masking alone.
- **Deploying homomorphic encryption without accounting for its computational cost**: FHE is powerful but resource-intensive; using it where a lighter technique (DP or SMPC) would suffice wastes compute without a corresponding privacy gain for that use case.

## Reference Tables
| Technique | Protects | Computational Cost | Best For |
|---|---|---|---|
| Differential Privacy | Aggregate outputs | Low-moderate | Census/statistical releases, recommendation systems |
| Federated Learning | Training-time data locality | Moderate (comms overhead) | Mobile/IoT, multi-institution training |
| Homomorphic Encryption | Data during computation | High | Cloud ML on untrusted infra |
| SMPC | Cross-party joint computation | High (comms overhead) | Multi-institution fraud detection, collaborative research |
| k-Anonymity / l-Diversity | Published datasets | Low | Medical research data release, demographic analysis |

## Key Takeaways
1. Match the privacy technique to the actual exposure point in your pipeline (output, training, computation, cross-party sharing, or publication) — they are not interchangeable.
2. Calibrate DP's epsilon deliberately — it is the explicit privacy/accuracy trade-off knob, not a fixed setting.
3. Never treat anonymization as re-identification-proof; combine with DP or access controls when the re-identification risk from linkage attacks is real.
4. Layer adversarial defenses (training + distillation + randomization + certified methods) — no single defense, especially gradient masking alone, is sufficient.
5. Maintain data provenance (versioning, blockchain, hashing, metadata) separately from data integrity checks — provenance is what lets you *prove* integrity to an auditor or regulator.
6. Combine techniques routinely: DP + Federated Learning is a standard pairing (noise added to shared model updates, not raw data) that neither achieves alone.

## Connects To
- **Ch3**: this chapter's adversarial-defense stack mirrors and extends Ch3's general robustness framework, applied specifically to data/privacy threats.
- **Ch12**: AI audit frameworks' "data integrity" pillar depends directly on this chapter's provenance and integrity-verification techniques.
