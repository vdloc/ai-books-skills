# Chapter 6: Evaluating Your RAG Application

## Core Idea
RAG evaluation splits into retrieval metrics (is the right context surfaced?) and generation metrics (is the response faithful, complete, and relevant?), and modern reference-free techniques (UMBRELA, AutoNuggetizer, HHEM) make continuous, scalable measurement possible without hand-curated "golden" datasets.

## Frameworks Introduced
- **Offline vs. online evaluation**: offline = deep, resource-heavy, run during development against a fixed benchmark (your primary tuning engine); online = lightweight, run on live traffic (your early-warning system for real-world drift). Use both — offline for tuning, online for monitoring.
- **RAG Failure Taxonomy** (three pipeline stages): **Retrieval failures** (failure to retrieve/low recall, irrelevant retrieval/low precision, architectural limits on multi-hop/sensemaking queries) → **Generation failures** (faithfulness/hallucination, context utilization failure, answer relevance failure) → **Ingestion failures** (structural parsing errors, content staleness). Diagnose top-down: fix retrieval before tuning the LLM, since "if your retrieval recall is 40%, no amount of prompt engineering will make your RAG application successful."
- **Faithful-incorrect vs. unfaithful-incorrect**: a response can be perfectly grounded in a chunk yet still wrong for the user (faithful to a *stale* document → fix ingestion/versioning) versus unfaithful (ignores/contradicts the correct chunk → fix the LLM/prompt). This distinction routes the fix to the right pipeline stage.
- **UMBRELA**: reference-free retrieval evaluation — an LLM judge scores each retrieved chunk 0 (irrelevant) to 3 (perfectly relevant) without needing pre-curated golden chunks. Correlates well with human judgment; usable as input to precision/nDCG but not recall (would require scoring every chunk in the dataset).
- **AutoNuggetizer**: reference-free generation/context-utilization evaluation — decomposes relevant chunks into atomic "nuggets" classified as Vital (must appear in a correct answer) or OK (nice to have), then checks whether the generated answer's nugget coverage is Supported/Partially supported/Not supported.
- **Measurement + Tuning as the two jobs of an evaluation framework**: measurement = continuous monitoring baseline (catches regressions); tuning = using the framework to run controlled experiments across chunking/embedding/retrieval/LLM/prompt configurations to find the best combination. A framework that only measures without enabling tuning is half-built.
- **Evaluation gate in CI/CD**: no new component (embedding model, reranker, prompt) ships without passing a "no-regression" check against a versioned benchmark — e.g. faithfulness and retrieval relevance must not drop, P95 latency must not increase >5%.

## Key Concepts
- **Precision@k**: fraction of top-k retrieved chunks that are actually relevant (signal-to-noise of retrieved context).
- **Recall@k**: fraction of all relevant chunks in the dataset that appear in the top-k results (completeness).
- **Precision-recall trade-off**: pushing recall up (return more chunks) tends to drop precision, and vice versa; F1@k (harmonic mean) balances both into one score.
- **MRR (mean reciprocal rank)**: average of 1/rank-of-first-relevant-chunk across queries — best for "find one good answer fast" tasks.
- **MAP (mean average precision)**: averages precision at each position a relevant chunk appears — rewards ranking multiple relevant chunks highly, not just the first.
- **nDCG (normalized discounted cumulative gain)**: the most flexible rank-aware metric — supports graded relevance (0–3 scale, not just binary relevant/irrelevant) and normalizes against an ideal ranking; the "gold standard" for complex retrieval but overkill for small top-k use cases.
- **Golden chunks/answers**: human-curated ground truth needed for traditional IR metrics — expensive to create and maintain, and go stale as source data evolves; this is exactly what reference-free methods (UMBRELA, AutoNuggetizer) avoid needing.
- **Self-referential bias (LLM-as-judge)**: judges systematically favor outputs that "sound like AI" (verbose, polished) over blunt-but-correct answers, and share blind spots with the models they evaluate — mitigate by calibrating against a human-verified golden test set.
- **Citation precision**: whether a response's inline citations actually support the specific statement they're attached to — prevents misattribution and enables user verification.
- **Response consistency**: does the same query, run multiple times, produce factually stable answers? Critical in regulated domains (finance/healthcare/legal) where variable answers to the same question are a compliance risk.
- **Evaluation flywheel**: combining async automated scores (UMBRELA/AutoNuggetizer sampled on live traffic) with human thumbs-up/down feedback and A/B testing (champion vs. challenger pipeline) to continuously promote hard real-world failures back into the offline golden dataset.

