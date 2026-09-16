# Chapter 1: Understanding AI Agents on AWS

## Core Idea
An agent differs from RAG by closing the loop from retrieval to action: RAG *retrieves and answers*, an agent *plans, acts, and reasons over the result* — and on AWS that reasoning loop is built from Bedrock (or SageMaker) as the "brain," tools/MCP for hands, and memory, orchestrated by a framework like Strands SDK or LangGraph.

## Frameworks Introduced
- **RAG pipeline (Retrieve → Augment → Generate)**: the base retrieval technique agents build on.
  - When to use: answering questions grounded in your own data, without needing the system to *act*.
  - How: ingest → chunk → embed → store in a vector DB (OpenSearch, Pinecone, FAISS, Chroma, pgvector) → retrieve top-k → optionally filter/rerank → generate.
- **Traditional RAG vs. Agentic RAG**: traditional RAG is a fixed one-shot pipeline (`query → retrieve → generate`); agentic RAG treats retrieval as just one tool an agent can call, refine, combine with other tools, or skip.
  - When to use agentic RAG: when the system needs to decide *whether/how many times* to retrieve, not just execute a fixed retrieval step.
- **Agent components**: Tools, Memory, Planning, LLM ("brain") — the four parts every agent architecture decomposes into.
  - How: Tools = external functions/APIs (Ch2); Memory = short/long-term state (Ch3); Planning = sequencing actions/tool choices; LLM = the reasoning engine consulted bidirectionally by the other three.
- **AWS Agentic Stack (3 tiers)**: Agentic applications (Amazon Q, zero-config) → Managed Solutions (Bedrock + AgentCore, moderate effort) → Do-it-yourself (SageMaker AI, full control).
  - When to use: pick the tier matching your control/effort trade-off — this book builds primarily on Bedrock (agent orchestration) and SageMaker AI (custom model training/fine-tuning), the "sweet spot" for developers.
- **LLM → Multi-agent maturity ladder**: LLM → LLM+RAG → Augmented LLM (tools+memory) → Agentic workflow (plan/execute/synthesize) → Autonomous agent (persistent, self-correcting) → Multi-agent collaboration.
  - When to use: diagnose where your current system sits on this ladder before deciding what to build next; the "Agent Evaluation / Security / Guardrails / Observability" concern grows in importance at every stage up the ladder.
- **MCP (Model Context Protocol)**: standardized client-server protocol (Host, Client, Server, Data sources) for connecting an agent to *tools/data*, developed by Anthropic.
  - When to use: whenever an agent needs to call external tools/data sources in a framework-agnostic, write-once-run-anywhere way.
