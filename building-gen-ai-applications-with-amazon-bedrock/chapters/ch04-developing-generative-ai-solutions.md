# Chapter 4: Developing Generative AI Solutions

## Core Idea
The three core AI-engineering techniques — prompt engineering, RAG, and fine-tuning — sit on an escalating cost/complexity ladder and are complementary, not competing: prompt engineering shapes behavior with zero training cost, RAG grounds responses in live/proprietary data without retraining, and fine-tuning is reserved for teaching a model a new skill or style that prompting/retrieval can't achieve.

## Frameworks Introduced
- **Core Prompt Design Patterns**: Zero-shot / Few-shot / Chain-of-Thought, plus the system-prompt vs user-prompt distinction.
  - When to use: Zero-shot for tasks the model already generalizes well to (e.g. sentiment classification with a clear instruction). Few-shot when the task has a specific output format/style the model needs examples to match. Chain-of-Thought when the task requires multi-step reasoning the model should make explicit before answering.
  - How: Zero-shot = instruction + input only, relying on pretrained generalization. Few-shot = instruction + several input-output example pairs before the real input. Chain-of-Thought = explicitly instruct the model to reason step-by-step (or provide exemplars that reason step-by-step) before producing the final answer. System prompts (especially with Claude) set persistent behavior/role/constraints separate from the per-turn user prompt — use the system prompt for standing instructions (tone, format, safety constraints) and the user prompt for the specific request.
