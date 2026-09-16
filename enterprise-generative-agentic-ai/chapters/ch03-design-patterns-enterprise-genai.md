# Chapter 3: Design Patterns for Developing Enterprise GenAI Applications

## Core Idea
Enterprise-grade GenAI applications need the same discipline as traditional software: a catalog of named, reusable design patterns covering data handling, model training/deployment, testing, multi-agent orchestration, and security/ethics — applied together with (not instead of) best practices.

## Frameworks Introduced
- **Pipeline Pattern**: structures data ingestion (collection, cleaning, harmonization, transformation, feature extraction) into a sequential, reliable workflow.
  - When to use: any AI system with a nontrivial data-ingestion stage — prevents system failures at the ingestion layer.
- **Model Factory Pattern**: automates creation/deployment of multiple model versions (e.g., per-language intent classifiers) for consistency.
- **Microservice Architecture Pattern**: decomposes an AI pipeline into independently deployable/scalable services (ingestion, preprocessing, inference, feedback).
  - When to use: when different pipeline stages have very different scaling needs (e.g., updating an inference model without touching ingestion).
- **A/B Testing vs. Shadow Testing**: two model-validation patterns.
  - A/B: splits live users across model versions to compare performance (needs load-balancing/routing).
  - Shadow: runs new model silently alongside production without exposing output to users (needs dual inference pipelines) — preferred for compliance-sensitive upgrades.
- **Federated Learning Pattern**: trains a shared model across decentralized regions without moving raw data — only model updates are shared with a central/anchor region.
  - When to use: cross-region/cross-org data-privacy constraints (GDPR/CCPA) where centralizing raw data is prohibited.
- **Differential Privacy Pattern**: mathematically guarantees an individual's presence/absence in a dataset can't be inferred from published statistics, by adding calibrated noise. Only protects information framed as *private*, not general facts.
- **Blue-Green vs. Canary Deployment**: Blue-Green runs new (blue) and old (green) models in parallel for zero-downtime rollback; Canary gradually releases the new model to a user subset to de-risk full rollout.
- **Orchestrator Pattern**: manages interaction among multiple models/agents (multi-agent, ensemble, hybrid systems).
- **Composite Pattern**: combines predictions from multiple models into one cohesive output (ensembling).
- **Model Compression Pattern**: pruning/quantization/knowledge distillation to shrink model size without significant accuracy loss.
- **Zero Trust Pattern**: default-deny access control, permissions granted strictly on a need basis — supports HIPAA/ISO 27001/PCI-DSS compliance.
- **Prompt-engineering pattern reuse**: Strategy Pattern (interchangeable prompt strategies selected dynamically), Template Pattern (skeleton with placeholders), Builder Pattern (assemble complex prompts from parts step by step).
- **Agentic-AI-supporting patterns**: Observer (agent reacts to environment state changes), State (behavior changes with internal state), Chain of Responsibility (pipeline of handlers each making a decision).

## Key Concepts
- **Design pattern**: a reusable, named solution to a recurring software-design problem, providing shared vocabulary across teams/organizations.
- **Model drift / data drift**: gradual decrease in model effectiveness over time as the world changes — addressed by the Monitoring & Feedback Loop Pattern, which triggers retraining.
- **Bias Detection & Mitigation**: identifies/reduces bias in training data and outputs.
- **Explainability and Auditability**: makes AI decisions interpretable to end users and regulators (supports EU AI Act "right to explanation").
- **Content Moderation Pattern**: filters generated output for harmful/inappropriate content.

## Mental Models
- **Scalability and security are interdependent, not competing**: a system that scales but leaks data is a liability; a secure system that can't scale fails under real demand — design for both simultaneously.
- **Decision tree for prompt-engineering pattern choice**: multiple interchangeable strategies → Strategy Pattern; same structure with minor tweaks → Template Pattern; multi-section composed prompts → Builder Pattern.
- **"Design patterns + best practices + architectural principles" as three distinct layers**: don't conflate a named pattern (Observer) with a best practice (data quality) or an architectural principle (multi-agent orchestration) — they compose together but are not interchangeable.