- **A2A (Agent-to-Agent Protocol)**: open standard from Google for *agent-to-agent* discovery/communication (vs. MCP's agent-to-tool scope), using JSON "Agent Cards" to advertise capabilities.
  - When to use: when an orchestrator agent (e.g. a Travel Agent) needs to delegate to specialized agents built by different teams/vendors without knowing their internals.
  - How they relate: MCP and A2A are complementary, not competing — MCP = agent↔tool, A2A = agent↔agent.

## Key Concepts
- **Chunking**: splitting documents into smaller pieces to fit LLM context windows and reduce the "lost-in-the-middle" effect.
- **Top-k retrieval**: the number of most-similar chunks returned by a vector search (e.g. top 3/5/10).
- **Agentic loop**: the Strands SDK cycle of Invoke model → reason/select tool → execute tool → return result → re-invoke model → final response.
- **Agent Card**: a JSON document an agent publishes under A2A to advertise its capabilities to other agents.
- **Bedrock AgentCore**: a scalable, secure, framework-agnostic platform for building/deploying/operating agents without managing infrastructure; AgentCore Runtime is its serverless low-latency execution layer.
- **Custom Model Import**: Bedrock feature to bring a SageMaker-fine-tuned model into Bedrock as an agent's reasoning engine.
- **Strands Agents SDK**: AWS's open-source, model-driven SDK for building agents with minimal code (model + tools + prompt + agentic feedback loop).

## Mental Models
- Think of an agent as "RAG plus a decision loop": RAG gives an LLM facts, an agent gives it the ability to *plan, act, observe, and re-plan* until a goal is met.
- Use the maturity ladder (LLM → LLM+RAG → Augmented LLM → Agentic workflow → Autonomous agent → Multi-agent) as a design checklist: don't reach for multi-agent collaboration when a single Augmented LLM with tools solves the problem.
- Think of MCP as the agent's "USB-C port for tools/data" and A2A as its "phone number for other agents" — different problems, complementary solutions.

## Anti-patterns
- **Building a RAG system when the user wants action**: if users ask the system to *do* something (book a meeting, update a database) rather than answer a question, a passive RAG pipeline will disappoint them — that gap is exactly why agents exist.
- **Skipping the AWS Agentic Stack tier decision**: picking SageMaker-from-scratch when a managed Bedrock Agent would suffice adds needless infrastructure burden; picking Amazon Q when you need custom code control under-delivers.
- **Ignoring agent protocols as systems scale**: ad hoc, undocumented tool/agent integration works for a demo but breaks down once you have multiple tools or multiple specialized agents — MCP/A2A exist precisely to standardize this before it becomes unmanageable.

## Code Examples
```python
from strands import Agent

# Define the agent with a system prompt
agent = Agent()

# Interact with the agent
response = agent("Explain Amazon Bedrock Agents")

print(response)
```
- **What it demonstrates**: the minimum viable Strands agent — a default system prompt, one call, one response — showing how little code is needed before adding tools/memory in later chapters.

## Reference Tables

| Service | Best suited for | Strengths |
|---|---|---|
| Amazon Bedrock | Rapid development of generative AI applications and agents | Managed, serverless-style access to foundation models; unified API; supports agentic features and multiple model providers |
| Amazon SageMaker AI | Advanced customization, model training, fine-tuning, specialized deployment | Deep control over training, hosting, tuning, and deployment architecture; multiple endpoint/infrastructure options |

Minimum IAM actions to start building: `bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`, `sagemaker:InvokeEndpoint` (SageMaker-hosted models), `s3:GetObject`/`s3:PutObject` (data/artifacts), `bedrock-agentcore:*` (AgentCore deployments, Ch6).

## Worked Example
Building the "Hello World" agent end-to-end on AWS:
1. Prerequisites: Python 3.10+, an AWS account with Bedrock model access enabled (Console → Bedrock → Model access → enable e.g. Nova Lite/Pro or Claude), IAM permissions per the table above.
2. Dev environment: either SageMaker Studio (JupyterLab, execution-role credentials, recommended) or a local IDE (VS Code/Kiro, `aws configure` or env vars for credentials) — both need Bedrock model access enabled in the console and packages installed via `pip install`.
3. Clone the repo and enter the chapter folder:
   ```
   git clone https://github.com/PacktPublishing/AI-Agents-on-AWS
   cd chapter1
   ```
4. Install Strands: `!pip install strands-agents -q`, then restart the kernel.
5. Run the 5-line agent above — the agent is created with a default/implicit system prompt, asked "Explain Amazon Bedrock Agents," and returns a natural-language answer, powered by Bedrock underneath.
This mirrors the travel-planner example used throughout the chapter (Figure 1.3/1.10/1.11): a single MCP host reasoning over multiple specialized tool/agent calls to satisfy one complex user request.

## Key Takeaways
1. The line between RAG and an agent is the decision loop: agents plan, act, observe, and re-plan; RAG just retrieves and answers.
2. Every agent decomposes into four parts: Tools, Memory, Planning, and an LLM "brain" — use this checklist when auditing/designing any agent.
3. Choose your AWS Agentic Stack tier by required control: Amazon Q (zero-config) → Bedrock/AgentCore (managed, this book's focus) → SageMaker AI (full control, used for fine-tuning the "brain").
4. MCP standardizes agent-to-tool communication; A2A standardizes agent-to-agent communication — they're complementary, not alternatives.
5. The 6-stage maturity ladder (LLM → LLM+RAG → Augmented LLM → Agentic workflow → Autonomous agent → Multi-agent) is a design tool: build only as far up the ladder as the problem requires.
6. Strands SDK reduces an agent to 3 core components (model, tools, prompt) plus an agentic feedback loop — a working agent can be under 10 lines of code.

## Connects To
- **Ch2**: expands the "Tools" component introduced here into full tool-building with the Strands framework.
- **Ch3**: expands the "Memory" component using AgentCore Memory and Mem0.
- **Ch4/Ch5**: expand the maturity ladder's top stages (autonomous agents, multi-agent collaboration) and the MCP/A2A protocols introduced here.
- **Ch6**: uses the `bedrock-agentcore:*` IAM permission introduced here for production AgentCore deployment.
