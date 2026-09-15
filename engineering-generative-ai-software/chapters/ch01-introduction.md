# Chapter 1: Introduction — Generative AI and Software Engineering

## Core Idea
Generative AI systems are transformer networks pre-trained to model a language corpus (self-supervised, masked-language-modeling) and then task-specifically fine-tuned; understanding this two-stage pipeline (pre-training → task-specific training) is the foundation for engineering any generative AI software.

## Frameworks Introduced
- **Pre-training vs. task-specific training**: pre-training makes the network "understand" a corpus generically (self-supervised, masking tokens); task-specific training (fine-tuning a small classifier like k-NN, or full fine-tuning) specializes it for one job.
  - When to use: pre-train once on a large representative corpus; fine-tune per-task on much less labeled data.
  - How: build a tokenizer (BPE) → configure the transformer (RoBERTa-style config: vocab size, attention heads, hidden layers) → mask tokens with a `DataCollatorForLanguageModeling` → train with `Trainer`/`TrainingArguments` (epochs, batch size).
- **Embeddings / latent space**: the bottleneck activation vector of a token/sequence — a numeric fingerprint of "where" the network places that input in its learned semantic space.
  - When to use: to inspect what a model has learned, build similarity search, or feed a downstream classifier.
  - How: extract via a `feature-extraction` pipeline; visualize by projecting to 2D with t-SNE.

## Key Concepts
- **Transformer**: encoder-decoder architecture with multiple attention heads; separates generic pre-training from task-specific training (Vaswani, 2017).
- **Tokenizer (BPE)**: converts text to numeric token IDs the network can consume; vocabulary size and special tokens (`<mask>`, `<unk>`, etc.) are configured up front.
- **Masked Language Modeling (MLM)**: self-supervised training objective — hide a token, train the network to predict it.
- **Embedding vector**: the neuron activation values at a given layer (often the bottleneck); size (768 for BERT-scale, ~16,000 for GPT-4-class models) bounds how much information the model can represent.
- **Fine-tuning classifier (k-NN)**: a lightweight downstream model trained on embeddings to perform a specific task (e.g., classify declarations as data vs. function).
- **Temperature / sampling**: controls how much the generation samples from lower-probability tokens ("creativity" vs. exactness).
- **Out-of-vocabulary (OOV) tokens**: tokens unseen during pre-training, marked `<unk>`; a major source of poor fine-tuning results when domains mismatch.

## Mental Models
- Think of pre-training as "teaching the language" and fine-tuning as "teaching the job" — reusing the expensive generic step across many cheap specific steps is why HuggingFace-style model reuse exploded generative AI adoption.
- Use the embedding space as a diagnostic tool: if a t-SNE plot shows tight clusters with a few outliers, the model has learned real distinctions; if it's a formless blob, the pre-training data or model capacity is insufficient.

## Anti-patterns
- **Assuming a small pre-trained vocabulary generalizes across domains**: a model pre-trained only on C declarations will have high OOV rates on Python — check domain match before reusing a model.
- **Judging model quality only by loss curves, not by inspecting embeddings/predictions on real examples** — the pipeline can appear to converge while nonetheless being incorrect for the target task.

## Worked Example
Book walks through training a toy transformer ("decBERTa") to complete C variable declarations:
1. Corpus: WolfSSL C declarations (`int x = 1;`, `void place(char* start)` etc.)
2. Tokenizer: `ByteLevelBPETokenizer`, vocab_size=5000, min_frequency=2, special tokens `<s>`,`<pad>`,`</s>`,`<unk>`,`<mask>`.
3. Model: `RobertaConfig` with vocab_size=5000, max_position_embeddings=150, 12 attention heads, 6 hidden layers.
4. Training: `DataCollatorForLanguageModeling` (mlm=True, mlm_probability=0.15) + `Trainer` (50 epochs, batch size 256).
5. Inference: `fill-mask` pipeline on `int i = <mask>;` → top prediction `"0"` (score 0.934), confirming the model learned realistic C idioms.
6. Fine-tuning: extract embeddings for each declaration line, label as data (0) vs. function (2), train a 3-NN classifier → 99.7% accuracy on held-out lines, because pre-training + fine-tuning on the *same* domain avoids OOV problems.

## Key Takeaways
1. Generative AI software always rests on two training stages — pre-training (generic, data-hungry, self-supervised) and task-specific training (fine-tuning or few/zero-shot prompting); know which stage you're skipping when reusing a foundation model.
2. Embedding vector size bounds representational capacity but also cost — bigger isn't free; match it to the diversity of your data.
3. Model quality on a fine-tuned task depends heavily on whether the pre-training domain matches the deployment domain — mismatch shows up as OOV tokens and unreliable embeddings.
4. Reuse (HuggingFace-style model hubs) is what makes "engineer, don't hack together" generative AI software economically feasible today — this is why the book insists on professional software engineering practice around these models (see later chapters).

## Connects To
- **Ch2**: instruct models are the pre-trained model from this chapter wrapped in an instruction-following pipeline (zero/few-shot, chain-of-thought).
- **Ch3**: MLOps/AI Engineering processes formalize how pre-training + fine-tuning fit into a software delivery pipeline.
- **Embeddings**: reused throughout the book for RAG/vector databases (Ch7) and portability discussions (Ch6).
