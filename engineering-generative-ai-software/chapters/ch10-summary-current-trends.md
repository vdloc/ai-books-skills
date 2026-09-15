# Chapter 10: Summary and Current Trends — Beyond the State-of-the-Art

## Core Idea
The shift from "bigger models" to "reasoning models" (mixture-of-experts, RLHF, human-in-the-loop training) has redirected the field's center of gravity from raw scale to hybrid architectures — classical software directing generative AI, or generative AI directing classical software functions — and this hybridization, not further scaling, is what the author bets on as the field's near-term trajectory.

## Frameworks Introduced
- **Two hybrid-architecture types**:
  1. **AI in the driving seat**: the generative AI model is given access to software functions/tools and invokes them as needed (foreshadows agentic tool-use patterns).
  2. **Classical software in the driving seat**: deterministic code calls into generative AI for specific sub-tasks it can't do itself (this is the pattern used throughout the book's own agent examples — the Python program controls when/how the model and compiler are invoked).
  - When to use: pick "AI in the driving seat" when the *sequence* of steps is genuinely unknown ahead of time (e.g., open-ended research assistance); pick "classical software in the driving seat" when you can enumerate the workflow and just need AI for specific generative sub-steps (e.g., code generation within a fixed compile-test loop) — the latter is more controllable and easier to test (per Ch3/Ch6's testing frameworks).
- **Reasoning-model trend break**: 2020-2023 = "bigger is better" (GPT-3→3.5→4, expectation of GPT-5); 2024 = OpenAI's first reasoning model uses mixture-of-experts (MoE) instead of pure scale; 2025 = DeepSeek's reasoning model adds human-in-the-loop training (model asks a human for facts during training) on smaller GPUs — model *capability to reason* replaces raw parameter count as the key differentiator.
- **EU AI Act risk-tiering** (regulatory framework, full force ~2026): categorizes AI systems from minimal to unacceptable risk; high-risk systems (including foundation models) face documentation, transparency, robustness, and human-oversight obligations, plus disclosure of training data, energy usage, and safeguards.

## Key Concepts
- **Mixture-of-Experts (MoE)**: architecture where only a subset of the model's parameters ("experts") activate per input, enabling reasoning-level capability without proportional compute cost — the technical driver behind the 2024-2025 reasoning-model shift.
- **RLHF (Reinforcement Learning with Human Feedback)**: combines LLMs with reinforcement learning so models don't need to map the entire state space, only the relevant part — shrinks models while preserving effectiveness (cited use case: AI-assisted fighter-jet control).
- **Human-in-the-loop training**: model queries a human for facts *during* training (not just via RLHF post-training) to speed convergence — DeepSeek's stated innovation.
- **Coopetition/business-model shift**: pre-ChatGPT, investment favored proprietary foundation models; post-~2023, investment shifted to *products built on* AI (e.g., CoPilot) rather than the models themselves — data access, not model ownership, is now the competitive moat.

## Mental Models
- When evaluating a new "bigger model" release against a "smaller reasoning model" release, ask which axis actually matters for your task: raw knowledge breadth (favors scale) vs. multi-step logical correctness (favors reasoning/MoE architectures) — don't assume scale wins by default anymore.
- When architecting a new agentic feature, explicitly decide "AI in the driving seat" vs. "classical software in the driving seat" as a first design decision, because it determines how testable and controllable the resulting system will be.

## Anti-patterns
- **Assuming continued parameter-count scaling is still the primary lever for capability gains**: the book documents this assumption breaking around 2024 (data scarcity for a hypothetical GPT-5-class jump, reasoning models providing a different capability lever entirely).
- **Building agentic systems with undefined stop conditions "because the AI will figure it out"**: explicitly flagged as an unsolved open problem (Section 10.7) — the book's own Ch7 agent experiments show agents do not self-terminate sensibly; production systems need an engineered stop criterion.
- **Treating AI-generated code/content as automatically free of IP/liability concerns**: Section 10.8 flags unresolved questions around derivative-work status, copyright ownership, and liability chains through model providers/toolchains/integrators — don't assume these are solved just because generation "worked."

## Worked Example
The book's own retrospective frames its content as an instance of "classical software in the driving seat": every worked example across chapters (Ch1's decBERTa training script, Ch7's compiler-in-the-loop agent, Ch9's Flask API) is deterministic Python code that *calls into* a generative model for a bounded sub-task, then validates/post-processes the result with ordinary software logic (accuracy checks, compilation, response-code handling) — never letting the model run an open-ended, unsupervised workflow. This is offered as the currently-reliable pattern, while "AI in the driving seat" (an agent autonomously deciding which tools to invoke and when to stop) is flagged as still largely unsolved (Section 10.7's "AI-powered problem solving without explicit objectives").

## Key Takeaways
1. The field's center of gravity has shifted from "scale up the model" to "add reasoning" (MoE) and "add human-in-the-loop training" — plan model-selection strategy around this axis, not just parameter count.
2. Default to "classical software in the driving seat" hybrid architectures for production systems — they're more testable/controllable than "AI in the driving seat," which remains an open research problem for stop-condition and objective-definition reasons.
3. Investment and business value have shifted from owning proprietary foundation models to owning products/data built on top of commodity models — plan your competitive moat accordingly (see Ch9's ecosystem layering).
4. Track the EU AI Act (full force ~2026) and equivalent US/China regulation now — high-risk system obligations (documentation, transparency, human oversight, training-data disclosure) will apply retroactively to systems built without this in mind.
5. IP ownership and liability for AI-generated content/decisions remain legally unresolved — build attribution tracking, human oversight, and license-aware pipelines into your process now rather than treating this as solved.

## Connects To
- **Ch7**: the "AI in the driving seat" vs. stop-condition problem is a direct callback to the multi-agent convergence findings there.
- **Ch3**: the hybrid-architecture recommendation reinforces the dual-track (deterministic + probabilistic) testing framework established in Chapter 3.
- **Ch9**: business-model shift (products over models) extends the ecosystem-layering discussion from Chapter 9.