- **Components of a RAG Pipeline** (the book's 5-ish stage pipeline): the standard architecture for grounding generation in external data.
  - When to use: any application needing responses grounded in proprietary/current data the base model wasn't trained on.
  - How: (1) Document Ingestion & Preprocessing — load from sources (S3, Confluence, SharePoint, Salesforce, web crawlers, Redshift/Glue for structured data), clean/extract text (AWS Textract for scanned/complex documents), then chunk; (2) Embedding — convert chunks to vectors (e.g. Titan Embeddings); (3) Vector storage/indexing — store in a vector store (OpenSearch Service Serverless/Managed, Aurora with pgvector); (4) Retrieval — semantic search against the query's embedding to fetch top-k relevant chunks; (5) Augmented Generation — inject retrieved chunks into the prompt context for the foundation model to generate a grounded answer.
- **Chunking Strategy Selection**: don't default to fixed-size chunking.
  - When to use: designing the ingestion stage of any RAG pipeline.
  - How: fixed-size (simplest, may split mid-thought) → recursive (splits on semantic separators like paragraphs/sentences) → structural (uses document structure — headings, sections) → semantic (uses embeddings to find natural topic breakpoints) → agentic (an LLM decides optimal chunk boundaries and size, most expensive but highest quality). Chunks too large dilute relevance with noise; chunks too small lose necessary context — tune against your embedding model's context limit (the book notes some embedding models cap at 512 tokens) and retrieval precision needs.

## Key Concepts
- **RAG (Retrieval-Augmented Generation)**: grounding generation by retrieving relevant external documents at query time and injecting them into the model's context, instead of relying only on parametric (trained-in) knowledge.
- **Chunking**: splitting large documents into smaller segments before embedding, to fit context limits and improve retrieval precision.
- **Provisioned Throughput**: a Bedrock purchase option (with a commitment period) that reserves dedicated model capacity — contrasted here with on-demand, no-commitment inference, chosen when workloads need predictable low latency at high volume.
- **Custom model (fine-tuning) job lifecycle**: Bedrock fine-tuning jobs move through states — `InProgress` → `Completed` or `Failed` — configured via a training-data S3 URI, hyperparameters, and a base model ID.
- **Instruction Tuning**: a fine-tuning approach specifically aimed at improving a model's ability to follow instructions/format, distinct from teaching new domain knowledge.
- **Domain-Specific Fine-Tuning**: fine-tuning to adapt a model's style/knowledge to a narrow domain (e.g. legal, medical) where prompting alone can't reliably encode enough specialized behavior.

## Mental Models
- Use the **"prompt engineering → RAG → fine-tuning" escalation** as the default evaluation order for any new requirement: try to solve it with prompt design first (cheapest, fastest iteration), add RAG if the gap is missing/outdated knowledge, and only fine-tune if the gap is style/format/skill that neither prompting nor retrieval fixes.
- Treat **system prompts as the place for standing constraints**, and user prompts as the place for the variable request — this separation (emphasized for Claude specifically) keeps prompts maintainable as an application grows past single-shot use.
- Think of **RAG accuracy/faithfulness as a retrieval problem first, generation problem second** — most RAG quality issues trace back to poor chunking or retrieval (wrong or incomplete chunks reaching the model), not the generation step itself.

## Anti-patterns
- **Reaching for fine-tuning to fix a knowledge-freshness problem**: if the issue is the model not knowing about recent/proprietary information, RAG is the fix — fine-tuning bakes knowledge in at training time and still needs constant retraining to stay current, unlike RAG's retrieval-time freshness.
- **Using naive fixed-size chunking for structured or highly technical documents**: ignores document boundaries (sections, tables), degrading retrieval precision; the book explicitly recommends matching chunking strategy to document nature.
- **Skipping Chain-of-Thought prompting on multi-step reasoning tasks**: asking for a direct answer to a task that needs intermediate reasoning steps increases error rates versus explicitly prompting the model to reason first.
- **Ignoring prompt injection risk in RAG-retrieved content**: the book flags that "complete prevention of prompt injection" from untrusted retrieved documents is not fully achievable — treat retrieved content as a trust boundary requiring guardrails (Ch1's Bedrock Guardrails), not implicit trust.

## Code Examples
```python
# Basic zero-shot prompt pattern
zero_shot_prompt = """
Analyze the following customer feedback and extract the key sentiment themes:
Customer Feedback: {feedback_text}
"""
```
- **What it demonstrates**: the book's canonical zero-shot pattern — instruction plus a single input variable, no examples, relying entirely on the model's pretrained understanding of "sentiment" and "themes."

## Reference Tables
**RAG Pipeline Stages**
| Stage | Purpose | Key AWS Components |
|---|---|---|
| Document Ingestion & Preprocessing | Load + clean + chunk source data | S3, Confluence/SharePoint/Salesforce connectors, AWS Textract |
| Embedding | Convert chunks to vectors | Amazon Titan Embeddings |
| Vector Storage/Indexing | Store embeddings for search | Amazon OpenSearch Service (Serverless/Managed), Aurora (pgvector) |
| Retrieval | Semantic search for relevant chunks | Bedrock Knowledge Bases |
| Augmented Generation | Inject retrieved chunks into prompt, generate | Bedrock foundation model (InvokeModel) |

**Chunking Strategies** (increasing sophistication/cost): Fixed-size → Recursive (semantic separators) → Structural (document structure) → Semantic (embedding-based breakpoints) → Agentic (LLM-determined chunk boundaries).

## Worked Example
The book's fine-tuning worked example walks a full Bedrock custom-model job: configure the Bedrock client, prepare training data as an S3 URI (one JSON example per line), set hyperparameters, call "Create fine-tuning job" with a base model ID, then poll job status through `InProgress` → `Completed`/`Failed`; once completed, the custom model must have Provisioned Throughput purchased (no on-demand option for custom models at the time of writing) before it can serve inference traffic — a cost implication the book flags as a key factor in the fine-tune-vs-RAG decision.

## Key Takeaways
1. Default to the prompt engineering → RAG → fine-tuning escalation order; don't fine-tune to solve a problem prompting or retrieval can fix more cheaply.
2. Match chunking strategy to document structure — fixed-size is the default fallback only, not the recommended starting point for structured/technical content.
3. Use system prompts for standing behavior/constraints and user prompts for the per-request content, especially with Claude models.
4. RAG quality problems are usually retrieval/chunking problems, not generation problems — debug the pipeline stage-by-stage from ingestion forward.
5. Custom (fine-tuned) Bedrock models require Provisioned Throughput to serve traffic — factor this fixed cost into the fine-tuning decision, not just training cost.
6. Treat retrieved RAG content as untrusted input for prompt-injection purposes — pair RAG with guardrails rather than assuming retrieved text is safe.

## Connects To
- **Ch3**: applies the sampling strategies and adaptation-method ladder from Ch3 directly to prompt design and fine-tuning decisions here.
- **Ch5**: extends the RAG pipeline introduced here into full Knowledge Bases integration with existing enterprise workflows.
- **Ch6**: Provisioned Throughput purchasing (introduced here for custom models) is revisited under scaling/cost-optimization.
