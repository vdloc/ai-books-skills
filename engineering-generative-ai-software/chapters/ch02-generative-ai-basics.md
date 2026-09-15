# Chapter 2: Generative AI Basics — How Instruct Models Work

## Core Idea
Instruct models are pre-trained language models wrapped in a fine-tuned instruction-following pipeline; beyond that, you can specialize behavior cheaply at inference time using zero/one-shot, few-shot, and chain-of-thought prompting instead of retraining.

## Frameworks Introduced
- **Instruct model architecture**: pre-trained language model + a smaller fine-tuned instruction/reasoning layer trained on (instruction, input, output) triples.
  - When to use: whenever you want conversational or task-following behavior rather than raw next-token completion.
  - How: fine-tune a base model (e.g., GPT-2) on an instruction dataset (e.g., Alpaca-style Python instructions) using the same tokenize → `Trainer` pipeline as Ch1, but with instruction+input as source and output as target.
- **Zero-shot / One-shot / Few-shot / Chain-of-Thought (CoT) prompting** — a ladder of cheap, no-retraining specialization techniques (Wei et al., 2022):
  - Zero-shot: a plain instruction, relying entirely on pre-trained + instruct-tuned knowledge.
  - One-shot: one example embedded in the prompt to bias the model's attention.
  - Few-shot: several positive/negative examples in-context; the model generalizes from them for that session only (no weights change).
  - CoT: prompt explains the reasoning pattern *before* giving examples, and asks the model to reason step-by-step before answering — improves accuracy on classification/reasoning tasks by causing the model to explicitly state which features it's keying on.
  - When to use CoT over few-shot: when the answer depends on integrating multiple weak signals (e.g., subtle sentiment/spam cues) rather than a single keyword match.
- **"Solve by creating a method" pattern**: instead of asking the model to answer directly, ask it to generate a program/artifact that solves the problem, execute that artifact, then have the model interpret the result — an early, minimal form of RAG/tool-use (foreshadows Ch7's agents).

## Key Concepts
- **Instruct pipeline**: non-ML scripting layer that sits around the model to manage conversation state, prompt abridging, and post-processing.
- **Conversation growth**: every prior prompt/response stays in context; "forget the previous solution" does not remove earlier tokens from the model's input — a key debugging fact when outputs seem stuck on earlier context.
- **Safe-for-work (SFW) filtering**: keyword/rule-based post-processing to catch NSFW, violent, or offensive content before showing it to a user.
- **Hybrid necessity**: some tasks (fact-checking, strict templates, complex multi-step reasoning) are not solvable by training/prompting alone — they need software wrapped around the model (search engines, template synchronizers, delegated programs).

## Mental Models
- Treat few-shot/CoT prompting as "temporary, session-scoped training" — it changes behavior without touching weights, so it is cheap but must be re-supplied every session/request.
- When a multi-turn conversation "goes wrong" after a correction, remember the model still sees the wrong turn — sometimes it helps, sometimes it derails; don't assume "forget X" actually erases X from context.

## Anti-patterns
- **Relying on iterative prompt corrections inside one conversation to fix wrong output**: because history persists, later prompts inherit earlier mistakes; may require starting a fresh context instead.
- **Skipping post-processing/guardrails because the base model "usually" behaves**: the book's SFW example shows this is a simple, cheap addition that should not be skipped for production systems.
- **Using training/fine-tuning when a template-and-fill (hybrid) approach solves the same problem more reliably**: e.g., structured form generation is often better solved by generating parts separately and syncing them in code.

## Worked Example
Building a spam classifier via increasing prompting sophistication, then via "solve by creating a method":
1. **Few-shot**: give 3 spam + 3 non-spam example titles, then ask the model to classify a new message. Model may misclassify.
2. **Chain-of-thought**: same examples, but explicitly told "here's why these are spam" (shared feature: urgency/inheritance language), plus "reason step by step then answer." Model correctly classifies "Act now to secure your lifetime membership at 70% off" as Spam, and states the reasoning ("urgent, promotional language... matches known spam patterns").
3. **Solve by creating a method**: instead of asking the model to classify directly, prompt it to *write a Python program* (using `CountVectorizer` + `MultinomialNB`) that classifies messages, run that program, then ask the model to *explain* the program's output. This delegates the actual decision to a deterministic classifier — more exact, and the pattern generalizes to any RAG-like architecture where the model reasons about a tool's output rather than generating the answer directly.

## Key Takeaways
1. Instruction-following behavior comes from fine-tuning on (instruction, response) pairs on top of a pre-trained model — it is a separate training stage from Ch1's pre-training.
2. Zero/one/few-shot and CoT prompting let you specialize behavior per-request without retraining — pick CoT specifically when reasoning must combine several weak cues.
3. Conversations are stateful and cumulative; corrections do not erase prior context, which explains "sticky" wrong answers.
4. Post-processing (SFW filters, output validation) is a required software-engineering layer around the model, not optional; it belongs in your pipeline design from day one.
5. Delegating computation to a generated program (then interpreting its output) is more reliable than pure text generation for tasks needing exactness — this is the conceptual seed of RAG and agentic tool-use (Ch5, Ch7).

## Connects To
- **Ch5**: the "solve by creating a method" pattern is architecturally formalized as RAG (LangChain/vector DB + LLM summarization).
- **Ch7**: agentic AI extends this further by giving models tools (compilers, databases) to validate their own output.
- **Ch1**: instruct models are built on the pre-trained transformer foundation from Chapter 1.
