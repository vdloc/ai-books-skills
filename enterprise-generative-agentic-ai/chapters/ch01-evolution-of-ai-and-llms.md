# Chapter 1: Evolution of AI and Large Language Models

## Core Idea
Modern generative AI is not a sudden leap but the culmination of decades of AI research — from symbolic logic and expert systems, through the neural-network "connectionist" revival, to the 2017 Transformer architecture that made today's LLMs possible.

## Frameworks Introduced
- **GOFAI (Good Old-Fashioned AI) vs. Connectionism**: two competing AI paradigms.
  - When to use: understand *why* modern LLMs learn from data instead of hand-coded rules — GOFAI (expert systems, semantic nets) required all knowledge to be manually encoded; connectionism (neural nets) learns patterns from examples.
  - How: GOFAI = knowledge base + inference engine (forward/backward chaining); connectionism = layered networks trained via backpropagation.
- **The Turing Test**: operational definition of machine intelligence — a human judge can't distinguish a machine's text responses from a human's.
  - When to use: as a conceptual benchmark, not a modern engineering test — most contemporary LLM eval frameworks (Ch 6) supersede it.
- **Transformer / Attention Mechanism** ("Attention Is All You Need," Google 2017): replaces sequential RNN encoding with parallel, multi-head attention that weighs token importance regardless of position.
  - When to use: whenever you need to explain *why* LLMs scale better than Seq2Seq/RNN — attention removes the fixed-size encoder-vector bottleneck and enables parallel token processing.

## Key Concepts
- **Turing machine**: conceptual model executing programmable tasks without changing physical machinery; theoretical basis for all computers.
- **Perceptron**: single-layer neural net, only classifies linearly separable data — its limits caused the first "AI winter."
- **Hidden layers / backpropagation**: intermediate network layers plus the training algorithm that propagates output error backward to adjust weights — key to multilayer perceptrons (deep learning).
- **Expert system**: domain knowledge base + inference engine emulating a human specialist's decisions (e.g., MYCIN, XCON/R1).
- **Semantic network**: graph of concepts (nodes) and relations (edges) enabling non-keyword understanding.
- **Commonsense knowledge problem**: challenge of encoding the implicit, everyday knowledge humans acquire naturally (Cyc project's core goal).
- **Seq2Seq bottleneck**: an RNN encoder compresses an entire input sequence into one fixed-size vector, degrading quality on long inputs — the problem Transformers solve.
- **Parameters**: the weights/biases adjusted during training to minimize prediction error.

## Mental Models
- Think of AI history as **alternating "boom" and "winter" cycles**: symbolic AI (1950s-60s) → first winter (perceptron limits) → expert systems boom (1980s) → second slowdown (cost of manual knowledge encoding) → connectionist/deep-learning boom (2010s-present, fueled by big data + compute + Transformers).
- Use **"knowledge encoded vs. knowledge learned"** as the axis to classify any AI system you encounter: is behavior hard-coded (rules/ontologies) or learned from data (weights)? This tells you how it will fail (rules fail on unseen cases; learned models fail on out-of-distribution data / hallucinate).

## Anti-patterns
- **Assuming bigger training data alone solves knowledge gaps**: the book stresses this doesn't scale to private/proprietary/constantly-changing data — this is the entire motivation for RAG and agentic patterns covered later in the book.
- **Treating the Turing test as a production quality bar**: it measures indistinguishability, not correctness, safety, or reliability.

## Key Takeaways
1. LLMs are blind to private/internal/recent data no matter how large their training corpus — this is *the* structural reason enterprises need RAG/agentic augmentation (sets up Ch 2+).
2. The 2017 Transformer's parallel attention mechanism, not just more data, is what unlocked GPT-scale models.
3. History alternates between rule-based/symbolic and data-driven/connectionist approaches — expect the same oscillation in future paradigms (useful for evaluating hype cycles).
4. Backpropagation + hidden layers is the mechanical reason today's networks can learn nonlinear, complex patterns that single-layer perceptrons could not.

## Connects To
- **Ch 2**: applies this LLM foundation to concrete business value/use cases.
- **Ch 4**: agentic AI is presented as the next step beyond generation — giving these models the ability to *act*, not just answer.
- **RAG (external concept)**: the "LLMs can't know your private data" problem this chapter raises is the direct justification for retrieval-augmented generation as an architecture.
