# Glossary

**Agentic AI** — semiautomated AI systems that pursue goals with substantial autonomy: interpreting instructions, planning steps, choosing tools, and adapting when initial attempts fail (Ch 6).

**Agentic transference** — see Algorithmic transference.

**AI overreliance** — users depending too heavily on AI, accepting incorrect recommendations and letting their own critical-thinking skills atrophy; includes skill atrophy, automation bias, confirmation bias, and ordering effects (Ch 5).

**Algorithmic transference** — users carrying assumptions/frustrations from earlier AI interactions into new ones, even when no longer applicable (Ch 2).

**AI Organizational Readiness levels** — 3-level maturity model (Individual exploration → Shared understanding/manual maintenance → Integrated AI organization) for how reliably model-capability knowledge flows through an org, adapted from the SAE autonomous-driving scale (Ch 1).

**Canvas** — an interactive, editable, spatially organized, persistent, multimodal workspace for AI-generated content, distinct from linear chat (Ch 5).

**CARE framework** — Context, Action, Results, Examples; Nielsen Norman Group's structure for what a strong prompt should supply (Ch 3).

**Checkpoint** — a pause point during multistep agentic work where progress is shown, assumptions confirmed, or approval requested before continuing (Ch 6).

**Confidence indicator** — a design pattern asking a model to self-report correctness likelihood; unreliable because LLM "confidence" is really next-token probability, not epistemic certainty (Ch 5).

**Constitutional AI** — Anthropic's training method using a written set of guiding principles (a "constitution") instead of only human feedback to shape model behavior (Ch 1, Ch 5).

**Context window** — the maximum number of tokens a model can process at once (Ch 4).

**Decomposition** — breaking a user's goal into actionable parts, shown as a plan before agentic execution begins (Ch 6).

**Delegation (agentic)** — the point where an agent moves from describing intent to carrying it out, often handing off to sub-agents or external tools (Ch 6).

**Diffusion model** — image-generation approach starting from pure noise and iteratively refining toward a target, as opposed to LLMs' token-by-token generation (Ch 4).

**Direct manipulation** — the third channel of user intent: buttons, sliders, selections, gestures that shape AI behavior without verbal articulation (Ch 3).

**Discoverability** — what a user knows is possible within a product and how they learn it (Ch 2).

**Discovery 2×2 matrix** — classifies discovery pattern choice along axes of user intent (does the user know they want this?) and system initiative (does the tool act proactively or wait?) (Ch 2).

**ELIZA effect** — the tendency to project humanlike understanding onto text that only simulates it (Ch 1).

**Explicit prompting** — the typed or spoken request that tells an AI system what to do; one of the three channels of intent (Ch 3).

**Fine-tuning** — training stage using a smaller, targeted dataset (often with RLHF) to shape a pretrained model toward specific behaviors like instruction-following (Ch 2).

**Grounding** — contextual signals (model version, agent/tool, locale, mode, session history) that reveal the lens through which an output was generated; distinct from verifiability (Ch 5).

**Hallucination** — fluent, confident, generated content unsupported by real data; models optimize for plausibility, not factual accuracy (Ch 1, Ch 4).

**HITL (human-in-the-loop)** — design approach keeping people actively involved in AI decision-making (providing feedback, reviewing escalated edge cases) rather than fully automating (Ch 5).

**Hick's Law** — decision time increases with the number/complexity of available choices (Ch 2).

**Implicit context** — information a system infers from the environment without being asked (open document, selection, location, history) (Ch 3).

**Inference** — the act of running a trained model on new input to generate an output (Ch 4).

**Intent (user)** — the specific goal or outcome a person wants to achieve; distinguished from capability, discovery, and orchestration (Ch 2).

**Jakob's Law** — users spend most of their time on other products/sites, so they expect new interfaces (including AI ones) to behave like familiar ones (Ch 3).

**Latent space** — a compressed, multidimensional map where a model represents data by relationships (meaning, style, category), not just identity (Ch 4).

**Lost in the Middle effect** — models attend less to information in the middle of long prompts than at the start/end (Liu et al. 2023) (Ch 3).

**Miller's Law** — humans hold roughly seven items in working memory at once (Ch 4).

**Model Context Protocol (MCP)** — an emerging standard formalizing how a tool describes its capabilities and required inputs so a model can invoke it (Ch 2).

**Model switching** — computational process of selecting different specialized models/configurations based on task requirements or performance optimization (Ch 4).

**Momentum behavior** — users sticking to a chosen interface path even when better options exist, because weak signaling failed to call attention to alternatives (Ch 2).

**Multiagent Collaboration pattern** — distributing work among specialized agents coordinated by an orchestrator (sequential handoff, parallel collaboration, or supervised collaboration) (Ch 6).

**Optimistic UI** — showing a confirmation before the server response is finalized, used to maintain user confidence in transactional flows (Ch 4).

**Orchestration** — the design layer connecting user intent and model capability: config choices like model/tool selection, usage metering, permissions, library management (Ch 2).

**Planning pattern** — enabling agents to break high-level goals into structured sequences of subtasks (linear, hierarchical, or adaptive/branching) (Ch 6).

**Post-processing (output)** — cleanup/formatting/enhancement stage between internal generation and delivery (detokenization, guardrail scanning) (Ch 5).

**Pretraining** — initial training stage teaching a model general next-word prediction over massive text corpora (Ch 2).

**PageRank** — Google's link-analysis algorithm ranking web pages by citation-like hyperlink authority rather than pure keyword matching (Ch 3).

**Progressive disclosure** — revealing functionality/detail in stages rather than all at once, to avoid overwhelming new users (Ch 2, Ch 4).

**ReAct pattern (Reason + Act)** — adaptive agentic loop alternating between reasoning about what to do, taking action, and observing results, without a fixed predetermined plan (Ch 6).

**Reflection pattern** — an agent evaluating and refining its own output (generate → critique → regenerate) before presenting it to the user (Ch 6).

**RLHF (reinforcement learning from human feedback)** — fine-tuning technique where human reviewers rank model responses to shape behavior toward helpfulness/alignment (Ch 2, Ch 5).

**Rollback** — restoring an agentic process to a prior checkpoint (Git-style version history), without deleting the current state (Ch 6).

**Routing** — determining which models, tools, or steps handle a given request (Ch 2, Ch 4).

**Sequencing (agentic)** — the dependency order in which agentic subtasks must run (Ch 6).

**Stochastic parrot** — critique (Bender et al. 2021) that model scale creates the illusion of understanding while merely recombining training-data patterns, amplifying bias (Ch 1, Ch 5).

**Sycophancy** — LLM tendency to affirm a user's stated premise rather than correct it, reinforced by RLHF's preference for agreeable-rated responses (Ch 3).

**SynthID** — Google DeepMind's watermarking technology embedding invisible statistical signatures into AI-generated content during (not after) generation (Ch 5).

**Temperature** — sampling-randomness parameter; low = deterministic/conservative, high = varied/creative but more error-prone (Ch 4, Ch 5).

**Tokenization** — breaking text into subword units mapped to numeric token IDs for model processing (Ch 4).

**Tool Use pattern** — extending an agent with external capabilities (APIs, databases, calculators) so it can act, not just describe (Ch 6).

**Verifiability** — whether a claim in an AI output is independently checkable by the user; the key question replacing bare confidence scores (Ch 5).

**Vibe coding** — AI-assisted development (Karpathy, 2025) where generated code is accepted without review and debugged by pasting errors back to the LLM; suited for prototypes, not production (Ch 2, Ch 5).
