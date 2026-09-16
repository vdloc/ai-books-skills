# Chapter 6: Building AI Workflows with LangChain.js

## Core Idea
LangChain.js provides composable "runnables" (LCEL) for chaining prompt/LLM/parsing steps, document ingestion + vector-store retrieval for grounding, memory components for multi-turn context, and ReAct agents that let an LLM autonomously choose and call tools in a reason→act→observe loop — all integrable with the Vercel AI SDK's streaming UI.

## Frameworks Introduced
- **Runnable chain composition (LCEL)**: any unit of work (`RunnableLambda`, a prompt template, a model, an output parser) implements a common invoke/batch/stream interface and is composed via `.pipe()` or `RunnableSequence.from([...])`.
  - When to use: whenever a workflow needs more than one processing step before/after the LLM call (preprocessing, prompt templating, postparsing).
  - How: order matters — each step's output must match the next step's expected input shape; chain: transform → transform → prompt → model → output parser.
- **ReAct agent loop**: Input → agent decision-making → (call tool → observe result) repeated → final response once end-goal criteria are met.
  - When to use: multistep tasks requiring external data/tools chosen dynamically by the model, not hardcoded by the developer (cf. Ch4's simpler, developer-defined tool calling).
  - How: `createReactAgent({ llm, tools, prompt })` — give it a tool list, a model, and a system prompt; it decides which tools to call and when.
- **Retrieval chain pattern**: text splitter → embeddings → vector store → retriever → `RunnableSequence` combining `{ context: retriever.pipe(formatDocumentsAsString), question: RunnablePassthrough() }` → prompt → model → parser.
  - When to use: grounding responses in a document corpus (the book's precursor to full RAG).
  - How: always pair the query-time embedding model with the one used to embed stored documents (echoes Ch5).

## Key Concepts
- **Runnable**: a unit of work that can be invoked, batched, or streamed; the base abstraction all LangChain chain components implement.
- **`RunnableLambda`**: wraps a plain function as a runnable so it can be piped into a chain.
- **`ChatPromptTemplate` / `FewShotPromptTemplate`**: factory-method-pattern classes that encapsulate prompt construction with placeholders (`{vowelCount}`) and, for few-shot, `examples` + `examplePrompt` + `prefix`/`suffix`/`inputVariables`.
- **Text splitters**: transform long documents into model-context-sized chunks; types include Recursive (recommended default), HTML, Markdown, Code, Token, Character — each differs in what it splits on and whether it attaches metadata.
- **`RecursiveCharacterTextSplitter`**: splits on a cascading list of separators (newlines → smaller units) with configurable `chunkSize`/`chunkOverlap` to preserve semantic continuity across chunk boundaries.
- **Vector store**: specialized DB for high-dimensional embedding vectors enabling similarity search (`MemoryVectorStore` for prototyping; Supabase/Pinecone/Chroma/etc. for production).
- **Retriever**: `vectorStore.asRetriever()` — wraps a vector store as a composable chain component for semantic document lookup.
- **`ConversationChain` + `ChatMessageHistory` + `BufferMemory`**: LangChain's structured approach to multi-turn memory — retrieve past messages → convert to `ChatMessageHistory` → wrap in a `ConversationChain` with the model.
- **Standalone question transformation**: an intermediate chain step that rewrites a context-dependent user question into a self-contained one before retrieval, improving retrieval accuracy.

## Mental Models
- Think of LangChain vs. the Vercel AI SDK's tool calling (Ch4) as depth vs. simplicity: Vercel's tool calling is direct and lightweight; LangChain agents add dynamic multi-step reasoning (ReAct), built-in tool ecosystems, and composable chains — reach for LangChain when the task needs iterative reasoning across many tools, not a single fixed lookup.
- Treat "argument matching" between chain steps as a first-class design constraint — a chain is only as correct as its weakest input/output contract between consecutive runnables; mismatches surface as opaque template errors (e.g., "Missing value for input context").
- Context-window budget is a hard constraint on agent tool count: every tool's name/description/args consumes prompt tokens — don't register more tools than the model's context can comfortably hold context for.

## Anti-patterns
- **Sharing one memory/history instance across multiple chains**: causes cross-contamination of conversation context between unrelated interactions — always use a dedicated memory instance per chain/session.
- **Skipping the standalone-question rewrite step before retrieval**: ambiguous, context-dependent user queries (e.g., "what about that?") degrade retrieval quality — rewrite to a self-contained question first.
- **Registering too many tools on one agent**: each tool's name/description/params costs context tokens; an oversized toolset risks hitting context limits and increases latency/propagation delay through the reasoning chain.
- **Mismatching embeddings generator to the target model**: e.g., embedding with `OpenAIEmbeddings` for an OpenAI model but querying against a store embedded with a different provider's embeddings — comparisons become meaningless.

## Code Examples
```js
// Runnable chain via .pipe() — transform, count, template, model, parse
const chain = RunnableLambda.from(toUpperCase)
  .pipe(RunnableLambda.from(vowelCountFunction))
  .pipe(prompt)
  .pipe(model)
  .pipe(new StringOutputParser());
await chain.invoke({ text: "hello world" }).then(console.log);
```
```js
// Document splitting + vector store + retrieval chain
const splitter = new RecursiveCharacterTextSplitter({ chunkSize: 100, chunkOverlap: 20 });
const output = await splitter.createDocuments([text]);

const vectorStore = await MemoryVectorStore.fromTexts(
  ["Hello world", "Bye bye", "hello nice world"],
  [{ id: 2 }, { id: 1 }, { id: 3 }],
  new OpenAIEmbeddings({ apiKey })
);

const retriever = vectorStore.asRetriever();
const chain = RunnableSequence.from([
  { context: retriever.pipe(formatDocumentsAsString), question: new RunnablePassthrough() },
  prompt, model, new StringOutputParser(),
]);
const answer = await chain.invoke({ question: "What is artificial intelligence?" });
```
```js
// ReAct agent with a Wikipedia tool
const tools = [new WikipediaQueryRun({ topKResults: 3, maxDocContentLength: 4000 })];
const prompt = ChatPromptTemplate.fromMessages([
  ['system', AGENT_SYSTEM_TEMPLATE],
  ['human', '{input}'],
  new MessagesPlaceholder('agent_scratchpad'),
]);
const agent = createReactAgent({ llm: model, tools, prompt });
const output = await agent.invoke({ messages: [new HumanMessage("Everest")] });
```
```js
// Conversational memory
const memory = new BufferMemory({ chatHistory: new ChatMessageHistory(pastMessages) });
const res1 = await chain.invoke({ input: "Can you give me an example of AI?" });
const res2 = await chain.invoke({ input: "What did I just ask you?" }); // remembers res1's context
```

## Reference Tables
| Text splitter | Splits on | Adds metadata | Use case |
|---|---|---|---|
| Recursive | Cascading characters | Yes | Recommended default — preserves semantic structure |
| HTML | HTML tags | Yes | HTML documents |
| Markdown | MD syntax | Yes | Markdown docs |
| Code | Language-specific syntax | Yes | Source code |
| Token | Token boundaries | No | Token-precise chunking |
| Character | One user-defined char | No | Simple/naive splitting |

| LangChain module | Purpose |
|---|---|
| `langchain/prompts` | Prompt templates + input variables |
| `langchain/agents` | Autonomous tool-using agents |
| `langchain/chains` | Combine components into pipelines |
| `langchain/vectorstores` | Vector DB integrations |
| `langchain/document_loaders` | Load documents from sources |
| `langchain/output_parsers` | Transform LLM output into structured formats |

## Worked Example
The vacation-planning agent trace: user asks to "Plan a seven-day vacation to Japan for a family of four with a budget of $5000." The ReAct agent iterates — decision → call a web-search tool for Japan travel/family activities/costs → observe results → decision → call a flight-comparison tool within budget → observe → decision → call a hotel-booking tool, then a currency-converter, then a weather-forecast tool → after enough iterations, compile a full 7-day itinerary (flights, hotels, daily activities, budget breakdown) as the final response. Each tool call is a distinct reason→act→observe cycle, fully abstracted from the end user.

## Key Takeaways
1. Use LCEL runnable chains (`.pipe()`/`RunnableSequence`) whenever a workflow needs multiple processing steps around the LLM call — order and input/output contract matching are the main failure points.
2. For retrieval-grounded chat, always pair the same embedding model between document ingestion and query time, and consider rewriting ambiguous questions into standalone form before retrieval.
3. Choose LangChain ReAct agents over simple Vercel AI SDK tool calling when the task needs dynamic, iterative, multi-tool reasoning rather than a single fixed lookup.
4. Give each conversation/session its own dedicated memory instance (`ChatMessageHistory` + `ConversationChain`) — never share memory across unrelated chains.
5. Watch two agent cost factors: context-window pressure from too many registered tools, and latency from propagation through multiple chained components — both are addressed further in Ch8.

## Connects To
- **Ch4**: LangChain agents explicitly extend and outclass the simpler Vercel AI SDK tool-calling pattern introduced there.
- **Ch5**: `FewShotPromptTemplate` operationalizes the few-shot technique from the previous chapter; retrieval here builds directly on Ch5's embeddings.
- **Ch7**: this chapter's retrieval chain is the direct foundation for the full document summarization/RAG system built next.
- **Ch8**: latency/propagation and testing concerns raised here are addressed with concrete techniques.
