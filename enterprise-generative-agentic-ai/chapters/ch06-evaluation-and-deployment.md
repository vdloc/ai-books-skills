# Chapter 6: Evaluation and Deployment

## Core Idea
GenAI evaluation requires a four-pillar framework (Quality/Relevance, Safety/Robustness, Efficiency/Scalability, Ethics/Alignment) beyond traditional accuracy metrics, because outputs are nondeterministic and open-ended; deployment then requires choosing among cloud/on-prem/hybrid/edge strategies and six topologies based on latency, compliance, and scale needs.

## Frameworks Introduced
- **Four Pillars of GenAI Evaluation**: Quality & Relevance (factual accuracy, fluency, coherence — BLEU/ROUGE/Perplexity/FID); Safety & Robustness (harmful/biased output detection, adversarial/jailbreak resistance); Efficiency & Scalability (latency, throughput, memory, compression); Alignment with Human Values & Ethics (fairness, transparency, accountability, bias audits).
  - When to use: as the top-level checklist before shipping any GenAI feature — a system that scores well on Quality but ignores Safety or Ethics is not evaluation-complete.
- **HELM (Holistic Evaluation of Language Models, Stanford CRFM)**: multimetric benchmarking framework measuring accuracy, calibration, robustness, fairness, bias, toxicity, and efficiency under standardized conditions, with full transparency of scenarios/prompts/predictions/code, continuously updated.
  - When to use: comparing foundation models on a level playing field rather than trusting vendor-reported benchmarks alone.
- **Functional vs. Nonfunctional Evaluation Parameters**: functional = accuracy/relevance/creativity/task-success-rate (core task quality); nonfunctional = Performance (latency/throughput/scalability), Reliability (availability/error-rate/fault-tolerance), Security (data privacy/adversarial robustness/prompt-injection protection), Usability (ease of use/satisfaction/accessibility), Cost (infra/API fees/maintenance).
  - How: each nonfunctional dimension needs a Definition + Benchmark + Metric-capture method + Threshold that triggers action (the book gives concrete numeric thresholds for every one — see Reference Tables).
- **MLOps → LLMOps lifecycle**: Planning/conceptualization → Data management (incl. RAG vector-DB setup) → Model selection & customization (foundation model choice, fine-tuning, prompt engineering, few-shot/chain-of-thought) → Evaluation & validation (quantitative: accuracy/F1/BLEU/ROUGE/perplexity; qualitative: expert review, user feedback) → Deployment strategies (CI/CD, versioning, hosting architecture) → Operation/Monitoring/Maintenance (OMM).
- **Deployment Strategy Selection (Cloud / On-Premises / Hybrid / Edge)**: choose based on data sensitivity, latency requirements, cost, regulatory constraints, and scalability needs — not a single default.
- **Six Deployment Topologies**: Single-node (standalone, PoC-only), Multinode/clustered (scalable, fault-tolerant, complex to manage), Microservices (independent scaling, fault isolation, higher comms overhead), Serverless (zero infra management, pay-per-use, cold-start latency risk), Distributed (multi-region, low latency, hard to synchronize), Cloud-edge hybrid (low-latency edge + heavy-duty cloud, complex orchestration).

## Key Concepts
- **Nondeterministic evaluation problem**: the same GenAI input can produce different valid outputs across runs, so evaluation must consider a *spectrum* of acceptable responses rather than a single ground truth — requires blending statistical, qualitative, and quantitative methods.
- **Reference-based vs. reference-free text metrics**: BLEU/ROUGE/METEOR need human-authored references and penalize correct-but-differently-worded output; reference-free metrics (TIGERScore, PERSE, Pomme, DiscoScore, CTC-Score) evaluate quality/factual-consistency/coherence without needing a reference — critical when generating high-diversity or creative content.
- **Embedding-based text metrics** (BERTScore, MoverScore, BLEURT): compare generated vs. reference text in semantic embedding space, capturing meaning beyond surface n-gram overlap.
- **pass@k**: generates k code samples and checks if any pass unit tests — the standard functional-correctness metric for code generation, superior to exact-match or BLEU-for-code.
- **RAI (Responsible AI) evaluation KPIs**: statistical parity, equal opportunity, toxicity score (ethical/bias pillar); trust calibration index, semantic diversity metrics (variability-management pillar); anomaly detection rate, correction success rate (uncertainty-handling pillar).
- **LLMOps common failure categories** (with root cause and fix): cost inefficiency (redundant large-model calls → use smaller models + caching + autoscaling), hallucination (no grounding → integrate RAG with verified sources), stale responses (training cutoff → external tool/API lookups), untracked performance (no telemetry → MLflow/W&B), poor enterprise-jargon understanding (generic corpus → fine-tune/embed proprietary docs), unauthorized access (weak IAM → enforce RBAC), data leakage (unmasked logs → PII anonymization), missing audit trail (no logging → log prompts/responses/user IDs securely), data-residency violation (cross-region data flow → region-aware endpoints).

