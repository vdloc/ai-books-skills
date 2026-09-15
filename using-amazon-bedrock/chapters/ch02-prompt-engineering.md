# Chapter 2: Prompt Engineering with Foundational Models on AWS

## Core Idea
Prompt engineering is the strategic craft of shaping inputs — content, structure, context, format — to steer an LLM's output toward a desired result; on Bedrock this spans playground experimentation, the five inference parameters, and defenses against prompt injection/jailbreaks/hallucinations.

## Frameworks Introduced
- **RACCCA evaluation framework**: assesses prompt/response quality across six criteria — Relevance, Accuracy, Completeness, Clarity, Coherence, Appropriateness.
  - When to use: systematically scoring AI-generated content instead of relying on gut feel.
  - How: score each response against all six criteria; use the specific assessment question per criterion (e.g., Relevance → "does it stay on-topic and directly answer the question?").
- **Zero-/One-/Few-Shot Inference spectrum**: three levels of example-provision at inference time (not training time).
  - When to use zero-shot: task is general, resource constraints prevent building a dataset, immediate understanding suffices (content moderation, general Q&A). Avoid for high-stakes specialized domains (medical diagnosis, legal advice).
  - When to use one-shot: a single example is enough to convey structure (style adaptation, low-resource translation, legal clause categorization). Avoid for complex domains needing broad coverage.
  - When to use few-shot: task benefits from pattern reinforcement across a handful of examples (movie-review sentiment classification, rare-disease diagnosis support); yields 10-30% (up to 2x in some studies) accuracy gains over zero-shot. Risk: overfitting to the shown examples in highly variable domains.
- **Advanced prompting techniques**: Chain of Thought Reasoning (force step-by-step reasoning — improves transparency/accuracy, costs more tokens/latency), Meta Prompting (ask the model to design the best prompt/strategy before answering — related to the Plan-and-Act paradigm separating planning from execution), Templating (fixed structural scaffold for consistent output format, e.g. via Bedrock Prompt Management).
  - How MCP fits in: deploy an MCP server hosting curated prompt templates/rulesets as reusable, parameterized JSON-schema templates; Bedrock models call out to it for dynamic prompt enrichment instead of static in-line instructions.

## Key Concepts
- **System prompt**: sets the model's persona/role at a level above user prompts (e.g., "You are a finance assistant, always include disclaimers").
- **Prompt injection (aka jailbreaking)**: disguising harmful intent (e.g., fictional framing) to override or erase system-prompt restrictions.
- **Jailbreak attempt**: convincing the model to role-play as an "unrestricted" version of itself to bypass safety guidelines.
- **Hallucination**: plausible-sounding but false/fabricated output, generated because the model optimizes for plausibility, not verified truth.
- **Temperature**: randomness dial — low = predictable/high-probability tokens, high = creative/diverse.
- **Top P (nucleus sampling)**: samples from the smallest token set whose cumulative probability ≥ P.
- **Top K**: restricts sampling to the K most likely next tokens.
- **Max tokens**: caps output length — also a direct cost lever.
- **Stop sequences**: token(s) that terminate generation (e.g., `"\n\nHuman"` to end a conversational turn).
- **Repetition penalty**: discourages reselecting already-used words/phrases.
- **LLM cascading**: route to a cheaper/simpler model first, escalate to a complex model only when needed.

## Mental Models
- Treat a prompt like a spec, not a question: define scope, use direct language, front-load the details that materially change the answer (time period, format, audience).
- Context and background aren't optional — a prompt without domain framing ("explain the Rosetta Stone") forces the model to guess your intent; scoping it ("...in deciphering Egyptian hieroglyphs") collapses the ambiguity.
- **Cost is a function of tokens processed, not "how smart the query is"** — few-shot prompts, chain-of-thought reasoning, and high max_tokens settings all inflate cost independent of task difficulty; each is a deliberate accuracy/cost trade you should make consciously.

## Anti-patterns
- **Vague, broad prompts** ("Discuss the dynamics of global economic structures") — the model has no scope to target, so output quality degrades.
- **Single massive complex prompt for a multi-step task** — break into sequential/templated steps instead; the model handles structured, staged instructions far better than one overloaded ask.
- **Trusting RAG alone to eliminate hallucinations** — imperfect retrieval, stale documents, or context mismatch can still hallucinate; combine with guardrails and human oversight for production/critical paths.
- **Leaving max_tokens at a high default for short-answer tasks** — directly inflates cost for no quality gain; the author cut monthly cost ~40% by dropping max_tokens from 2048→512 and routing 70% of traffic to a smaller model.

## Code Examples
```python
bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    aws_access_key_id=os.getenv('aws_access_key_id'),
    aws_secret_access_key=os.getenv('aws_secret_access_key'),
    region_name='us-west-2'
)
request_body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "messages": [{"role": "user", "content": scenario_description}],
    "max_tokens": 300,
    "temperature": 0.1,
    "top_p": 0.9
})
model_response = bedrock_runtime.invoke_model(
    body=request_body, modelId=modelId, accept=accept, contentType=contentType)
response_contents = json.loads(model_response['body'].read())
generatedText = response_contents['content'][0]['text']
```
- **What it demonstrates**: the base `invoke_model()` call pattern — build a JSON request body with the Anthropic message schema, invoke, then parse the nested response body to extract generated text.

