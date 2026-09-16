# Chapter 1: Introduction to Retrieval-Augmented Generation (RAG)

## Core Idea
LLMs are "blind" to private, up-to-date, or niche data because their knowledge is frozen at training time; RAG fixes this by retrieving relevant facts at query time and grounding generation in them, rather than retraining or fine-tuning the model.

## Frameworks Introduced
- **The R-G split (Retrieval + Generation)**: a RAG query always has two steps — retrieve relevant facts, then generate an answer grounded in only those facts.
  - When to use: any time an LLM needs access to private, domain-specific, or frequently-changing data.
  - How: embed the query → similarity search a vector DB (or hybrid/lexical search) → pass retrieved chunks as `<context>` in the prompt → LLM answers using only that context.
- **Ingestion flow vs. Query flow**: the two structural halves of any RAG stack.
  - Ingestion: extract → chunk → embed → store in vector DB (+ raw text alongside vectors).
  - Query: embed query → retrieve (semantic and/or lexical) → optionally hybrid search / rerank → generate → apply guardrails (hallucination check, bias/toxicity filter) → return response with citations.
- **The closed-book vs. open-book test analogy**: pure LLM = closed-book (relies only on parametric/weight-encoded knowledge); RAG = open-book (consults retrieved reference material at answer time).
- **The Borg effect (naming the fine-tuning access-control failure)**: fine-tuning assimilates all training data into one set of model weights, so you cannot selectively expose "CEO-only" vs. "all-employee" knowledge — the model doesn't know which employee is asking. RAG solves this because retrieval can filter by metadata/permissions per query.

## Key Concepts
- **Hallucination**: LLM-generated content unsupported by its own knowledge or the provided context.
- **Semantic search**: retrieval via embedding similarity in vector space.
- **Lexical search**: retrieval via matching in written/token form (e.g., BM25-style).
- **Hybrid search**: combining semantic + lexical search for better recall/precision.
- **Reranking**: a second-pass relevance scoring step applied after initial retrieval.
- **Dataset vs. corpus vs. index**: dataset = raw source files; corpus = cleaned/organized body of content; index = the optimized data structure built from the corpus for fast retrieval.
- **Parametric knowledge**: information encoded in an LLM's weights from training, as opposed to knowledge supplied externally at inference time.
- **Agentic RAG**: RAG where autonomous agents can iteratively re-retrieve, call external tools, and plan multi-step reasoning instead of a single retrieve-then-generate pass.

## Mental Models
- Think of an LLM as a brilliant closed-book test-taker: broad general knowledge, zero access to your company's specific documents. RAG turns the exam open-book.
- Think of RAG vs. fine-tuning as "swap the reference books" vs. "retrain the student." Swapping books (RAG) is instant and per-user filterable; retraining the student (fine-tuning) is slow, expensive, and blends everything into one undifferentiated skill set.
- Use RAG (not "stuff everything into a huge context window") whenever the corpus is large, multi-source, or requires per-user access control — raw context-stuffing doesn't scale on cost, latency, or document-selection grounds.

## Anti-patterns
- **"Chat with PDF" at enterprise scale**: dumping full document text into a giant context window. Fails on cost (paying to process irrelevant tokens), latency, and document selection (someone still has to pick which of thousands of documents to feed in — which is retrieval in disguise).
- **Fine-tuning for freshness or access control**: fine-tuning bakes all training data into one set of weights, so it can't cleanly separate confidential departmental data (the Borg effect), and it must be redone (expensive, GPU-hours) every time the underlying data changes.
- **Scaling training data instead of retrieving**: the world's data is too vast, too private, and too dynamic to ever fully fit inside a training run — model will always be stale.

## Code Examples

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import LanceDB
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

pdf_url = "https://www.adobe.com/be_en/active-use/pdf/Alice_in_Wonderland.pdf"
loader = PyPDFLoader(pdf_url)
pages = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
)
chunks = text_splitter.split_documents(pages)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = LanceDB.from_documents(chunks, embeddings)

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the following context:

{context}

Question: {question}"""
)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```
- **What it demonstrates**: the minimal end-to-end RAG pipeline — ingest (load → chunk → embed → store) then query (retrieve → format → prompt → generate) — using LangChain, LanceDB, and OpenAI.

## Worked Example
The book grounds its first example in *Alice's Adventures in Wonderland*. Ingestion: download the PDF, split into 1000-char chunks (200-char overlap), embed with `text-embedding-3-small`, store in LanceDB. Query: `"Describe the Mad Hatter's tea party."` The retriever pulls the top-3 chunks, `format_docs` joins them, the prompt template restricts the LLM (`gpt-4o-mini`) to answer only from that context, and the chain returns a grounded summary of the tea party scene — illustrating that even this "toy" pipeline already produces an answer traceable to specific retrieved text, not the model's parametric memory.

## Key Takeaways
1. RAG's core value is grounding — it trades hallucination-prone parametric recall for retrieval-anchored, verifiable answers.
2. Don't reach for context-stuffing or fine-tuning by default: use fine-tuning only when you already have deep ML expertise, a large clean dataset, and a static domain that doesn't need per-user access segmentation.
3. Access control is a first-class RAG benefit, not an afterthought — implement it via metadata filtering at retrieval time.
4. RAG scales because search infrastructure scales; LLM self-attention does not (quadratic cost with sequence length), so pushing selection work into retrieval is what makes enterprise-scale grounding tractable.
5. Citations aren't cosmetic — they're what lets users (and evaluators) distinguish a hallucination from a retrieval failure (bad source data).
6. Production RAG requires DevOps discipline beyond the pipeline itself: automated data refresh, automated evaluation in CI/CD, prompt/model versioning, observability, and security controls.

## Connects To
- **Ch2**: builds the concrete base RAG stack (parsing, chunking, embeddings, vector DBs, LLMs) referenced here only at a high level.
- **Ch3**: covers hybrid search, reranking, and guardrails mentioned here as "advanced retrieval."
- **Ch7**: expands agentic RAG (iterative retrieval, tool use) introduced here.
- **Ch8**: expands multimodal RAG (tables, images) introduced here.
- **Ch9**: expands RAG with knowledge graphs / GraphRAG introduced here.
