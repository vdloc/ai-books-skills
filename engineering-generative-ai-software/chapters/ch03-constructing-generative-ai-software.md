# Chapter 3: Constructing Generative AI Software — Developing

## Core Idea
Building generative AI software requires merging two disciplines that historically worked separately — MLOps (productive ML) and Agile/Lean software engineering — into one iterative process spanning requirements, training/testing, architecture, and deployment, all reshaped by the model's probabilistic nature.

## Frameworks Introduced
- **MLOps + AI Engineering combined lifecycle**: requirements → model training/testing → architecture → testing → deployment, run as an iterative loop rather than a line, because generative AI is probabilistic at every phase.
  - When to use: any time software includes a generative AI component — not just at "the ML part."
  - How: requirements phase folds in model non-functional requirements (accuracy, latency, prompt/output engineering); testing splits into oracle-based (deterministic API tests) and benchmark-based (BLEU/ROUGE/model quality) tracks (Fig 3.4); deployment decides cloud vs. local vs. hybrid.
- **Lean Startup / iterative release for generative AI** (Reis, 2011): ship an imperfect but usable MVP, measure user + model-quality signals, iterate — exemplified by ChatGPT's own evolution.
  - When to use: instead of chasing a "perfect model," release early and let usage data (accuracy, latency, satisfaction) drive the next iteration.
- **Dual-track testing** (Fig 3.4): traditional software testing (unit → integration → system → UAT) run in parallel with ML-specific testing (model benchmark scores → API-level testing under load → full-system testing with target users). Both are necessary because generative AI mixes probabilistic model behavior with deterministic API/software behavior.

## Key Concepts
- **MLOps**: "designing and maintaining productive ML" (Treveil et al., 2020).
- **AI Engineering**: "an extension of Software Engineering with new processes and technologies needed for development and evolution of AI systems" (Bosch et al., 2021).
- **BLEU** (Bilingual Evaluation Understudy): n-gram overlap metric (0–1) between generated and reference text; clipped variant avoids trivial repetition gaming.
- **CodeBLEU**: BLEU variant using ASTs instead of raw n-grams, suited to comparing generated *code* rather than natural language.
- **ROUGE**: recall-oriented n-gram metric for summarization tasks.
- **Concept drift**: a model's learned distribution becomes stale as real-world data changes over time — a data-quality risk unique to ML-based software (Staron, 2024).

## Mental Models
- Treat "functional requirements" for generative AI software as deceptively simple (e.g., "generate Python from an English prompt") — nearly all the engineering effort is in the *non-functional* requirements (next chapter) and in choosing the right benchmark metric for the task.
- When picking an evaluation metric, match it to the artifact type: BLEU for natural language, CodeBLEU for programs, ROUGE for summaries — using the wrong metric (e.g., BLEU on code) actively rewards degenerate behavior (verbatim copying).

## Anti-patterns
- **Treating ML model training and software engineering as separate, sequential silos** (the author's own early-career anecdote): produces a great confusion matrix but no insight into which errors matter to the product — combine the two disciplines from the start.
- **Optimizing purely for "a bigger/better model" instead of customer value**: the book contrasts OpenAI's ChatGPT (smaller model, but shipped fast, focused on value) against a technically larger contemporaneous model that lost market relevance.
- **Using BLEU on generated source code**: encourages copy-paste-like outputs; use CodeBLEU (AST-based) instead.

## Worked Example
Calculating and contrasting BLEU vs. CodeBLEU:
- Reference sentence: "Santa Claus is coming to town." Candidate "Santa Santa Santa Santa Santa Claus" scores BLEU 1.0 uncapped (all words present) but only 1/3 with clipping (words capped at reference count) — showing why raw BLEU is unreliable without clipping, and why word order isn't checked (bigram BLEU catches some of this, dropping the score to 0.2).
- For code: comparing `def hello(): print("hello world")` against two candidates via CodeBLEU (AST-based) gives 0.68 and 0.61 for near-identical programs with renamed variables/messages, vs. 0.01 for an unrelated `sum(x,y)` function — demonstrating CodeBLEU correctly rewards structural similarity regardless of superficial token differences that would confuse BLEU.

## Key Takeaways
1. Generative AI software development is one iterative loop combining Agile/Lean principles with MLOps/AI Engineering — not two separate pipelines bolted together.
2. Testing must run two parallel tracks: benchmark-based model quality checks and classical oracle-based software tests (unit/integration/system) — see Ch3 pytest+benchmark example.
3. Pick evaluation metrics to match the output type: BLEU (text), CodeBLEU (code, AST-based), ROUGE (summaries) — using the wrong metric produces misleading "good" scores.
4. New/expanded roles are required beyond classical software engineering: ML engineers (training/optimization), data scientists (data governance/quality), infrastructure managers (cost/scalability/security), alongside software engineers/architects/testers/product managers who now also need ML literacy.
5. Ship early and iterate (Lean Startup) rather than waiting for a "perfect" model — this is validated by ChatGPT's own release history.

## Connects To
- **Ch4**: dives deeper into the functional/non-functional requirements introduced here.
- **Ch5**: expands on the "software architecting" phase mentioned in the MLOps lifecycle.
- **Ch6**: expands on the "testing" phase, especially metamorphic testing for models.
