# Glossary — Building AI-Powered Products

**0-to-1 AI product** — Applying an emerging AI technology to a brand-new product/experience; high uncertainty, unproven market fit. (Ch 1, 2)

**1-to-n AI product** — Enhancing/scaling an already-successful product with AI; clearer market fit, focus on UX integration. (Ch 1, 2)

**Accuracy** — Percentage of correct classifications made by a model out of all classifications. (Ch 6)

**Agent (AI agent)** — A system that acts autonomously in an environment, learns from experience, and acts proactively — distinct from a chatbot, which responds only to explicit input via scripted rules. (Ch 8)

**AI builder PM** — AI PM category focused on foundational AI technologies/models, working closely with researchers and data scientists (e.g. infrastructure PM, security PM). (Ch 1)

**AI-enhanced PM** — AI PM category that uses AI tools to improve their own PM workflow, regardless of whether the underlying product is AI-centric. (Ch 1, 7)

**AI experiences PM** — AI PM category focused on user-facing AI features layered onto products (e.g. ranking PM, recommendations PM, responsible AI PM). (Ch 1)

**AI investment (RICE extension)** — Additional RICE factor representing the complexity of training/integrating a model (data, compute, cost); transforms RICE into R × I × C / (E × A). (Ch 2)

**AI Lifecycle (five technical stages)** — Project scoping → Data collection → Model training → Validation & testing → Deployment; nested inside the AIPDL's Concept/Prototype stage. (Ch 3)

**AI MVP** — A functional AI product designed to add real value from day one using live data and real integrations, distinct from a prototype that uses mock data. (Ch 2)

**AI PM Career Ladder** — Illustrative levels from execution (4-6) through AI/ML PM (5-7) to strategic leadership (8+, 9+). (Ch 4)

**AI Product Development Lifecycle (AIPDL)** — The book's master framework: Ideation → Opportunity → Concept/Prototype → Testing & Analysis → Rollout, iterative and cyclical. (Ch 2)

**AI product management** — The craft of building products that are themselves powered by AI technologies (distinct from "AI for product managers"). (Ch 1, 7)

**AI proxy metrics** — Metrics measuring underlying model performance (accuracy, precision, recall, loss functions) as a proxy for, not identical to, the ultimate product goal. (Ch 6)

**Algorithm** — A set of rules defining how to perform a task/make decisions (e.g. decision trees, regression); distinct from a model, which is a trained instance of an algorithm. (Ch 3)

**AGI (Artificial General Intelligence)** — Speculative future AI type capable of cross-domain reasoning and problem-solving comparable to humans. (Ch 1)

**ASI (Artificial Superintelligence)** — Hypothetical future AI type surpassing human intelligence across all domains. (Ch 1)

**Autonomy (agent design axis)** — The degree to which an agent acts independently on the user's behalf vs. only suggesting actions; designed as a progressive, explicitly scoped dial. (Ch 8)

**Black-box model** — A complex model (e.g. deep neural network) whose decision-making process is opaque to humans, raising interpretability/trust challenges. (Ch 1, 3)

**Build-vs-Buy Decision Matrix** — Seven-factor table (core competency, resources/expertise, time to market, long-term strategy, cost, risk, data privacy, competitive landscape) for deciding whether to build AI in-house or license it. (Ch 5)

**Churn** — The inverse of retention; the rate at which users stop using the product. (Ch 6)

**Confusion matrix** — A 2×2 table (true positive, true negative, false positive, false negative) used to evaluate a classification model's performance. (Ch 6)

**Consumer-facing agent** — An agent that interacts directly with users (e.g. Siri, Alexa), as opposed to a behind-the-scenes agent. (Ch 8)

**Core product management craft** — One of the four AI PM skill buckets: user segmentation, user stories, vision, prioritization — the foundation shared with traditional PM. (Ch 3)

**CustomGPT / custom Gem** — A tailored foundation-model configuration (prompts + tool integrations + workflows, no retraining) that can cross from chatbot into agent territory once it acts with reduced explicit prompting. (Ch 8)

**Data preprocessing** — Cleaning, labeling, and structuring raw collected data into a format suitable for model training; a continuous, evolving process. (Ch 3)

**DataOps** — Team/discipline responsible for collecting, cleaning, and ensuring compliant, high-quality data feeding AI models — distinct from MLOps. (Ch 3, 4)

**Disruptive innovation** — An innovation that initially appears inferior/niche but targets an unmet future need and eventually redefines the market (Christensen's Innovator's Dilemma). (Ch 5)

**Engineering foundations (for PMs)** — AI PM skill bucket covering version control, build processes, testing, resource management, APIs, algorithms, system architecture — enough fluency to communicate with engineers, not to code. (Ch 3)

**Explainable AI (XAI)** — Practices/tools (SHAP, LIME, InterpretML, feature importance, counterfactual explanations) that make AI decisions understandable to users, regulators, and the engineers debugging the model. (Ch 2, 3)

**FATE framework** — Fairness, Accountability, Transparency, Ethics; Nika's go-to lens for Responsible AI practices. (Ch 3)

**Fine-tuning** — Retraining a pretrained model further on a specific labeled dataset for a well-defined, precision-critical task; resource intensive. (Ch 5)

