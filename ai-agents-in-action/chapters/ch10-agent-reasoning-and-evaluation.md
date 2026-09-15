# Chapter 10: Agent Reasoning and Evaluation

## Core Idea
LLMs aren't trained to reason, but prompt engineering can elicit reasoning-like behavior along a spectrum of increasing power and cost — from direct/few-shot/zero-shot prompting, through chain-of-thought and prompt chaining, to self-consistency and tree-of-thought — and because no technique guarantees a correct answer, evaluation (embedding similarity, LLM-judged scoring, or consistency voting) must be paired with generation to have any confidence in the result.

## Frameworks Introduced
- **Direct Solution Prompting Family**: question-answer, few-shot, zero-shot prompting.
  - When to use: question-answer for grounded Q&A over given context (RAG-style); few-shot when you need the LLM to follow a demonstrated pattern (even a fictitious one); zero-shot when you want the LLM to generalize from internal knowledge with only rules/guidelines, no examples (e.g., sentiment classification without naming "sentiment").
  - How: question-answer injects context + question into the prompt; few-shot gives example input/output pairs establishing a pattern before the real input; zero-shot gives only instructions/format, no examples.
- **Chain of Thought (CoT) Prompting**: few-shot examples that include worked reasoning steps, not just answers.
  - When to use: multi-step logic/math/word problems where showing *how* to reason (not just what to answer) improves the model's own step-by-step output.
  - How: write example problems paired with explicit step-by-step reasoning chains in the system prompt; end with "Think step by step but only show the final answer."
- **Zero-Shot CoT**: the "magic phrase" trigger — "Let's think step by step" — without needing worked examples.
  - When to use: as a cheaper substitute for full CoT when you want reasoning behavior without authoring expensive example chains.
  - How: add the phrase directly to the system prompt; ask for a single final-statement answer to keep output parseable.
- **Prompt Chaining**: decompose a problem into sequential LLM calls (decompose steps -> calculate each step -> synthesize final solution), each stage's output feeding the next.
  - When to use: when you want visibility/control over each reasoning stage, or when a single monolithic CoT prompt is too unreliable/verbose.
  - How: stage 1 LLM lists only the steps (no solving); stage 2 LLM calculates each step's output only; stage 3 LLM combines calculated steps into one final concise sentence.
