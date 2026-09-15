# Chapter 4: Functional and Non-Functional Requirements for Generative AI Software

## Core Idea
For generative AI software, functional requirements are deceptively simple to write ("generate an image from a prompt"), but nearly all engineering difficulty — and nearly all specification effort — lives in the non-functional requirements (latency, privacy, fault tolerance, interoperability) and in acceptance criteria that must describe probabilistic, not deterministic, correctness.

## Frameworks Introduced
- **Functional vs. non-functional balance for GenAI** (Fig 4.1): traditional software front-loads functional specification with non-functional as an afterthought; generative AI software inverts this — the functional requirement is often one sentence, while acceptance criteria and non-functional requirements dominate.
  - How: write functional requirements as user stories, but pair every one with explicit, checkable acceptance criteria (e.g., "the image should contain all nouns from the prompt as elements; no visible merge artifacts").
- **Two data quality models** (Foidl et al. 2024 / DQSOps, Bayram et al. 2023): map ISO/IEC 25012 data dimensions (accuracy, completeness, consistency, credibility, currentness) to concrete "influencing factors" (validation processes, metadata docs, versioning, provenance, freshness) or to computable statistics (missing-value fraction, Kolmogorov-Smirnov/Anderson-Darling goodness-of-fit for concept drift, Jensen-Shannon divergence for skew).
  - When to use: when auditing whether your training/RAG data is fit for purpose — pick the model that matches your pipeline maturity (Foidl for governance-level audits, DQSOps for continuous/streaming quality monitoring).
- **ISO/IEC 25010 software quality model adapted to GenAI**: of the eight standard attributes (functional suitability, performance efficiency, compatibility, usability, reliability, security, maintainability, portability), the book flags **performance efficiency** and **compatibility** as the two GenAI-critical ones, because model size/latency/hosting choice directly trades off against them.
- **Latency/quality/model-size triangle** (Fig 4.2): you can typically only prioritize two of {fast response, high response quality, small/local model} at once — pick based on the use-case category (embedded-in-workflow → speed; standalone quality product → quality; on-device/private → fixed small model).

## Key Concepts
- **User story with acceptance criteria (GenAI-specific)**: must specify probabilistic-content checks (e.g., "no visible artefacts from merging elements") rather than deterministic ones.
- **Data Dependencies / Representation / Variety / Volume**: the four "data characteristics" factors that determine what pipeline and hardware a GenAI system needs.
- **Data Governance / Security / Metadata**: the three "data management" factors — licensing and provenance tracking became legally urgent after early lawsuits over untracked training data.
- **Concept drift**: data distribution changes over time, silently degrading model relevance — not covered by ISO/IEC 25000 because that standard predates modern ML.
- **Maintainability (GenAI-specific)**: ability to swap the underlying model without breaking the product — critical because "a generative AI model ages quickly, and not gracefully."
- **Fault tolerance sources**: (1) malformed/too-long user input, (2) infrastructure/datacenter unavailability — each needs its own explicit non-functional requirement (e.g., "fall back to Phi-3-mini locally if the datacenter connection is lost within 30 seconds").
- **Privacy & Integrity requirements**: must state that user prompt data is not used for training, and that responses cannot re-identify real individuals — informed by real incidents (e.g., unauthorized voice cloning).
- **Latency mitigation tactics**: streaming responses, predictive prompt completion, server-side response caching, and eager multi-variant generation.
- **Interoperability**: both client-platform portability (Windows/Linux/macOS/Android/iOS) and model portability across compute architectures (NVidia vs. ARM via ONNX).

## Mental Models
- When writing a GenAI requirement, ask "what would the acceptance criteria even look like?" first — if you can't state a checkable criterion, the functional requirement isn't finished yet, regardless of how simple the one-line description reads.
- Use the latency/quality/model-size triangle as a first-pass architecture filter before writing detailed non-functional requirements: classify your use case (real-time-embedded / premium-quality / fixed-device) and let that classification drive which quality attributes get priority.

## Anti-patterns
- **Writing GenAI user stories like login-screen boilerplate**: "As a user, I want to log in..." template thinking doesn't transfer — GenAI requirements need explicit acceptance criteria about model output correctness, which is qualitatively different work.
- **Applying ISO/IEC 25000 data quality models unmodified**: they don't cover concept drift, model robustness, fairness, or explainability — supplement with a GenAI-aware data quality model (Foidl et al. / DQSOps).
- **Specifying "the response should be good" without a latency bound or streaming strategy**: latency requirements need explicit numeric thresholds (e.g., <1 second, or <30 seconds before declaring the datacenter connection lost) — vague quality language is not testable.

## Worked Example
A functional requirement for an LLM image generator, showing the functional/non-functional imbalance:
> **Functional**: "As a user, I want to generate realistic images from prompts... e.g., 'Create an image of a bee on a sunflower.'"
> **Acceptance criteria (non-functional-heavy)**: the image must contain every noun from the prompt as an element; adjectives tied to those nouns must be visually present; no visible merge artifacts; no generation artifacts.
This is contrasted with a **fault-safety pair** of requirements for datacenter loss:
> Primary: "When the connection to the datacenter is lost, use a local smaller model (e.g., Phi-3-mini instead of Phi-3-large)."
> Alternative: "...notify the user and display a default error message."
> Detection: "The connection should be considered lost if no response arrives within 30 seconds."
This triplet shows the pattern: define the failure condition precisely, then define the response as its own testable requirement.

## Key Takeaways
1. In GenAI software, acceptance criteria carry almost all the specification weight — write them as concretely checkable statements about model output, not vague quality language.
2. Use a GenAI-aware data quality model (Foidl et al./DQSOps), not raw ISO/IEC 25000, when auditing training/RAG data — ISO predates concept drift, fairness, and explainability concerns.
3. Prioritize performance efficiency and compatibility above the other six ISO/IEC 25010 attributes for GenAI products specifically — these are where model/architecture choices bite hardest.
4. Model maintainability (the ability to swap models without redesign) must be an explicit non-functional requirement, because models age faster than traditional software components.
5. Every fault-tolerance requirement needs three parts: the failure condition (with a numeric detection threshold), the primary response, and (often) a fallback response.

## Connects To
- **Ch5**: architectural styles are largely a response to the latency/quality/model-size triangle defined here.
- **Ch7**: data governance/quality concepts here underpin the RAG/vector-database data-handling practices.
- **Ch3**: builds on the functional/non-functional split first introduced there.