## Anti-patterns
- **Treating design patterns as sufficient for ethical AI on their own**: the book explicitly warns patterns must be combined with ethical guidelines, stakeholder engagement, diverse teams, and ongoing monitoring — patterns alone don't guarantee fairness.
- **Monolithic AI systems**: harder to scale/maintain than microservice-decomposed pipelines; specific components can't be scaled or updated independently.
- **Static rule-based prompt handling** in place of the Strategy/Template/Builder patterns — leads to brittle, unmaintainable prompt logic as use cases multiply.

## Reference Tables
**A/B Testing vs. Shadow Testing**
| Feature | A/B Testing | Shadow Testing |
|---|---|---|
| Purpose | Compare 2+ model versions live | Test new model silently alongside current |
| User exposure | Split across versions (e.g., 50/50) | Only current model's output shown |
| Deployment | Needs load balancing/routing | Needs dual inference pipelines |
| Common usage | UI/content-ranking models | LLM upgrades, compliance-sensitive systems |

**Prompt pattern selection checklist**
| Question | Preferable pattern |
|---|---|
| Multiple interchangeable prompt strategies? | Strategy Pattern |
| Switch prompt behavior by user intent/context? | Strategy Pattern |
| Same structure, minor tweaks, repeatedly? | Template Pattern |
| Need placeholders for dynamic content? | Template Pattern |
| Prompt composed of multiple sections? | Builder Pattern |
| Need step-by-step modular construction? | Builder Pattern |

**Security/privacy pattern → compliance standard**
| Pattern | Objective | Standard |
|---|---|---|
| Federated learning | Keeps data local | GDPR, CCPA |
| Differential privacy | Anonymizes via statistical noise | GDPR, FERPA, OECD |
| Zero trust architecture | Least-privilege, default-deny | HIPAA, ISO 27001, PCI-DSS |
| Encryption at rest/transit | Protects stored/moving data | HIPAA, ISO 27001, PCI-DSS |
| Content moderation | Blocks harmful output | Platform policy, EU DSA |
| Explainability/auditability | Transparency, traceability | EU AI Act, GDPR |
| Bias detection/mitigation | Fairness | EEOC, GDPR, FTC AI guidance |

## Worked Example
**Telecom customer-service chatbot pipeline**, combining two scalability patterns:
- *Model Factory Pattern*: trains multiple per-language/region intent-recognition models, containerizes and auto-deploys them via CI/CD.
- *Microservice Architecture*: splits the pipeline into a data-ingestion microservice (collects conversation logs), a preprocessing microservice (cleans/tokenizes input), a model-inference microservice (runs the latest deployed classifier), and a feedback-loop microservice (captures ratings for retraining) — each independently scalable and updatable (e.g., ship a new inference model without touching ingestion).

## Key Takeaways
1. Map any GenAI system's design problem to the life-cycle stage (data/train/deploy/monitor/multi-agent/optimize/security/test) and pick the named pattern for that stage — don't invent bespoke solutions.
2. Prefer Shadow Testing over A/B Testing for compliance-sensitive model upgrades; use A/B for UX-facing changes where live comparison is acceptable.
3. Federated Learning + Differential Privacy + Zero Trust together form the standard enterprise privacy/security stack for regulated data (GDPR/HIPAA/PCI-DSS).
4. Agentic AI patterns (Observer, State, Chain of Responsibility) are ordinary software patterns repurposed for autonomous decision loops — no fundamentally new pattern vocabulary is required.
5. Patterns are necessary but not sufficient for ethical AI — pair them with governance processes.

## Connects To
- **Ch 4**: directly extends the Agentic AI Pattern preview here into a full chapter on agentic workflows, registries, and implementation.
- **Ch 6**: the Monitoring & Feedback Loop Pattern and A/B/Shadow testing here are the mechanisms that feed the evaluation framework covered later.
- **Ch 7**: the security/compliance patterns (Zero Trust, Differential Privacy) reappear as concrete implementations of the Responsible AI/Risk framework.