## Mental Models
- Diagnose top-down through the pipeline: retrieval → generation → ingestion. A generation-quality problem is often actually a retrieval or ingestion problem in disguise (e.g. "no relevant answer" because the corpus is stale, not because the LLM failed).
- Treat an LLM judge like a piece of production code: version its prompt and model, log its full reasoning alongside its score, and audit it — otherwise you can't tell "quality regressed" from "the yardstick moved" (evaluation drift).
- Think of online evaluation as asynchronous, sampled, and non-blocking by default — never make a user wait for a judge call; log-and-evaluate out-of-band on 5–10% of traffic, which is usually enough for statistical significance.
- User satisfaction (thumbs-up/down) is not just a vanity metric — use it to *validate* your automated metrics via correlational analysis: if low faithfulness scores don't correlate with thumbs-down, your automated metric may be misleading you.

## Anti-patterns
- **Tuning prompts before fixing retrieval**: no amount of prompt engineering compensates for a retriever with 40% recall — always optimize retrieval first.
- **Relying solely on answer-similarity metrics (ROUGE-L, BERTScore) to catch answer-relevance failures**: these compare against a golden answer and miss cases where the response is factually correct but fails to actually answer the user's real question (e.g. listing features instead of answering "is it safe to push?"). Use a custom LLM-as-judge rubric instead.
- **Trusting a single LLM-as-judge run**: LLMs are stochastic — even at temperature 0, a judge can "change its mind" across runs. Average multiple runs or switch to pairwise comparison for stability.
- **Blindly trusting rank-aware metrics as proxies for LLM-perceived quality**: "lost in the middle" affects humans reading ranked lists more consistently than it affects an LLM synthesizing from retrieved chunks — don't over-index on nDCG-style position sensitivity for small top-k RAG use cases.
- **Evaluating 100% of live traffic with LLM-as-judge**: doubles operational cost and adds latency for no proportional benefit — sample 5–10% of interactions instead.
- **Never versioning your benchmark dataset or judge prompts**: causes "evaluation drift," where score changes reflect a moved yardstick, not a real system change — version both alongside code.
- **Ignoring content staleness in ingestion**: even with regular refresh, superseded documents left un-purged will get retrieved and presented as current — use entity IDs + versioning and periodically test for duplicate-entity-ID conflicts.

## Code Examples

```python
# Minimal LLM-as-judge for factuality + answer relevance (Ragas-style)
prompt = f"""
You are an impartial judge evaluating the quality of an answer generated by
a Retrieval-Augmented Generation (RAG) system.

Your task is to evaluate the generated answer based on two criteria:
1. Factuality: Is the generated answer factually grounded in the provided context?
2. Answer Relevance: Is the generated answer relevant and helpful for the given query?

You must provide a score from 1 to 5 for each criterion and a brief explanation.

Query: {query}
Retrieved Context: {context}
Generated Answer: {generated_answer}

Respond only in valid JSON with keys: factuality_score, factuality_reasoning,
relevance_score, relevance_reasoning.
"""
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are an expert evaluator that responds only in valid JSON."},
        {"role": "user", "content": prompt},
    ],
    temperature=0,
)
# Example result: factuality_score=4 (missing unit/condition specificity), relevance_score=4
```
- **What it demonstrates**: a structured, JSON-only LLM-judge prompt that scores two independent criteria with explanations — the base pattern behind Ragas-style evaluation.

```python
# UMBRELA: reference-free retrieval relevance scoring (0-3 scale)
UMBRELA_PROMPT = """
Given a query and a passage, score 0-3:
0 = unrelated, 1 = related but no answer, 2 = has an answer but unclear/buried,
3 = passage is dedicated to and contains the exact answer.
Query: {query}
Passage: {chunk}
Provide only the final integer score.
"""
# query = "How does photosynthesis work in plants?"
# chunk 1 (defines photosynthesis directly) -> Score 3
# chunk 2 (chlorophyll/chloroplasts, related but partial) -> Score 2
# chunk 3 (mitochondria/ATP, unrelated) -> Score 0
```
- **What it demonstrates**: no golden chunks needed — the LLM judge scores relevance per-chunk directly, enabling retrieval evaluation at production scale.

