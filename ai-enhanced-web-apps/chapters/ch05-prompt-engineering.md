# Chapter 5: Prompt Engineering in Web Applications

## Core Idea
Prompt engineering is a training-free optimization lever available at three prompt types (basic text, messages, system) and three escalating techniques (few-shot, chain-of-thought, embeddings for retrieval) — each trades off effort against reliability, and none require fine-tuning.

## Frameworks Introduced
- **Prompt type hierarchy**: Basic text prompt (`prompt:` attribute, single-turn) → Messages prompt (`messages:` attribute; simple / compound-with-attachments / tool-call) → System prompt (`system:` attribute, sets persona/behavior for the whole session).
  - When to use: basic text for one-off queries; messages for multi-turn/RAG/tech-support agents; system for persona, compliance framing, or standing instructions.
  - How: prefer `messages` over bare `prompt` in production — it's strictly more flexible (supports history) at no extra cost.
- **Few-shot learning procedure** (5 steps): provide representative examples → design a prompt embedding them → run the model → review output → adjust prompt/examples.
  - When to use: adapting a model to a specific tone/format/domain without fine-tuning.
  - How: 2-3 diverse, representative examples beat many similar ones; end with an explicit directive ("Now respond to the following using the same tone").
- **Chain-of-thought (CoT) procedure** (5 steps): define prompt structure → provide a worked example → explicitly instruct step-by-step explanation → test with varied inputs → iterate/refine wording.
  - When to use: multi-step reasoning tasks (math, logic, planning) where showing work improves both correctness and auditability.
  - How: add an explicit instruction like "Solve step-by-step" — but always validate intermediate steps; CoT doesn't guarantee correctness.
- **Embeddings-based retrieval pipeline**: content → generate embeddings → store in a vector DB → query → generate query embedding (same model) → similarity search (cosine / Euclidean) → return ranked results.
  - When to use: semantic search, FAQ/knowledge-base lookup, and as the foundation for RAG (expanded in Ch7).
  - How: always embed queries with the *same* model used for stored content, or comparisons are invalid.

## Key Concepts
- **Zero-shot vs. few-shot vs. fine-tuning**: zero-shot relies purely on pretrained knowledge; few-shot adds inline examples at inference time; fine-tuning retrains model weights on a task-specific dataset (most accurate, most resource-intensive).
- **Embedding**: a numerical vector representation of text/data capturing semantic meaning, used for similarity search, classification, recommendations.
- **`embed` / `embedMany`**: Vercel AI SDK functions for generating one or many embeddings respectively, mirroring `generateText`'s ergonomics.
- **Cosine similarity / Euclidean distance**: two standard vector-similarity metrics — cosine measures angle (closer to 1 = more similar), Euclidean measures straight-line distance (shorter = more similar).
- **Tree of thoughts**: extends CoT from a single linear reasoning path to exploring multiple branching paths — better for tasks needing strategic lookahead.
- **Self-refine**: a feedback loop where the model critiques and improves its own prior output iteratively, no extra training needed.
- **LLM-as-a-judge**: using an LLM to evaluate another LLM's output quality — cheaper than human eval but can carry its own biases.

## Mental Models
- Treat prompt *type* selection as an architecture decision, not a style choice: single-turn tools use `prompt`, anything with history/context/tool-calls should use `messages` from the start.
- CoT and few-shot are complementary, not competing — combine a worked example (few-shot) with an explicit "explain step-by-step" directive (CoT) for the strongest effect.
- When ambiguous prompt wording ("list *some* languages") produces inconsistent output volume/scope, the fix is precision in the prompt itself ("list *5*"), not a different technique.

## Anti-patterns
- **Ambiguous quantifiers in prompts** ("some", "a few"): produces unpredictable output length/scope — always specify exact counts/constraints.
- **Trusting chain-of-thought output as correct because it "shows its work"**: models can produce plausible-looking but wrong intermediate steps (misunderstood problem, ambiguous phrasing, hallucinated facts, overfit to narrow examples) — always validate steps for tasks requiring precision.
- **Embedding a query with a different model than was used for stored content**: makes similarity comparisons meaningless — always match embedding models between index-time and query-time.
- **Skipping this chapter's advanced techniques (Tree of Thoughts, Self-Refine, LLM-as-judge) as "just theory"**: explicitly framed as optional for practitioners focused purely on shipping, but load-bearing once evaluation/quality rigor is needed (see Ch6 eval-heavy content).

