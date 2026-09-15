# Chapter 9: Mastering Agent Prompts with Prompt Flow

## Core Idea
Prompt engineering is inherently iterative, and OpenAI's "Test Changes Systematically" strategy demands a systematic infrastructure — Microsoft's Prompt Flow — for running prompt/profile variants, batch-testing them against many inputs, and scoring results with rubric-based LLM-as-judge grounding, so you can pick the "best" agent profile with evidence rather than vibes.

## Frameworks Introduced
- **Systemic Prompt Engineering Loop**: iterate a prompt -> evaluate the response -> refine -> repeat, formalized as infrastructure rather than manual trial-and-error in the ChatGPT UI.
  - When to use: whenever you have more than a couple of prompt candidates or need repeatable, comparable evaluation.
  - How: use Prompt Flow's variant + batch-run + evaluation-flow pipeline instead of manually re-running chats.
- **Prompt Flow**: Microsoft's Azure ML-originated, now open-source, visual/YAML flow tool for building, running, and evaluating LLM prompt pipelines (blocks: Inputs -> LLM/Python blocks -> Outputs).
  - When to use: any time you need reproducible, scalable (multi-threaded batch) evaluation of prompts/profiles, or want to deploy a flow as a local app/API/container.
  - How: define a `flow.dag.yaml` of blocks; connect an LLM resource (connection, API type, model, temperature, stop, max tokens); write prompts as Jinja2 templates; add Python `@tool` blocks for parsing/aggregation; run single, run multiple variants, or batch-run against a JSONL input file.
- **Rubric + Grounding Evaluation**: an education-derived structured scoring method applied to LLM output quality.
  - When to use: whenever "is this response good?" isn't a simple right/wrong check — i.e., almost always for agent profile quality.
  - How: (1) identify purpose/objectives, (2) define specific measurable criteria, (3) create a rating scale (e.g., 1-5), (4) write descriptions per scale level, (5) apply the rubric (manually or via a second LLM prompt), (6) calculate a total/weighted score, (7) ensure evaluator consistency, (8) review/revise/iterate. Grounding = how well a response aligns with the rubric's criteria/context — a well-grounded response satisfies all criteria in context; a poorly grounded one misses them.
- **LLM-as-Judge Grounding**: use a second LLM prompt/block to score the first LLM's output against the rubric automatically.
  - When to use: to remove human evaluation bottleneck/bias at scale; prefer a different model than the one being evaluated (same model is fine only when comparing profiles against each other, not against an absolute baseline).
  - How: write an evaluation prompt encoding the rubric criteria and scale; feed it the generated output; parse its structured (JSON-like) scores with a Python tool block; aggregate scores (e.g., average per criterion) across a batch run.

## Key Concepts
- **Agent profile (formal definition)**: the full set of component prompts describing an agent — persona plus instructions/strategies for actions/tools, knowledge, memory, reasoning, evaluation, planning, feedback — more than just a system prompt.
- **Jinja2 templates**: the templating engine Prompt Flow (and the book) uses to define prompt/profile content with variable injection and light logic.
- **Variant**: an alternate version of a prompt/profile (or LLM, temperature, max tokens, advanced params, or function calls) that Prompt Flow can run side-by-side for comparison.
- **Batch run**: running a flow across many input rows (a JSONL file of varied criteria) in parallel worker processes, rather than one input at a time.
- **Evaluation flow**: a separate Prompt Flow flow (often pure Python, no LLM blocks) that scores and aggregates a prior run's results — decoupled so it can be reused across any recommendation run.
- **Key LLM connection parameters**: connection (service), API type (`chat` vs `completion`), model/deployment, temperature, stop, max tokens, advanced params (top_p, presence_penalty, frequency_penalty, logit_bias).
- **Input priming options (UI vs. conversational)**: constrained UI/form inputs (validated, but limits future use cases) vs. a conversational proxy agent that extracts inputs naturally (more flexible, harder to evaluate consistently).

## Mental Models
- Treat any agent profile as provisional until it's been run through variant comparison + rubric-based grounding — "it looks good in one chat" is not evidence.
- Use a different (often stronger) LLM as judge when establishing an absolute quality baseline; use the same LLM as judge only when the comparison is relative (profile A vs. profile B).
- Decouple generation flows from evaluation flows — this lets one evaluation rubric be reapplied across many different generation runs/variants without rebuilding it each time.

## Anti-patterns
- **Evaluating prompts by eyeballing single-example outputs**: "the results will likely look similar" at small scale — real differences between variants often only appear under batch evaluation across many varied inputs.
- **Skipping rubric definition and jumping to automated scoring**: without explicit criteria/scale/descriptions first, an LLM judge's scores are ungrounded and hard to interpret or trust.
- **Using the same model for both generation and evaluation when trying to establish an absolute baseline**: this can bias the evaluation toward whatever that model favors; use a different model for baseline grounding.
- **Manually parsing every LLM text response with ad hoc string code when structure could be requested/parsed more systematically**: the chapter shows a `parsing_results.py` `@tool` block converting rubric text into a clean dictionary — treat structured parsing as a standard pipeline stage, not a one-off hack.

