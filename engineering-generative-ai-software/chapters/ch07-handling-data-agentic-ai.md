# Chapter 7: Handling Data for Generative AI Systems — Agentic AI

## Core Idea
Agents are software wrappers around a model that maintain conversation state and pursue a goal; on their own they hallucinate, drift off-task, and don't know when to stop, so production-grade agentic AI requires grounding them in external data (files/databases/vector stores) and giving them tools (compilers, validators) to check their own output.

## Frameworks Introduced
- **Agent class pattern**: encapsulate `server_address`, `model_name`, a system-role string, and a running `messages` list behind a `get_response(prompt)` method; multiple agents (e.g., a "programmer" and a "designer") can converse by feeding each other's responses back in as prompts.
  - When to use: whenever you want a model to specialize into a role (via a fixed system message) and hold multi-turn state cleanly.
  - How: `AgentAI.__init__(server_address, model_name, my_role)` seeds `messages` with a system role; `get_response` appends user turn, POSTs to the model server (with retry/timeout handling), appends and returns the assistant turn.
- **Multi-agent conversation loop**: chain two (or more) agents' outputs into each other for N iterations to simulate collaborative work (e.g., programmer ↔ designer improving code).
  - Key finding: agents converge to a solution within ~10-20 iterations or never at all; letting them run longer just produces off-topic chatter or mutual praise loops, independent of model — you need an explicit stop condition.
  - Model-choice finding: mixing two different model families (e.g., Gemma-3 + Llama 3.3) in different roles outperforms using one model as both roles; reasoning models (DeepSeek-R1, GPT-o4-mini) are better as the "designer/critic" role, and plain LLMs are faster/adequate as the "generator" role.
- **RAG-grounded agent (ChromaDB)**: retrieve relevant documents/code snippets from a vector database and inject them into the prompt before calling the model — converts a zero-shot request into an implicit few/one-shot request, reducing hallucination.
  - How: `collection.query(query_texts=[prompt], n_results=k)` → concatenate retrieved snippets into the prompt → send enhanced prompt to the agent.
- **Tool-equipped agent (compiler-in-the-loop)**: extract code from the model's markdown response, save it, compile with GCC, and if compilation fails, feed the compiler error back to the model in a retry loop (bounded by `tries`) until it compiles or attempts run out.
  - When to use: any code-generation agent where correctness is checkable by an external tool (compiler, linter, test suite) — dramatically improves output quality even with a small/non-reasoning model (Llama 3.3 alone, tool-equipped, beat un-equipped larger setups).

## Key Concepts
- **Agent**: a program tasked with executing a workflow to answer one specific question/goal, using generative AI within that workflow.
- **AI-assisted Software Engineering 3.0**: the author's term for one engineer overseeing multiple specialized agents (programmer/tester/designer) rather than doing all tasks manually.
- **Data storage options for agents** (choose by scale/structure need): CSV/pandas (small, tabular), JSON (object-shaped, verbose), edge/in-memory DB (SQLite — fast, embeddable, good default), SQL (PostgreSQL/MySQL/SQL Server/Oracle — structured, scalable, poor fit for unstructured code/text), Apache Spark (high-performance, large-scale SQL+in-memory), NoSQL (MongoDB/ElasticSearch/Cassandra/DynamoDB — document-shaped, good for code/images), **vector/embedding database** (ChromaDB and similar — semantic similarity search via embeddings, not exact match).
- **Embedding database mechanics**: a language model embeds both the stored documents and the incoming query into the same latent space; nearest-neighbor search retrieves semantically similar documents even without exact keyword overlap (though the book notes results are always returned — you must also filter, e.g., `where_document`, to confirm relevance).

## Mental Models
- Treat unattended multi-agent conversations as bounded-attempt processes, not open-ended ones — decide your iteration cap and success condition (e.g., "compiles successfully" or "no code changes for 2 rounds") *before* running, because agents left alone will not self-terminate sensibly.
- When picking a data store for an agent, ask "structured rows, verbose objects, need for scale, or need for semantic similarity?" — that question alone routes you to SQL, JSON, Spark/NoSQL, or a vector DB respectively.

## Anti-patterns
- **Letting two agents converse for thousands of iterations unattended, expecting convergence**: observed outcomes are convergence, infinite mutual praise, mutual insults, or degeneration to `*` — none of which happens reliably; always bound iterations and define a stop/check condition.
- **Using a single model in two different roles instead of two different models**: measurably worse results than mixing model families for programmer/designer-style multi-agent setups.
- **Skipping the embedding-database relevance filter**: an embedding query *always* returns a "closest" result even when nothing relevant exists — treat unfiltered vector search results as untrusted until a distance/where-clause threshold confirms genuine relevance.

## Worked Example
Compiler-in-the-loop agent workflow (Listings 7.15-7.17, condensed):
1. Agent responds to "Write a linked list implementation in C" (optionally enriched with ChromaDB-retrieved snippets for bubble sort, factorial, linked-list `struct Node`).
2. `__extract_c_code` pulls the code block out of the markdown response via regex.
3. `__compile_code` writes it to a temp `.c` file and runs `gcc -w code_temp.c -o a.out -lm`, capturing stderr.
4. If `"error"` appears in stderr, the workflow loops: feed the compiler error back to the agent (`__solve_problem`), re-extract, re-compile, up to a fixed `tries` count.
5. Result: even the non-reasoning Llama 3.3 model, equipped with this compiler feedback loop, produced meaningfully better compiling code than an unequipped setup — demonstrating that tool access can substitute for a larger/reasoning model.

## Key Takeaways
1. Wrap models in a small stateful `AgentAI`-style class (system role + message history + retry-safe HTTP call) rather than calling the model ad hoc — it's the reusable unit for all agentic patterns in this chapter.
2. Multi-agent conversations need an explicit stop condition and iteration cap — they do not self-terminate sensibly, converging in ~10-20 turns or never.
3. Mix model families across agent roles (reasoning model as critic/designer, plain LLM as generator) rather than reusing one model in multiple roles.
4. Ground agents in a vector database (ChromaDB) to reduce hallucination by converting zero-shot requests into implicit few-shot ones — but always apply a relevance filter since nearest-neighbor search always returns *something*.
5. Giving an agent access to a validating tool (compiler, test runner) and a retry loop on failure improves output quality more reliably than upgrading to a bigger model.

## Connects To
- **Ch5**: this chapter's ChromaDB pattern is the concrete implementation of the RAG architecture introduced in section 5.4.2.
- **Ch6**: the compile-and-retry loop is a practical application of the metamorphic/oracle testing ideas — the compiler *is* the oracle here.
- **Ch10**: agent coordination and "when to stop" are flagged there as open research problems this chapter's experiments foreshadow.