## Code Examples
```js
// Basic text prompt vs. equivalent messages prompt
generateText({ prompt: 'Hello, what is your name?', model: model('models/gemini-2.0-flash') });
generateText({ messages: [{ role: 'user', content: 'Hello, what is your name?' }], model });
```
```js
// Few-shot prompt for a customer-support chatbot (persona + example interactions)
const system = `
You are a customer support chatbot. Adapt your tone and sentiment based on the
following example interactions for each supported use case:
**Use Case 1: Technical Support**
**User:** My internet connection is really slow. Can you help me?
**Chatbot:** I'm sorry to hear that... Can you provide your speed test results?
**Use Case 2: Billing Inquiry**
**User:** I was charged twice this month.
**Chatbot:** I understand how concerning that is. Let me check your account.
Now, respond to the following user inquiries using the appropriate tone and sentiment:
`;
```
```js
// embed / embedMany — single and batch embedding generation
import { embed, embedMany } from "ai";
import { createGoogleGenerativeAI } from "@ai-sdk/google";
const google = createGoogleGenerativeAI({ apiKey });

const { embedding } = await embed({
  model: google.textEmbeddingModel("text-embedding-004"),
  value: "The quick brown fox jumps over the lazy dog",
});

const { embeddings } = await embedMany({
  model: google.textEmbeddingModel("text-embedding-004"),
  values: ["sentence one", "sentence two", "sentence three"],
});
```

## Reference Tables
| Technique | Effort | Fixes | Limitation |
|---|---|---|---|
| Few-shot learning | Low (inline examples) | Format/tone/domain adaptation | Overfits to overly narrow examples |
| Chain-of-thought | Low (instruction wording) | Multi-step reasoning transparency | Doesn't guarantee step correctness |
| Embeddings + retrieval | Medium (DB + pipeline) | Semantic search, grounding (RAG) | Requires consistent embedding model |
| Tree of thoughts | High | Strategic/multistep planning tasks | More expensive (multiple paths) |
| Self-refine | Medium (feedback loop) | Iterative quality improvement | More LLM calls per output |
| LLM-as-a-judge | Medium | Scalable eval vs. human review | Judge model can carry its own bias |
| Fine-tuning | Highest | Deep domain accuracy | Resource/compute intensive |

## Worked Example
IT support knowledge base via embeddings: (1) offline, every FAQ Q/A pair is embedded with `google.textEmbeddingModel("text-embedding-004")` and stored in a vector-capable store (pgvector, Pinecone, Milvus, etc.); (2) at query time, the user's question is embedded with the *same* model; (3) cosine similarity ranks stored embeddings against the query embedding; (4) the top-ranked Q/A(s) are returned to the user as the answer/recommendation. This is the direct precursor to the RAG pattern the book expands in Chapter 7 (LangChain.js document summarization/RAG).

## Key Takeaways
1. Choose prompt type (`prompt` vs. `messages` vs. `system`) based on whether you need history, attachments, or standing persona/behavior — default to `messages` for anything beyond a single throwaway query.
2. Few-shot examples must be diverse and representative, not just plentiful — 2-3 well-chosen examples beat many narrow ones, and end with an explicit closing directive.
3. Chain-of-thought improves transparency and often accuracy but never guarantees correct intermediate steps — validate for precision-critical tasks.
4. Always embed queries with the exact model used to embed stored content; embeddings from different models are not comparable.
5. Advanced eval techniques (Tree of Thoughts, Self-Refine, LLM-as-a-judge) matter once you need to systematically improve or grade AI output quality — treat them as the next layer up from basic prompting.

## Connects To
- **Ch4**: reuses the `messages`/`system` prompt shapes already introduced for `generateText`/`streamUI`.
- **Ch6**: LangChain.js formalizes chaining prompts/tools into full workflows.
- **Ch7**: embeddings + similarity search here become the retrieval half of full RAG pipelines.