**General-purpose agent** — An agent with an internal world model that adapts across multiple domains/tasks (e.g. LangChain-style orchestration), as opposed to a task-specific agent. (Ch 8)

**Generative AI (GenAI)** — Subset of AI producing new content (text/image/video/audio); does not replace traditional AI, adds a new capability layer. (Ch 1)

**Goal-based agent** — Task-specific agent that selects actions to accomplish a specific stated goal. (Ch 8)

**Go/No-Go Decision** — The explicit checkpoint closing the AIPDL's Testing & Analysis stage; a "No-Go" loops back to Opportunity or Concept, not abandonment. (Ch 2)

**Grounding** — Lightweight prompt-engineering approach to steer a base model's behavior without retraining; low latency, minimal new data. (Ch 5)

**Guardrail metric** — An OKR component capping an acceptable side effect of optimizing the North Star metric (e.g. "listening time must not drop more than 5%"). (Ch 6)

**Human-in-the-loop** — Design pattern where AI recommends and a human makes the final decision; used to manage automated-decision risk, threaded through the entire AI lifecycle. (Ch 1, 3)

**Innovator's Dilemma** — Clayton Christensen's framework explaining why successful companies fail against disruptive technologies; used to classify AI bets as sustaining or disruptive. (Ch 5)

**Loss function / MSE (Mean Squared Error)** — Training-time proxy metric measuring average squared difference between predicted and actual values; minimized during model training. (Ch 6)

**Minimum Viable Quality (MVQ)** — The quality threshold, set by the PM per use case, at which the AI lifecycle stops iterating and the product ships. (Ch 3)

**MLOps** — Team/discipline responsible for deploying and continuously monitoring ML models in production at scale. (Ch 3, 4)

**Model** — A specific trained instance of an algorithm on a specific dataset; the same algorithm can produce different models on different data. (Ch 3)

**Model drift** — The way AI models change behavior over time as they learn/retrain, unlike static software that only changes on manual release. (Ch 1)

**North Star metric** — The single primary KPI capturing the core value an AI product delivers; every OKR gets exactly one. (Ch 6)

**Objective function** — A proxy metric (e.g. loss function) evaluating model performance during training, guiding the learning process. (Ch 6)

**OKR (Objectives and Key Results)** — The goal-setting structure combining an ambitious objective with a North Star metric plus supporting KPIs from product health, system health, and AI proxy metric buckets. (Ch 4, 6)

**Precision** — Ratio of true positive results to all positive predictions made by the model; high precision = few false positives. (Ch 6)

**Probabilistic nature (of AI)** — AI predicts with a confidence level, never certainty — unlike deterministic software. (Ch 1)

**Product health metrics** — Engagement, satisfaction, adoption, conversion, retention, financial metrics; the AI PM's direct responsibility within the metric blend. (Ch 6)

**Product-market fit (three-pillar test)** — Business viability + technical feasibility + user desirability; all three must be satisfied — a strict AND, not an average. (Ch 2)

**Proxy metric** — Any metric approximating the true product goal but not identical to it (e.g. model accuracy vs. user satisfaction). (Ch 6)

**RAG (Retrieval-Augmented Generation)** — Enhances a generative model by retrieving from a live corpus without retraining; best for fast-changing information. (Ch 5)

**Recall / Sensitivity** — Model's ability to correctly identify all actual positive cases; high recall = few false negatives. (Ch 6)

**Reflex agent (simple reflex agent)** — The simplest agent type, reacting to stimuli via fixed if-then rules with no memory or learning. (Ch 8)

**Reinforcement learning (RL)** — Learning method where an agent learns via reward/penalty from interacting with its environment; strong for control/optimization tasks. (Ch 3, 8)

**Responsible AI practices** — Embedding fairness/accountability/transparency/ethics checks throughout the AI lifecycle, not just pre-launch. (Ch 3)

**RICE framework** — Reach × Impact × Confidence / Effort; feature prioritization framework, with an AI-specific "AI Investment" extension. (Ch 2)

**Self-supervised learning** — ML learning method where the model generates its own labels from data (powers LLMs/transformers); useful when labeled data is scarce. (Ch 3)

**Supervised learning** — ML learning method trained on labeled data; used for classification and regression tasks. (Ch 3)

**Sustaining innovation** — An innovation that incrementally improves an existing product to meet current customers' current needs. (Ch 5)

**Synthetic data** — Artificially generated data simulating real-world scenarios; useful for sensitive domains, rare scenarios, early development, or when real data is prohibitively expensive — but not a substitute for real user behavior data. (Ch 5)

**System health metrics** — Uptime, latency, scalability, error rate; technical performance metrics the AI PM must track though rarely owns directly. (Ch 6)

**Task-specific agent** — An agent designed for a specialized function within a defined scope (e.g. email filtering), as opposed to a general-purpose agent. (Ch 8)

**Trade space** — A visualized, multidimensional model of competing product/technical trade-offs (cost, time, risk, etc.), built via a six-step process rather than treated as a binary choice. (Ch 3)

**Unsupervised learning** — ML learning method trained on unlabeled data, used for clustering, association, and dimensionality reduction. (Ch 3)

**Utility-based agent** — Task-specific agent that selects actions to maximize a defined utility function (e.g. minimize energy consumption). (Ch 8)