```python
inference_modifier = {
    "max_tokens": 4096,
    "temperature": 0.5,
    "top_k": 250,
    "top_p": 1,
    "stop_sequences": ["\n\nHuman"],
}

def textgen_llm(prompt, model_kwargs={}):
    payload = {
        "anthropic_version": "bedrock-2023-05-31",
        "messages": [{"role": "user", "content": prompt}]
    }
    payload.update(model_kwargs)
    response = bedrock_runtime.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=json.dumps(payload).encode('utf-8'),
        contentType='application/json',
        accept='application/json'
    )
    result = response['body'].read().decode('utf-8')
    return result
```
- **What it demonstrates**: centralizing inference parameters (`inference_modifier`) and merging them into the request payload via `model_kwargs` — the standard pattern for tuning temperature/top_k/top_p/stop_sequences per call.

```python
multi_var_prompt = PromptTemplate(
    input_variables=["marketingManager", "productType", "targetAudience", "productFeatures"],
    template="""Human: Draft a briefing for the marketing team led by {marketingManager}...
    Example 1: ...
    Example 2: ...
    Now draft a similar briefing for Alex and the eco-friendly bicycle.
    Assistant:"""
)
prompt = multi_var_prompt.format(
    marketingManager="Alex", productType="eco-friendly bicycle",
    targetAudience="urban commuters", productFeatures="...")
```
- **What it demonstrates**: LangChain `PromptTemplate` for few-shot prompting with reusable, parameterized variables plus embedded examples ("shots").

## Reference Tables
| Parameter | Controls | Effect of increasing |
|---|---|---|
| Temperature | randomness of token selection | more creative/diverse, less predictable |
| Top P (nucleus) | cumulative-probability token pool | larger pool → more variety |
| Top K | fixed candidate-token pool size | larger K → more variety |
| Max tokens | output length cap | longer output, higher cost |
| Stop sequences | where generation halts | n/a (structural, not a magnitude dial) |
| Repetition penalty | reuse of prior words/phrases | less repetition, more novel phrasing |

| Setting | Temperature | Max Tokens | Result |
|---|---|---|---|
| A | 0.3 | 200 | Very concise, minimal creativity |
| B | 0.7 | 500 | More diverse, moderately detailed |
| C | 1.0 | 1024 | Highly creative, risk of going off-topic |

| RACCCA Criterion | Assessment Question |
|---|---|
| Relevance | Is content on-topic and directly answers the query? |
| Accuracy | Is the information factually correct and verifiable? |
| Completeness | Are all essential aspects covered without omissions? |
| Clarity | Is it simple, readable, unambiguous? |
| Coherence | Does it follow a logical structure/sequence? |
| Appropriateness | Is tone/language suitable for the audience, ethically sound? |

## Worked Example
**Practical Exercise: Finding the Right Prompt** (3 tasks, Bedrock Playground + Jupyter API calls):
1. **Creative story generation**: prompt "Write a short story about astronauts discovering a new planet" with Claude Sonnet 4; temperature=1.0/max_tokens=512 truncates the story mid-way; switching to temperature=0.5/max_tokens=2048 makes output far less creative and closer to a "default" narrative. Adding explicit setting/characters/plot details constrains the story to the desired elements — demonstrating that specificity trades against pure creativity.
2. **Technical summarization**: prompt "Briefly summarize the following article, returning key points" with Claude 3.5 v2, temperature=0.6, max_tokens=512 (lower creativity intentional — fidelity to source facts matters more than novelty).
3. **Context-aware email drafting**: given a customer inquiry about a smartwatch's health-tracking and battery life, construct a prompt embedding the specific facts to include (heart-rate variability tracking, 7-day battery life) so the model produces a professional, accurate response instead of generic filler.

**Why it works**: each task pairs a *goal* (creative, factual, transactional) with the *parameter setting* that matches it (high temp for creativity, low temp for fidelity) — making the abstract temperature/top-p guidance concrete and testable.

## Key Takeaways
1. RACCCA (Relevance, Accuracy, Completeness, Clarity, Coherence, Appropriateness) is the structured framework for scoring prompt/response quality.
2. Zero-shot/one-shot/few-shot describe examples given *at inference time*, not training — few-shot yields the largest accuracy gains but costs the most tokens.
3. Chain of thought, meta prompting, and templating are the three advanced techniques for shaping output quality and format; CoT trades latency/cost for transparency and accuracy.
4. Prompt injection, jailbreaks, and hallucinations are distinct threat/failure modes requiring different mitigations: guardrails + denied topics for injection, adaptive monitoring for jailbreaks, RAG + human oversight for hallucinations.
5. The five inference parameters (temperature, top_p, top_k, max_tokens, stop_sequences, repetition penalty) are the direct levers for both output quality and cost.
6. Cost optimization is a first-class practice: trim prompts, tune max_tokens down, use LLM cascading (cheap model first), batch inference, and caching.
7. Bedrock Playground (Chat/Text/Image) and PartyRock are no-code venues for iterating on prompts before committing them to production API code.

## Connects To
- **Ch1**: builds directly on the `InvokeModel()` mechanics and token/pricing concepts introduced there.
- **Ch3**: the Bedrock API patterns shown here (boto3 client setup, request/response parsing) are extended into full applications.
- **Ch6**: RAG is referenced here as a hallucination-mitigation technique and is covered in full depth.
- **Ch7**: prompt-performance evaluation via Model Evaluation is expanded in the performance-optimization chapter.
- **Ch8**: prompt injection/jailbreak mitigation connects to Bedrock Guardrails covered in the security chapter.
