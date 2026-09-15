# Cheatsheet — Building AI-Powered Products

## Decision rules

- **When scoping a new AI feature**: run the "AI might not be the answer" checklist first. If a simpler solution works equally well, or you can't get good data, or you can't commit to indefinite maintenance — don't use AI, even if you could.
- **When choosing 0-to-1 vs. 1-to-n**: if there's no established user base for this capability, treat it as 0-to-1 (market-discovery-heavy Ideation). If you're enhancing an existing successful product, treat it as 1-to-n (pain-point-mining-heavy Ideation).
- **When a RICE Confidence score comes back low**: that's a signal the Opportunity stage isn't finished, not a reason to discount the idea — go validate technical feasibility and business viability before scoring.
- **When judging product-market fit**: require all three pillars (business viability, technical feasibility, user desirability) — a 2-of-3 pass is a fail. Check each explicitly; don't let a strong pillar mask a weak one.
- **When building at the Concept stage**: ship an AI MVP with live data and real integration, not a mock-data prototype — hardcode non-critical parts if that gets you there faster.
- **When judging a new AI bet's early performance**: classify it first (sustaining or disruptive). If disruptive, don't kill it for underperforming your mature product on today's metrics — that's the expected pattern (Innovator's Dilemma).
- **When deciding build vs. buy**: build if AI is core to your value proposition and a long-term strategic asset; buy if it's a supporting feature and speed matters; go hybrid (build the differentiator, buy the commodity) when signals are mixed — this is the common real-world outcome.
- **When choosing fine-tuning vs. RAG vs. grounding**: fine-tune for a well-defined, precision-critical, relatively static task; RAG when the needed information changes faster than a retrain cycle; ground when you need fast, cheap, lightweight behavior tuning via prompts.
- **When choosing synthetic vs. real data**: synthetic for sensitive/rare/expensive-to-collect data; real when user behavior or contextual nuance is central to the product; hybrid (synthetic to start, real to refine) is the default for most production systems.
- **When writing an OKR**: give it exactly one North Star metric, include at least one KPI from each of product health / system health / AI proxy metrics, and always add a guardrail metric.
- **When scoping an agent's autonomy**: start at "suggests only" and progressively earn autonomy (acting on the user's behalf) as trust/accuracy are demonstrated — don't default to max autonomy at launch, especially for consequential actions (purchases, scheduling).
- **When picking an agent's UI pattern**: match it to activation type — proactive agents → side panel / integrated UI / pop-up; reactive agents → floating bubble / chat interface; blended autonomy → collaborative browser interface.
- **When sharing data with third-party AI tools**: route through legal/privacy review first — don't treat this as optional even for "just prototyping."

## Trade-off vocabulary (name the tension explicitly, in every product review)

| Trade-off | Left side | Right side |
|---|---|---|
| Accuracy vs. speed | Slower, more accurate decisions | Faster, real-time decisions |
| Complexity vs. simplicity | Better performance, harder to explain/debug | More transparent, may underperform |
| Data quality vs. quantity | Smaller, clean, compliant dataset | Larger, possibly noisy/biased dataset |
| Generalization vs. specificity | Broad-reach general model | Narrow, higher-accuracy specialized model |
| Privacy vs. personalization | More user trust, less tailoring | More tailoring, more privacy exposure |
| Ethics vs. business goals | Slower, fairer, more defensible | Faster, cheaper, higher regulatory/reputational risk |
| Explainability vs. performance | Interpretable ("white-box"), may underperform | Higher accuracy, "black-box" |

## Build-vs-buy quick matrix

| Factor | Lean build | Lean buy |
|---|---|---|
| Core competency | AI is central to value prop | AI is a supporting feature |
| Resources/expertise | Org has ML talent + infra | Org lacks ML talent/infra |
| Time to market | Not urgent | Urgent, competitive pressure |
| Long-term strategy | AI is a long-term strategic asset | AI is a short-term/interim need |
| Cost | Willing to absorb high upfront cost for long-term savings | Want lower initial cost, ok with recurring fees |
| Risk | Comfortable managing risk internally | Want risk mitigated by a proven vendor |
| Data privacy | Need full control over sensitive data | Comfortable sharing data with vendor |
| Competitive landscape | AI is a differentiator worth protecting | AI is table-stakes, not a differentiator |

## Model adaptation quick matrix

| Factor | Fine-tuning | RAG | Grounding |
|---|---|---|---|
| Latency | Higher (deep processing) | Optimizable with fast retrieval | Low |
| Data needs | Large labeled dataset | Large retrieval corpus | Minimal, relies on context |
| Accuracy | High, precision tasks | Variable, depends on retrieval | Moderate |
| Scalability | Resource intensive | Scales with corpus size | Scales easily |
| Best for | Well-defined, static, precision-critical | Fast-changing information | Rapid, cheap iteration |

## Thresholds & defaults

- **RICE score** = (Reach × Impact × Confidence) / Effort; AI variant divides further by an AI Investment factor.
- **MVQ (Minimum Viable Quality)**: no universal number — set per use case (e.g. NPS/CSAT bar for a recommender vs. ~95% accuracy for a medical diagnostic tool). Decide explicitly before debating launch readiness.
- **Product review checklist**: pre-read shared beforehand; explicit goal (decision/discussion/alignment/status) stated up front; trade-offs laid out with risks/costs/benefits; post-review summary with named owners.
- **Guardrail metric**: always pair a North Star metric with at least one cap on an acceptable side effect (e.g. "no more than 5% drop in Y while optimizing X").

## Tells & smells

- If you can't name which of the "seven superpowers" (Ch 1) a feature delivers, it isn't scoped yet.
- If a feature idea is being pitched with "because it's AI" instead of a named user pain point, it's the "shiny AI object" trap.
- If someone says "isn't ChatGPT an agent?" — no: check autonomy, learning, and proactive goal-driven action before applying the "agent" label.
- If an OKR has no guardrail metric, assume the North Star is being optimized at some hidden cost.
- If a build-vs-buy debate keeps circling without resolution, the answer is probably hybrid — stop looking for a clean binary.
- If a disruptive AI bet's early metrics look weak against your flagship product, that's expected — check whether it's being judged by the wrong (sustaining-innovation) yardstick before killing it.
