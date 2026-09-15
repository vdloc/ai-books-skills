# Chapter 5: Architecting Generative AI Software

## Core Idea
Choosing an architectural style for generative AI software (monolithic, MVC, microservice, embedded/edge) is fundamentally a trade-off exercise between scalability, maintainability, performance, latency, and security — there is no universally best style, only the one that matches your deployment constraints and quality priorities.

## Frameworks Introduced
- **Monolithic architecture**: UI, business logic, and model all in one process/module.
  - When to use: prototypes, small single-user tools.
  - How: instantiate the model pipeline directly inside the UI code (e.g., Gradio + StableDiffusion in one script).
  - Trade-off: simplest to build, but each new user/instance duplicates the entire model in memory — poor scalability, fault tolerance, and cost profile.
- **Model-View-Controller (MVC)**: separate Model (wraps the generative pipeline), View (UI), Controller (mediates input/output, e.g., truncates over-long prompts).
  - When to use: when you need to swap UIs (Gradio vs. CLI vs. web) without touching model logic, and want lower complexity/higher maintainability than a monolith, without yet needing full network-level scaling.
  - How: Model exposes one method (e.g., `generate_text`); View calls Controller; Controller validates input then calls Model. Swapping the View (e.g., Gradio → plain input loop) requires zero changes to Model/Controller.
- **Microservice-based architecture**: model hosted behind a JSON/REST API (e.g., via Flask, or frameworks like **Ollama** — uniform REST API for many open models — or **LangChain** — connects to hosted or local models and adds RAG/agent capability).
  - When to use: need multi-client reuse of one hosted model, cross-language clients (Python, C, C#), or want to compose multiple services (e.g., search + summarization).
  - Trade-off: best scalability/maintainability/interoperability, but adds latency and a larger security attack surface (data crosses the network).
- **RAG (Retrieval-Augmented / "Reality Augmented" Generation) architecture** (via LangChain): embed a user query → search a vector database for nearest documents → LLM summarizes the retrieved documents into the answer; can be extended with a live web-search fallback or a reasoning-engine chain (e.g., generate-and-execute Matlab code).
  - When to use: to reduce hallucination by grounding answers in retrieved documents instead of pure generation.
- **Embedded/edge models**: a small on-device model handles fast/low-stakes inference; a large backend model handles the rest.
  - When to use: latency-sensitive or privacy-sensitive partial tasks (e.g., grammar-check while typing) paired with heavier tasks (e.g., full document generation) sent to the cloud.
- **Other named patterns**: **Data Lake** (centralized data repo for all ML pipelines), **ML Versioning** (track model releases like HuggingFace/TensorHub do), **Workflow pipeline** (package each ML workflow in its own container), **Lambda architecture** (batch layer + speed layer + serving layer, by latency tier).
- **Architectural tactics**: load balancing (single entry point, thin controller distributing to N model replicas) for scalability; hardware-accelerator portability (ONNX) and pipeline parallelism for performance; safe tensors, secure APIs, MFA, encrypted protocols for security.

## Key Concepts
- **Client-server / microservice pattern for models**: thin client sends JSON prompt over HTTP, server hosts and runs the model — dominant pattern because clients historically lacked compute for large models.
- **Ollama**: framework giving a uniform JSON REST chat API across many open models (LLaMA, DeepSeek, etc.) — "no programming needed" to add a new model.
- **LangChain / LangSmith**: framework for RAG pipelines and agent development, connecting models to vector databases, search engines, and reasoning chains; a superset of what Ollama offers.

## Mental Models
- Use the "where does the model live" question to pick a starting architecture: same process (monolith) → same machine, separate process (MVC/local service) → different machine (microservice) → split between device and cloud (embedded/edge).
- Read Table 5.1 (Scalability/Maintainability/Performance/Latency/Security by architecture) as a first-pass filter — pick the row matching your dominant constraint, then apply tactics (Section 5.8) to shore up the weak dimensions.

## Anti-patterns
- **Defaulting to microservice architecture for everything ("cloud-native" cargo-culting)**: the book explicitly warns it's "not the silver bullet" — latency scales with the number of hops, and security risk scales with the number of network boundaries.
- **Choosing monolithic architecture for anything beyond a single-user prototype**: it hard-locks you to one programming language and duplicates full model cost per user/instance.
- **Running an unauthenticated Ollama-style server on a public port**: the author reports 100+ attacks/hour from doing exactly this — always gate model-serving endpoints behind auth (see Ch9).

## Worked Example
The book evolves the *same* text-generation feature through three architectures using the same underlying model:
1. **Monolith**: `StableDiffusion3Pipeline` + Gradio, all one script — works, but doubling users means doubling GPU/model instances.
2. **MVC**: split into `TextGenerator` (Model, one `generate_text` method), `TextGeneratorInterface` (View, Gradio), `TextController` (validates prompt length ≤512 words, then calls Model). Swapping to a CLI view (`TextGeneratorInterfaceText`) required *zero* changes to Model or Controller — proving the maintainability benefit concretely.
3. **Microservice**: wrap the same `TextGenerator` behind a Flask `/prompt` GET endpoint; Controller now calls `requests.get(...)` instead of the class directly. A C client can now talk to the same server via raw sockets — demonstrating true cross-language interoperability, at the cost of needing explicit error handling for network failures (missing in the C example, called out as a gap).

## Key Takeaways
1. Pick architecture by asking where the model runs (same process / same machine / remote / split device-cloud) — this single decision determines your scalability/latency/security profile more than any other choice.
2. MVC gives you most of the maintainability benefit of microservices (swappable UI, swappable model) without the network latency/security cost — use it before jumping to microservices.
3. RAG (embed → vector search → LLM summarize) is the standard architectural answer to hallucination, but its quality is bounded by what's in the vector database — a search-engine fallback extends coverage.
4. Embedded/edge splits let you trade a small amount of quality for latency and privacy on a subset of tasks — decide the split by task criticality, not uniformly.
5. Never expose a model-serving endpoint without authentication — attacks start within hours.

## Connects To
- **Ch6**: implementation frameworks (Ollama in C++/C#, ONNX, DirectML) build directly on architectures introduced here.
- **Ch7**: agentic AI and ChromaDB RAG examples are a direct continuation of the RAG architecture introduced in 5.4.2.
- **Ch9**: the API design chapter formalizes the JSON REST endpoints sketched here (heartbeat, versioning, auth).
