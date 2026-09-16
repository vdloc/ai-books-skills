# Chapter 7: Evaluation, Observability, and AI Governance

## Core Idea
"Deployed" and "production-ready" are different states — closing the gap requires three connected layers: evaluation (did the agent do the right thing?, built on session/trace/span telemetry), observability (what did the agent actually do, at the reasoning level, not just HTTP status?), and governance (what is the agent allowed to do, enforced deterministically outside the model's reasoning loop, not left to prompt instructions).

## Frameworks Introduced
- **Session → Trace → Span hierarchy**: a session groups a user's full conversation; each user turn within it is a trace (one complete agent execution, request to response); each trace decomposes into spans (one span per operation — a model call, a tool call, a memory retrieval).
  - When to use: as the universal vocabulary for both evaluation and observability — evaluation metrics attach to one of these three levels, and observability tooling visualizes exactly this hierarchy.
- **Agent evaluation process (4 checks)**: Tool Selection (right API/tool chosen?), Parameter Check (right inputs passed?), Context Accuracy (user's actual context used correctly?), Response Quality (accurate/clear/right tone?) — evaluated via Human-in-the-loop or LLM-as-a-judge.
  - When to use: any agent evaluation — evaluating only the final text misses tool-selection and parameter errors that don't show up in a plausible-sounding wrong answer.
- **Built-in vs. custom evaluators**: Built-in (GoalSuccessRate, Correctness, Helpfulness — ready-to-use, LLM-as-judge, no labeled dataset needed for general quality) vs. Custom (LLM-as-judge with your own rubric, or code-based/deterministic for exact-match/regex/business-rule checks).
  - When to use built-in: general quality dimensions (helpfulness, relevance, clarity) for common scenarios like a general customer-support agent.
  - When to use custom: domain-specific requirements a generic evaluator can't know — e.g. "any out-of-scope answer scores 0.0" for a travel agent, which built-in Correctness would miss entirely (it might score a polite, factually-correct off-topic refusal as "OK").
- **Evaluation levels (3-tier, matching the session/trace/span hierarchy)**: Session level (e.g. GoalSuccessRate — did the whole conversation achieve its goal?), Trace level (Correctness, Helpfulness, faithfulness, coherence — how good was this one turn?), Span/tool level (tool selection accuracy, parameter selection accuracy — the most granular, for diagnosing exactly where an agent went wrong).
- **Online vs. On-demand evaluation**: Online = continuous evaluation on sampled/filtered live production traffic, for trend monitoring; On-demand = evaluate specific selected traces/spans, for debugging a reported issue, validating a fix, or build-time testing.
- **Four layers of agent observability**: Layer 1 Infrastructure metrics (CPU/memory/network — CloudWatch, says nothing about agent correctness), Layer 2 Request-level telemetry (latency/errors/throughput — standard API monitoring, tells you *that* something broke, not *why*), Layer 3 Agent execution traces (the agent-specific layer — full reasoning/tool/token lifecycle per invocation), Layer 4 Behavioral analytics (patterns across many traces — catches slow degradations no single request would flag as an error).
  - When to use: Layer 3 is where agent-specific instrumentation is essential; Layers 1-2 work the same for agents as any other app.
- **OpenTelemetry (OTel) as the portability layer**: an open standard for traces/metrics/logs; instrument once, send to any OTel-compatible backend (Jaeger, Langfuse, LangSmith, CloudWatch) without changing agent code.
  - How: `StrandsTelemetry().setup_otlp_exporter()` plus `OTEL_EXPORTER_OTLP_ENDPOINT` env var — Strands has native OTel support, auto-emitting spans for every model call and tool invocation.
- **AI Governance's 3 levels**: Guardrails on model inputs/outputs (content filtering — controls what the agent *says*), Policy enforcement on tool calls (Cedar rules — controls what the agent *does*), Organizational governance (model cards, audit trails, incident response — the human accountability layer no technology replaces).
  - Why 3 levels, not 1: guardrails filter text but never see tool-call decisions; policy governs tool execution but has no opinion on phrasing; neither substitutes for human accountability on "should we build this agent at all."
- **Cedar policy language (AgentCore Policy)**: declarative `forbid`/`unless` rules evaluated deterministically against every tool call, outside the model's reasoning — "no amount of prompt engineering overrides it."
  - When to use: any rule with real-world consequences that must hold 100% of the time (refund limits, cross-tenant data isolation, role-based record access) — never rely on a system-prompt instruction for these.

## Key Concepts
- **Instrumentation library vs. instrumentation agent**: the library (OpenTelemetry, OpenInference) defines *what* telemetry to capture in code; the agent (ADOT) *collects and exports* that telemetry automatically at runtime without manual per-call instrumentation.
- **LLM-as-a-judge**: a model scoring another agent's trace/output against a rubric, usable even without a ground-truth answer.
- **ADOT (AWS Distro for OpenTelemetry)**: AWS's distribution of OTel; auto-instruments agents on AgentCore Runtime (just add `aws-opentelemetry-distro` to requirements) or agents running elsewhere (manual `opentelemetry-instrument` wrapper).
- **Policy evaluation span**: a trace entry unique to AgentCore showing that a Cedar rule was checked and its permit/deny outcome — neither Langfuse nor LangSmith can capture this because they have no policy engine.
- **Model card**: production-required documentation of what an agent does, its tools, known limitations, evaluation results, and active guardrails/policies — lives in version control alongside the agent's code.
- **WORM (Write Once Read Many) archival**: CloudWatch Logs exported to S3 with Object Lock, giving an immutable audit trail satisfying regulatory record-keeping (e.g. 7-year retention).

## Mental Models
- Treat evaluation and observability as the same data pipeline viewed two ways: instrumentation captures sessions/traces/spans; evaluators score that captured data; without the traces, there is nothing to evaluate except the final text.
- Diagnose "why is my agent slow/expensive" the way the book's fintech loan-underwriting story does: a 200 OK with correct output can still hide 3x redundant tool calls burning latency and tokens — only Layer 3 traces reveal an *interaction pattern* shift that input/output logging never would.
- Split "what the agent says" from "what the agent does": guardrails own the former (text filtering, PII, denied topics), Cedar policy owns the latter (tool-call authorization) — a well-governed system needs both, since a guardrail can't stop an unauthorized `$50,000` refund tool call (it never sees the call, only the resulting text), and a policy can't stop the model from describing PII in prose if the tool call to fetch it was itself authorized.

## Anti-patterns
- **Evaluating only the final response text**: misses wrong tool selection, wrong parameters, or misused context that a plausible-sounding final answer can mask entirely.
- **Assuming built-in evaluators cover domain-specific rules**: a travel agent's polite, factually correct refusal to a Python-scripting question can score "OK" on generic Correctness while a custom evaluator correctly scores it "Very Poor" for violating domain scope — built-in metrics don't know your business rules.
- **Logging everything to stdout and calling it observability**: print statements have no parent-child relationships, no timing data, and can't be correlated across concurrent requests — use structured OTel traces instead.
- **Monitoring only the happy path**: the 5% of failing requests are exactly the ones that matter; traces must capture failures (timeouts, retries, silent hallucination fallback) with the same fidelity as successes.
- **Ignoring token cost until the monthly bill arrives**: cost tracking belongs in observability from day one as a baseline, so a cost spike can be diagnosed as "traffic doubled" (expected) vs. "the agent started making 2x model calls per request" (a bug) — not discovered a month later.
- **Treating observability as a post-launch concern**: retrofitting instrumentation while simultaneously debugging a live production incident is exactly the wrong time to add tracing; the two-line OTel setup takes 30 seconds pre-deployment.
- **Relying on system-prompt instructions ("don't issue refunds over $1,000") as a compliance control**: prompt instructions are suggestions the model may or may not follow depending on phrasing/context/version/luck — "you cannot build a compliance program on 'the model usually follows the system prompt.'" Use Cedar policy instead, which is deterministic and enforced outside the reasoning loop.
- **Reusing an inbound identity token to authorize a downstream tool call, or trusting conversation-derived parameters for tenant isolation**: Cedar policies must check *authenticated* identity attributes (`principal.orgId`), never values a prompt-injected conversation could have influenced.

## Reference Tables

| Scenario | Evaluator to use | Why |
|---|---|---|
| Customer support agent, general questions | Built-in evaluator | Standard quality dimensions (helpfulness, relevance, clarity) suffice |
| Healthcare scheduling agent, strict business rules | Custom evaluator | Domain-specific checks (insurance verification, no medical advice, escalation rules) need custom logic |

| Evaluation Level | Examples | What it diagnoses |
|---|---|---|
| Session | GoalSuccessRate, custom business metrics | Did the overall interaction succeed? |
| Trace | Correctness, Faithfulness, Helpfulness, Coherence, Instruction-following | Was this one turn good end-to-end? |
| Span/Tool | Tool selection accuracy, Parameter selection accuracy | Exactly where inside a turn did it go wrong? |

| Layer | Captures | Answers |
|---|---|---|
| 1. Infrastructure metrics | CPU, memory, network, container health | Is the compute healthy? |
| 2. Request-level telemetry | Latency, error rate, throughput | Is something slow/broken? |
| 3. Agent execution traces | Full invocation lifecycle: prompt, model calls (tokens), tool calls (params/results), response | *Why* is something slow/broken — the agent-specific layer |
| 4. Behavioral analytics | Patterns across many traces (drifting tool-call counts, degrading prompt patterns) | Is quality silently degrading over time? |

| | Bedrock Guardrails | AgentCore Policy |
|---|---|---|
| What it controls | Model text output | Tool call execution |
| How rules are defined | Bedrock console config | Cedar policy language |
| Enforcement mechanism | Content filtering (regex, ML classifiers) | Deterministic rule evaluation |
| Bypassable by prompt engineering? | Difficult but theoretically possible | No — evaluated outside the model |
| User context awareness | No | Yes — references authenticated identity |
| Scope | Per model call | Per tool call |

## Worked Example
End-to-end evaluation of a travel assistant deployed to AgentCore Runtime:
1. Build a Strands agent with 3 tools (`get_flight_info`, `get_hotel_recommendations`, `get_weather_forecast`) and a scope-restricting system prompt ("Only answer questions related to travel... For anything outside travel, politely decline").
2. Deploy via `runtime.configure(entrypoint="agent_app.py", ...)` then `runtime.launch()` — CodeBuild builds an ARM64 image, no local Docker needed (~10 min first deploy).
3. Register a custom LLM-as-judge evaluator from `travel_quality_metric.json`, whose rubric explicitly states: *"IMPORTANT: If the assistant answers non-travel questions, classify as Very Poor."*
4. Invoke with 4 test prompts sharing one `session_id` (3 in-scope: flights/hotels/weather; 1 out-of-scope: "write a Python script to sort a list"), then wait ~3 minutes for OTel traces to land in CloudWatch Transaction Search.
5. Run 3 evaluation passes: Pass A `Builtin.GoalSuccessRate` (session level) → 0.75 "Good"; Pass B `Builtin.Correctness` + `Builtin.Helpfulness` (trace level) → 1.00 "Very Good" (the agent correctly refused the Python question, so generic correctness scores it fine); Pass C the custom `travel_response_quality` evaluator (trace level) → 0.00 "Very Poor" on that same turn.
6. **Key insight**: Pass B and Pass C disagree on the exact same response because built-in evaluators don't know the business rule about staying in-domain — only the custom evaluator catches the out-of-scope violation, proving why domain-specific custom evaluators are necessary even when general-purpose metrics look fine.

## Key Takeaways
1. Evaluation requires visibility into the full execution (tool selection, parameters, context use, response quality) — not just judging the final text.
2. Attach evaluation metrics to the right level: session (GoalSuccessRate) for overall conversation success, trace (Correctness/Helpfulness) for per-turn quality, span/tool (selection/parameter accuracy) for granular diagnosis.
3. Built-in evaluators cover general quality; custom evaluators are required to catch domain-specific rule violations that generic metrics structurally cannot see.
4. Agent observability needs a 4th, agent-specific layer (execution traces: reasoning, token counts, tool calls) beyond standard infra/request monitoring — a 200 OK can still hide hallucination, wrong tool calls, or 40x cost blowup.
5. Instrument with OTel from day one (two environment variables, thirty seconds) — retrofitting tracing during a live incident is the worst possible time to add it.
6. Use CloudWatch/AgentCore Observability for zero-config production alerting, and Langfuse (self-hostable, open-source) or LangSmith (LangChain-ecosystem, playground for prompt iteration) for deep trace analysis — they're complementary, not exclusive, since all three speak OTel.
7. Never rely on system-prompt phrasing to enforce a hard business rule — Cedar policy (AgentCore Policy) enforces deterministically outside the model's reasoning, immune to prompt injection.
8. Guardrails control what the agent *says* (text filtering); Policy controls what the agent *does* (tool-call authorization) — deploy both, since neither substitutes for the other.
9. Organizational governance (model cards, immutable WORM audit trails, incident-response plans) is the human accountability layer that no Cedar rule or guardrail can replace — "technology handles enforcement, humans handle accountability."

## Connects To
- **Ch1-Ch6**: this chapter is the capstone — every capability built across the book (tools, memory, multi-agent patterns, MCP/A2A, AWS deployment) is made trustworthy here through evaluation, observability, and governance.
- **Ch3**: AgentCore Memory operations (retrieval/writes) appear as their own spans in AgentCore Observability traces.
- **Ch5**: MCP tool calls and A2A message exchanges are exactly the kind of spans captured and evaluated at the span/tool level here.
- **Ch6**: AgentCore Policy and Identity, introduced architecturally in Ch6, are used operationally here (Cedar rules, authenticated-identity-based policies); the three deployment paths (Lambda/ECS/AgentCore) each have different containment procedures during incident response.