## Reference Tables

| Failure mode | Pipeline stage | Mitigation |
|---|---|---|
| Failure to retrieve (low recall) | Retrieval | Hybrid search or reranker |
| Irrelevant retrieval (low precision) | Retrieval | Hybrid search or reranker |
| Architectural limits (multi-hop/sensemaking) | Retrieval | Agentic RAG or knowledge graphs |
| Hallucination (faithfulness failure) | Generation | Hallucination detection model |
| Context utilization failure | Generation | Prompt engineering to weight all chunks |
| Answer relevance failure | Generation | Custom LLM-as-judge rubric |
| Structural parsing error | Ingestion | Robust logging + specialized parsers |
| Content staleness | Ingestion | Entity IDs + versioning |

| Evaluation framework | Approach | Best for |
|---|---|---|
| Open RAG Eval | Reference-free (UMBRELA, AutoNuggetizer, HHEM, citation, consistency) | Large-scale enterprise, no golden datasets |
| Ragas | LLM-as-judge, needs golden answer dataset | Rich metric library, LangChain/LlamaIndex integration |
| DeepEval | pytest-style unit testing (`assert_test`) | CI/CD regression testing, custom G-Eval criteria |
| Amazon Bedrock Evaluation | Fully managed, Claude-as-judge | AWS-native teams wanting managed infra + responsible-AI scoring |

## Worked Example
Diagnosing a compliance-relevant failure: query "What is the mandatory final approval step for a project launch, and what is the specific deadline for submitting the risk assessment form?" The corpus has Chunk A (Stakeholder Sign-off — relevant), Chunk B (72-hour risk-assessment deadline — relevant), and Chunk C (team celebration budget — irrelevant noise). The retriever surfaces A and C but misses B (a recall failure). The LLM, forced to work with A and C, correctly reports the sign-off step, admits it doesn't know the deadline (avoiding a hallucination), but pads its answer with the irrelevant celebration-lunch detail. Net effect: a technically "non-hallucinated" but practically failed response — the user may wrongly conclude there's no deadline, causing a real compliance miss. Precision@k here is low (2 of 3 relevant among retrieved, if C counted) but recall is the real culprit (B never surfaced) — illustrating why *recall failures*, not hallucination, are often the true root cause of a "bad" RAG answer, and why fixing the retriever (hybrid search/reranking) is the correct first move, not further prompt tuning.

## Key Takeaways
1. Always diagnose in pipeline order — retrieval, then generation, then ingestion — because fixing generation without first fixing retrieval wastes effort.
2. Reference-free metrics (UMBRELA, AutoNuggetizer, HHEM) solve the practical infeasibility of maintaining golden datasets at production scale — trading human labeling cost for LLM inference cost.
3. Precision/recall/F1 and rank-aware metrics (MRR/MAP/nDCG) each answer a different question — pick based on whether "find one good result fast" (MRR) or "rank multiple relevant chunks well" (MAP/nDCG) matches your use case.
4. Treat your LLM judge as production code: version its prompt/model, log full reasoning, and calibrate against human-verified samples to guard against self-referential bias and evaluation drift.
5. Combine offline (tuning engine, gated in CI/CD) and online (async, sampled, non-blocking monitoring) evaluation — they serve different purposes and neither alone is sufficient.
6. Human feedback (thumbs-up/down) is not just a vanity metric — use it to validate whether your automated metrics actually predict user satisfaction via correlational analysis.
7. System metrics (latency P95/P99, uptime, cost, resource efficiency) belong in the same evaluation dashboard as quality metrics — a system with perfect answers but poor reliability will still fail in production.

## Connects To
- **Ch3**: hallucination detection (HHEM) and guardrails introduced there are the concrete mechanisms behind the "faithfulness" metric here.
- **Ch4**: the production KPI table (context precision/recall, hallucination rate, answer relevance, UMBRELA) referenced there is fully defined in this chapter.
- **Ch5**: Vectara's factual-consistency-score and hallucination-correction endpoints are platform-native implementations of the metrics discussed here.
- **Ch7**: agentic RAG is introduced as the answer to "architectural limits" (multi-hop, sensemaking queries) that standard retrieval evaluation flags but can't fix on its own.