## Code Examples
```python
from promptflow import tool

@tool
def parse(input: str) -> str:
    rblocks = input.strip().split("\n\n")

    def parse_block(block):
        lines = block.split('\n')
        rdict = {}
        for line in lines:
            kvs = line.split(': ')
            key, value = kvs[0], kvs[1]
            rdict[key.lower()] = value
        return rdict

    parsed = [parse_block(block) for block in rblocks]
    return parsed
```
```python
@tool
def aggregate(processed_results: List[str]):
    items = [item for sublist in processed_results for item in sublist]
    aggregated = {}
    for item in items:
        for key, value in item.items():
            if key == 'title':
                continue
            if isinstance(value, (float, int)):
                aggregated[key] = aggregated.get(key, 0) + value
    for key, value in aggregated.items():
        value = value / len(items)
        log_metric(key=key, value=value)
        aggregated[key] = value
    return aggregated
```
- **What it demonstrates**: the two-stage evaluation pipeline — `parse` converts raw LLM rubric-scoring text ("Title: X\nSubject: 5\nFormat: 5...") into structured dictionaries, and `aggregate` flattens and averages numeric criterion scores across a whole batch run, logging each as a metric for cross-run comparison via Prompt Flow's Visualize Runs feature.

## Reference Tables
| Variation option | Example comparison | Notes |
|---|---|---|
| Jinja2 prompt template | System-prompt injection vs. user-prompt injection | Effectively unlimited prompt-engineering techniques to try |
| LLM | GPT-3.5 vs. GPT-4, commercial vs. open-source | Also useful for tuning a profile to work on cheaper/local models |
| Temperature | 0 (deterministic) vs. 1 (max variability) | Can significantly change response quality/consistency |
| Max tokens | Small vs. large limits | Trade-off between cost and completeness |
| Advanced parameters | top_p, presence_penalty, frequency_penalty, logit_bias | Covered in later chapters |
| Function calls | Alternative function/tool definitions | Ties back to chapter 5's actions |

| Rubric rating | Description |
|---|---|
| 1 | Poor alignment — opposite of expected |
| 2 | Bad alignment — not a good fit |
| 3 | Mediocre alignment — may or may not fit |
| 4 | Good alignment — not 100% but a good fit |
| 5 | Excellent alignment — a good recommendation for the criteria |

## Worked Example
The chapter builds and evaluates a generalized recommendation profile end to end: (1) a base flow (`recommender_with_variations`) exposes Subject/Genre/Format/Custom inputs and defines two prompt variants — one injecting inputs into the user prompt, one into the system prompt; (2) a rubric is defined (subject/format/genre alignment, 1-5 scale) and encoded as an `evaluate_recommendation.jinja2` LLM-judge prompt, producing text like `Title: The Butterfly Effect / Subject: 5 / Format: 5 / Genre: 4`; (3) a `parsing_results.py` tool block converts this into structured dictionaries; (4) a 10-row JSONL file of varied subject/format/genre/custom combinations (generated by asking ChatGPT to produce it) drives a batch run of both variants; (5) a separate `evaluate_groundings` flow (pure Python `score` + `aggregate` blocks) computes per-criterion average scores and logs them as metrics; (6) Prompt Flow's Visualize Runs / Run-against-Run comparison reveals that the system-prompt-injection variant scores measurably better than the user-prompt-injection variant — a data-backed answer to "which profile design is better," not a guess.

## Key Takeaways
1. An agent profile is the full bundle of component prompts (persona + actions/knowledge/memory/reasoning/planning instructions), not just a system prompt.
2. Systematic (not ad hoc) prompt engineering requires infrastructure for variant comparison and batch evaluation — Prompt Flow is Microsoft's answer, but the pattern (iterate + evaluate at scale) generalizes to any tool.
3. Build rubrics before automating evaluation: purpose -> criteria -> scale -> descriptions -> apply -> aggregate -> ensure consistency -> iterate.
4. Use a second LLM (ideally a different/stronger model for absolute baselines) as an automated judge against your rubric — this scales evaluation far beyond manual review.
5. Decouple generation flows from evaluation flows so one rubric-scoring pipeline can be reused across many prompt/profile variants and runs.
6. Structure evaluation output early (parse LLM judge text into dictionaries) so aggregation/comparison across a batch is straightforward and visualizable.
7. Small-sample manual comparison often looks inconclusive — batch runs across varied inputs, aggregated and visualized, are what actually reveal which profile design wins (e.g., system-prompt vs. user-prompt input injection).

## Connects To
- **Ch2**: directly implements the "Test Changes Systematically" prompt engineering strategy introduced there.
- **Ch5**: notes that structured-output parsing (tool/function calling) is a better long-term alternative to hand-rolled text parsing shown here.
- **Ch10**: rubric-based grounding here is a precursor to the fuller agent reasoning/evaluation systems covered next.