## Mental Models
- **"No single metric is universally optimal"**: metric choice must match the task (text/code/image), the desired quality attribute (creativity vs. factuality), the risk profile (healthcare/finance need robustness+fairness; creative apps prioritize novelty), and regulatory context — always justify metric selection against these four factors rather than defaulting to whatever's easiest to compute.
- **Evaluation as a multidisciplinary decision, not just a technical one**: technical experts (implement metrics), domain experts (define quality attributes), end users (usability feedback), product managers (business KPI alignment), ethicists/legal (compliance), and evaluation specialists (methodology rigor) must all weigh in — a metrics choice made by engineers alone misses real-world risk.
- **Evaluation trade-offs are inherent, not bugs**: optimizing accuracy can reduce diversity; optimizing efficiency can reduce output quality — practitioners must explicitly balance metrics against priorities rather than seeking a single "best" configuration.
- **The evaluation feedback loop drives retraining decisions**: collect results (automated + human) → error analysis → prioritize by severity/impact → targeted fix (prompt/data/architecture) → retrain/fine-tune → redeploy → remonitor — evaluation isn't a one-time gate, it's a continuous loop that decides *what* to fix next.

## Anti-patterns
- **Relying solely on reference-based metrics (BLEU/ROUGE) for creative or open-ended generation**: penalizes correct-but-differently-phrased outputs and requires resource-intensive reference-text creation for every scenario.
- **Treating accuracy as the only evaluation dimension**: ignores robustness, fairness, and reliability — a model that's 95% accurate but fails silently on adversarial input or biased demographics is not production-ready.
- **No baseline/telemetry before fine-tuning**: makes it impossible to know whether a fine-tune actually improved anything (book's own example: "No baseline to compare post-fine-tune quality").
- **Full model fine-tuning as a default for cost/accuracy problems**: expensive and slow — prefer LoRA/QLoRA for parameter-efficient tuning, or prompt/RAG fixes first.
- **Ignoring cold-start latency when choosing serverless for real-time inference**: serverless is cost-effective for bursty/event-driven workloads but a poor fit when consistent low latency matters.
- **Storing unmasked PII in logs or preview outputs**: a recurring root cause of data-leakage incidents in the book's issue table — always anonymize before persisting or displaying LLM interaction logs.

## Reference Tables
**Comprehensive evaluation metrics by modality** (condensed)
| Metric | Task | Key feature |
|---|---|---|
| BLEU | Text/Code | N-gram overlap w/ reference |
| ROUGE | Text | Recall-oriented, summarization |
| METEOR | Text | Synonyms + word order |
| BERTScore/MoverScore/BLEURT | Text | Embedding-based semantic similarity |
| TIGERScore | Text | Reference-free, instruction-guided, explainable error analysis |
| CTC-Score | Text | Factual consistency with source |
| CodeBLEU | Code | AST-aware syntactic/semantic overlap |
| pass@k | Code | Functional correctness via unit tests |
| RepoExec / DIR | Code | Repository-level correctness + dependency usage |
| Inception Score / FID | Image | Quality/diversity vs. real image distribution |
| LPIPS / SSIM | Image | Perceptual/structural similarity |

**Nonfunctional parameter thresholds (representative examples from the book)**
| Dimension | Benchmark | Action threshold |
|---|---|---|
| Latency | <500ms text gen | >1.5s triggers fallback to cached output |
| Availability | 99.9% monthly uptime | <99.5% flags incident review |
| Error rate | <1% invalid responses/10k | >3% triggers retraining review |
| Adversarial robustness | >90% attack detection | <80% triggers defense tuning |
| Prompt injection | <1% injection success | Any detected attempt auto-disables session |
| User satisfaction | NPS>60, SUS>80 | Drop >10 NPS pts/2wks alerts product team |

**Deployment strategy trade-offs**
| Strategy | Best for | Key risk |
|---|---|---|
| Cloud | Elastic scale, GPU/TPU access | Cost creep, data-privacy/latency concerns |
| On-premises | Regulated/sensitive data, full control | High capex, scalability ceiling |
| Hybrid | Mixed sensitivity workloads | Management complexity, data sync |
| Edge | Real-time, offline, privacy-critical | Limited compute (needs quantization/pruning) |

**Deployment topology quick-pick**
| Need | Topology |
|---|---|
| PoC / dev only | Single-node |
| High-traffic inference, fault tolerance | Multinode/clustered |
| Independent component scaling | Microservices |
| Bursty/event-driven, no infra mgmt | Serverless |
| Global low-latency, multi-region | Distributed |
| Real-time edge + heavy cloud training | Cloud-edge hybrid |

## Worked Example
**IDFE production deployment pipeline** (Azure DevOps → AKS, 3 stages): (1) **Containerize** — Dockerfile installs Python + MSSQL ODBC drivers, copies app, exposes port 8002, runs `fastapi_main.py`. (2) **Build & push** — Azure DevOps YAML pipeline logs into Azure Container Registry (`az acr login`), builds and tags the image (`latest` + `$(imageVersion)`), pushes to ACR. (3) **Deploy to AKS** — packages a Helm chart, retrieves AKS credentials (`az aks get-credentials`), and runs `helm upgrade --install` against environment-specific `values.yaml`/`dev-values.yaml` (differing replica counts, ingress rules, secrets sourced from Key Vault). Manual fallback path documented too: `docker build` → `az acr login` → `docker push` → `kubectl apply -f <file>.yml` → `kubectl rollout restart deployment -n <namespace>`.

The Helm `values.yaml` demonstrates production-grade config: `replicaCount: 2`, resource requests (`cpu: 500m`, `memory: 500Mi`), ingress with TLS via Key-Vault-issued cert, secrets injected from Azure Key Vault (never hardcoded), and autoscaling parameters (`minReplicas`/`maxReplicas`/`targetCPUUtilizationPercentage`) — showing exactly how the Ch4/Ch5 security and scalability principles land in a real deployment manifest.

## Key Takeaways
1. Build an evaluation plan around all four pillars (Quality, Safety, Efficiency, Ethics) before launch — a system optimized only for Quality metrics will still fail a security or fairness audit later.
2. Match evaluation metrics to modality and task: text→BLEU/ROUGE/BERTScore/TIGERScore, code→pass@k/CodeBLEU, image→FID/LPIPS — and prefer reference-free metrics for open-ended/creative generation.
3. Every nonfunctional parameter needs an explicit benchmark AND an action threshold — "we monitor latency" is not the same as "P95 latency >1.5s auto-triggers cached fallback."
4. Treat evaluation as multidisciplinary: pull in domain experts, end users, product, and legal/ethics — not just ML engineers — when choosing what to measure.
5. Choose deployment strategy and topology from data sensitivity + latency + cost + compliance constraints, not from whichever is trendiest; hybrid and cloud-edge-hybrid exist precisely because most enterprises don't fit a single-strategy answer.
6. Automate the full container→registry→Kubernetes pipeline via CI/CD (Helm + environment-specific values files) so deployments are repeatable and auditable, with secrets always sourced from Key Vault rather than hardcoded.

## Connects To
- **Ch 5**: this chapter's evaluation methodology formalizes the accuracy/precision/recall approach IDFE used ad hoc; the AKS deployment pipeline here is the production build-out of IDFE's architecture.
- **Ch 3**: reuses Blue-Green/Canary deployment patterns and the Monitoring & Feedback Loop pattern from the design-patterns chapter, now applied specifically to LLMOps rollout.
- **Ch 7**: the ethics/fairness/bias evaluation pillar here is expanded into the full Responsible AI and Risk Framework in the next chapter.
- **Ch 4**: MCP's tool/resource discovery and agent registry concepts underpin the RAG vector-DB and external-tool-lookup fixes referenced in the LLMOps issues table.
