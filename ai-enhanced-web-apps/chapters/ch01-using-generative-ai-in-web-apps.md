# Chapter 1: Using Generative AI in Web Apps

## Core Idea
Generative AI web apps integrate LLMs and other generative models into standard web-app architecture (UI, backend, data pipeline, external APIs) to produce dynamic, personalized content rather than static responses; the book's stack is React + Next.js + Vercel AI SDK + LangChain.js, defaulting to Google Gemini with occasional OpenAI use.

## Frameworks Introduced
- **The generative AI web app component stack**: User → UI/conversational components → Backend infrastructure ↔ LLMs/AI models.
  - When to use: Any time you scope a new generative-AI feature — use it as a checklist of which layer (UI, backend, data processing, API integration, deployment) a design decision belongs to.
  - How: Map each concern (caching, containerization, serverless, model-serving) to the "backend infrastructure" layer; keep model calls and UI state separate via the "conversational AI components" layer.
- **The four-stage interaction flow**: User input → Backend processing (clean/extract/select model) → Content generation → Response delivery (+ optional feedback loop).
  - When to use: When designing any request/response cycle for an AI feature, to decide where preprocessing vs. postprocessing steps live.
  - How: Explicitly place data cleaning and model selection in stage 2, generation in stage 3, and treat user feedback as an optional loop back into stage 2/3, not a bolt-on.

## Key Concepts
- **LLM (large language model)**: model trained on massive text corpora to predict the next token, enabling text generation, Q&A, summarization.
- **Transformer / self-attention**: architecture (Vaswani et al., "Attention Is All You Need") that lets a model weigh the relative importance of every other token when interpreting one token, capturing long-range dependencies.
- **Feature extraction**: technique where a model identifies and extracts meaningful patterns/representations from raw input (text phrases, image shapes) during training, then applies that ability at inference.
- **Hallucination**: AI generating false, inaccurate, or nonexistent content while presenting it as fact.
- **Bias auditing**: systematically testing a model with diverse inputs to identify and measure biased behavior.
- **PII (personally identifiable information)**: data identifying a specific individual (names, addresses, phone, email) — governed by GDPR/CCPA.
- **GAN / autoregressive model / transformer / variational autoencoder / RNN**: five model families, each suited to different generation tasks (see Mental Models below).

## Mental Models
- Use **autoregressive/transformer models** when the task is text, code, or any sequence where global context matters (large-scale LLM capability).
- Use **GANs** when the task is realistic image-to-image generation from competing generator/discriminator networks.
- Use **variational autoencoders** for anomaly detection or generating small variations of existing (mostly visual) data.
- Use **RNNs** only for small-scale sequential tasks (legacy; transformers have mostly superseded them).
- Think of pretrained vs. self-hosted models as a control/cost trade-off: self-hosting maximizes customization but demands significant compute and ML expertise; pretrained public APIs (Gemini, OpenAI) trade some control for speed to market.

## Anti-patterns
- **Treating generative AI output as inherently trustworthy**: outputs are probability-based next-token predictions, not verified facts — always validate before presenting as authoritative.
- **Ignoring PII/regulatory obligations (GDPR, CCPA) when building AI features that touch user data**: leads to compliance exposure; requires explicit consent, secure storage, and deletion mechanisms.
- **Picking a model family by hype rather than task fit**: e.g., using a GAN for text generation instead of a transformer — architecture should match the data modality and sequence requirements.

## Reference Tables
| Model type | Best for | Example |
|---|---|---|
| GAN | Image-to-image generation | These Cats Do Not Exist |
| Autoregressive | Sequential/text/code generation | GPT-4 |
| Transformer | Text generation w/ global context | Google Bard |
| Variational autoencoder | Anomaly detection, data variation | — |
| Recurrent neural network | Small-scale sequential (legacy) | Text-to-speech |

## Worked Example
Three use cases illustrate the same four-stage flow: (1) a digital marketing agency wiring DALL-E/CycleGAN/Pix2Pix + GPT-4 into a toolbox for image generation and ad copy; (2) a customer support platform combining a GPT chatbot with sentiment-analysis APIs (Google Cloud NL, IBM Watson) to generate personalized responses; (3) a job-training platform building an AI interview simulator with GPT-4 agents, speech-to-text input, and Google Dialogflow-driven personalized feedback and adaptive difficulty. Each case reduces to: pick the right model per sub-task, wire it through backend/API integration, and route output back through the UI with a feedback loop.

## Key Takeaways
1. A generative AI web app is standard web architecture (UI, backend, data pipeline) with an LLM/AI-model layer added — not a different kind of application.
2. Pick model architecture by task modality: transformers/autoregressive for text/code, GANs for image translation, VAEs for anomaly/variation, RNNs only for small legacy sequence tasks.
3. Validate AI output explicitly — hallucination and bias are default risks, not edge cases.
4. Regulatory compliance (GDPR/CCPA, PII handling) must be designed in from the start, not retrofitted.
5. The book's stack (React + Next.js + Vercel AI SDK + LangChain.js, Gemini-first) is a deliberate choice for tight framework integration and convention-over-configuration, not the only option.

## Connects To
- **Ch2**: builds the first working app on top of this stack (React + OpenAI client + Next.js backend).
- **Ch3**: goes deeper on the Vercel AI SDK introduced here as "the cherry on top."
- **Ch6/Ch7**: LangChain.js, only introduced here, becomes central for AI workflows and RAG.