- **Self-Consistency Prompting**: generate the same CoT prompt N times (often via batch run on duplicated input rows, since DAG-based tools like Prompt Flow can't natively loop) and pick the most representative answer by embedding-similarity voting.
  - When to use: when a single CoT run may hit unlucky sampling variance and you want a "wisdom of crowds" answer instead of trusting one generation.
  - How: batch-run the same CoT prompt N times; embed each answer; compute the mean embedding; return the answer whose embedding is most similar (highest cosine similarity) to that mean.
- **Tree of Thought (ToT) Prompting**: combine prompt chaining with self-evaluation at every node, forming a tree of candidate reasoning paths that gets pruned (branches with low evaluation scores don't propagate).
  - When to use: complex problems where CoT/self-consistency both fail — at the cost of significantly more LLM calls (the chapter's example: up to 27 calls per problem).
  - How: at each node, generate a candidate next-step *and* evaluate it (e.g., score 0-100); only propagate to children if the evaluation exceeds a threshold (e.g., >25); execute breadth-first (depth-first is impractical in a DAG-based tool like Prompt Flow) so each level is evaluated before expanding further.

## Key Concepts
- **Reasoning**: the ability to understand the process of thought/thinking through a problem — that actions have outcomes, and to select actions accordingly.
- **Planning**: reasoning out the order of actions/tasks and applying correct parameters to achieve a goal; can span strategic, tactical, operational, and contingent levels.
- **Zero-shot / one-shot / few-shot learning**: terms borrowed from ML describing how many examples a model needs to generalize to a new pattern; LLMs can do all three via prompting alone, without retraining.
- **Evaluation via embedding similarity**: comparing predicted vs. expected answers by embedding both and computing similarity/distance — a quantitative stand-in for "is this correct."
- **DAG (directed acyclic graph) limitation**: Prompt Flow (and similar tools) execute as DAGs — no native loops — so techniques requiring repetition (self-consistency) must simulate it via batch processing over duplicated inputs rather than true loops.
- **Breadth-first vs. depth-first tree execution**: ToT ideally could go either way, but DAG-based tools can only practically implement breadth-first (evaluate every node at a level before continuing) since depth-first requires dynamic/looping control flow.
- **Node pruning by evaluation score**: in ToT, a node returning empty output signals its evaluation failed (or its parent's did) — this is how invalid reasoning branches get short-circuited without explicit control-flow code.

## Mental Models
- Treat prompting-for-reasoning techniques as a cost/reliability ladder: direct/zero-shot (cheap, least reliable for hard problems) -> CoT/zero-shot-CoT (moderate cost, better for structured problems) -> prompt chaining (more control, still can fail) -> self-consistency (more calls, better for problems with "typical" correct answers) -> ToT (most calls, most robust to bad reasoning paths, but expensive).
- Never trust a single LLM-generated plan/answer for a nontrivial reasoning task without some form of evaluation (similarity scoring, LLM-judge scoring, or consistency voting) — the chapter's running time-travel problem is repeatedly gotten wrong even by CoT and self-consistency, underscoring this.
- When a DAG-based orchestration tool can't loop, simulate repetition via batch processing over duplicated/varied input rows instead of trying to force iteration into the graph.

## Anti-patterns
- **Assuming CoT or any single reasoning technique guarantees a correct answer**: the chapter's own worked example shows CoT, zero-shot-CoT, prompt chaining, and self-consistency all producing incorrect answers to the same complex time-travel problem at various points — reasoning-eliciting prompts improve odds, they don't guarantee correctness.
- **Treating "most consistent" as "most correct" in self-consistency**: the most frequent/similar answer across samples is not necessarily the right one — it's a proxy that works better for simpler problems and can still converge on a wrong-but-popular answer.
- **Applying expensive techniques (CoT with worked examples, ToT) indiscriminately**: CoT is "expensive in terms of prompt generation" per problem class, and ToT can cost a dozen-plus LLM calls per answer — reserve heavier techniques for problems where cheaper prompting has demonstrably failed.
- **Letting evaluation prompts return anything other than a clean, parseable score**: the chapter notes an evaluation prompt call got confused when the "predicted" answer was verbose rather than a single final statement — always instruct generation prompts to isolate the final answer for downstream evaluation.

## Code Examples
```python
from promptflow import tool
from typing import List
import numpy as np
from scipy.spatial.distance import cosine
from promptflow import log_metric

@tool
def consistency(texts: List[str], embeddings: List[List[float]]) -> str:
    if len(embeddings) != len(texts):
        raise ValueError("The number of embeddings must match the number of texts.")
    mean_embedding = np.mean(embeddings, axis=0)
    similarities = [1 - cosine(embedding, mean_embedding) for embedding in embeddings]
    most_similar_index = np.argmax(similarities)
    log_metric(key="highest_ranked_output", value=texts[most_similar_index])
    return texts[most_similar_index]
```
- **What it demonstrates**: the self-consistency voting mechanism — compute the centroid (mean) embedding of all sampled answers, then return whichever individual answer is closest to that centroid, operationalizing "most consistent answer" as "closest to the average" rather than picking a majority-vote string match.

## Reference Tables
| Technique | Examples needed? | Relative cost | Best for |
|---|---|---|---|
| Question-Answer / Direct | No (context given) | Low | Grounded Q&A over provided content |
| Few-shot | Yes (input/output pairs) | Low-moderate | Pattern-following, including novel/fictitious concepts |
| Zero-shot | No | Low | Generalization tasks (e.g., classification) using internal knowledge only |
| Chain of Thought (CoT) | Yes (worked reasoning examples) | Moderate-high | Multi-step logic/math problems |
| Zero-shot CoT | No (magic phrase only) | Low-moderate | Cheaper substitute for CoT |
| Prompt chaining | No | Moderate (multiple calls) | Visibility/control over each reasoning stage |
| Self-consistency | No (same CoT prompt, N samples) | High (N calls + evaluation) | Reducing variance/sampling luck on a single CoT run |
| Tree of Thought (ToT) | No (chaining + per-node evaluation) | Very high (up to dozens of calls) | Complex problems where simpler techniques fail |

## Worked Example
The chapter uses one running "hard" problem throughout — Alex the time traveler (arrives 3 days before a 10-day battle, spends 6 days in the past, jumps forward 50 years for 20 days, returns to see the battle's end; correct answer: 3 initial + 3 more + 20 + 7 = 33 days) — to stress-test every technique: direct/zero-shot CoT with worked time-travel examples gets partial/incorrect answers; prompt chaining (decompose_steps -> calculate_steps -> calculate_solution) produces "13 days" (still wrong but shows full reasoning trace); self-consistency batch-runs the same CoT prompt 5 times on duplicate inputs and picks the answer closest to the mean embedding (still not guaranteed correct); tree of thought builds a breadth-first tree of candidate reasoning branches, evaluating and pruning at each level (nodes returning empty text failed evaluation or had a failed parent), ultimately surfacing candidate answers like "29 days" or "9 days" at different leaves — illustrating that even the most expensive technique in the chapter doesn't reliably converge on the correct 33-day answer, driving home that reasoning-elicitation and answer-correctness are separate problems requiring separate evaluation.

## Key Takeaways
1. LLMs aren't trained to reason, but prompting can elicit reasoning-like behavior — treat this as "extracting a generality" from training data, not literal thought.
2. Match the technique to the problem: zero-shot for generalization/classification, few-shot for pattern-following, CoT/prompt-chaining for multi-step logic, self-consistency/ToT for complex problems where simpler techniques fail.
3. "Let's think step by step" (zero-shot CoT) is a remarkably cheap, effective trigger for reasoning behavior without authoring worked examples.
4. Always pair generation with evaluation (embedding similarity, LLM-judge scoring, or consistency voting) — no single reasoning technique in this chapter reliably produced the correct answer to the running example problem.
5. Self-consistency approximates "most consistent" via nearest-to-mean-embedding voting, not majority string matching — and consistency is not the same as correctness.
6. Tree of Thought's per-node evaluation-and-pruning approach handles complex problems more robustly than flat CoT, but at a steep multiplier in LLM calls (dozens per answer) — reserve it for cases that justify the cost.
7. DAG-based orchestration tools (Prompt Flow) can't natively loop — simulate repetition (self-consistency) via batch processing over duplicated inputs rather than forcing iteration into the graph.

## Connects To
- **Ch1**: implements the "reasoning/evaluation" component of the five-component agent model.
- **Ch9**: builds directly on Prompt Flow's variant/batch-run/evaluation-flow infrastructure introduced there.
- **Ch11**: reasoning and evaluation techniques here become the building blocks for the fuller planning-and-feedback systems in the next chapter.
