# Chapter 3: Building Applications with the Amazon Bedrock API

## Core Idea
Every Bedrock application boils down to invoking one of four specialized endpoints (`bedrock`, `bedrock-runtime`, `bedrock-agent`, `bedrock-agent-runtime`) with the right model-specific request body — and production readiness comes from layering exception handling, streaming, prompt caching, and cost controls on top of that base call.

## Frameworks Introduced
- **Four-endpoint decision model**: `bedrock` (control plane — model lifecycle), `bedrock-runtime` (data plane — direct inference, `InvokeModel`), `bedrock-agent` (control plane — agents/knowledge base setup), `bedrock-agent-runtime` (data plane — invoking agents/querying knowledge bases).
  - How: simple text/image generation → `bedrock-runtime`; multistep/context-aware orchestration → `bedrock-agent-runtime`; model/config management → `bedrock`; agent setup/supervision → `bedrock-agent`.
- **Generative AI Web App build sequence**: Architect concept/functionality → Choose technology stack → Set up backend → Integrate AI model → Develop front-end → Handle input/display responses → Test & iterate → Deploy.
  - When to use: any time you're moving from "I can call the API" to "I have a shippable app" — the author walks this exact sequence building a Flask food-recommender.

## Key Concepts
- **Synchronous (`InvokeModel`) vs. streaming (`InvokeModelWithResponseStream`)**: sync returns the full output at once (good for batch); streaming delivers output token-by-token (good for chat/real-time UX, lower perceived latency).
- **Steerability**: a model's capacity to be guided toward desired behavior/style via prompts, system instructions, or sampling parameters.
- **Prompt caching**: caches static prompt prefixes (system instructions, few-shot examples) so repeated portions skip recomputation — up to 90% input-token cost reduction and 85% latency reduction on supported models (Claude Sonnet 3.7, Claude 3.5 Haiku, Nova Micro/Lite/Pro). Cache entries live 5 minutes, TTL resets on each hit.
- **Exception types**: `AccessDeniedException` (403 — IAM permissions), `ResourceNotFoundException` (404), `ThrottlingException` (429 — rate limit), `ModelTimeoutException` (504), `InternalServerException` (500), `ModelStreamErrorException` (424), `ValidationException` (400 — bad input format), `ModelNotReadyException` (429), `ServiceQuotaExceededException` (402), `ModelErrorException` (500).
- **LangChain `RunnableWithMessageHistory`**: wraps a chat model + a session-keyed history factory so multi-turn context (system prompt seeded once, user turns appended) is managed automatically instead of manually.

## Mental Models
- Treat the four Bedrock endpoints as a **control-plane/data-plane split**: `bedrock`/`bedrock-agent` manage things, `bedrock-runtime`/`bedrock-agent-runtime` *do* things (inference). Picking the wrong plane is the most common early confusion.
- Think of **prompt caching as pinning the unchanging half of your prompt** — system instructions and few-shot examples are stable across requests, so cache exactly that and leave dynamic user input uncached to balance reuse against freshness.
- **Cost control is architectural, not just parameter tuning**: model selection, on-demand/event-driven invocation (Lambda + EventBridge), concurrency/rate limiting, and monitoring (Cost Explorer, Budgets, CloudWatch) are all levers — pick the smallest model that meets the bar, invoke only on demand, and instrument spend before it surprises you.

## Anti-patterns
- **Ignoring exception types and hardcoding a single catch-all** — each exception (throttling, validation, access-denied) has a distinct root cause and different remediation (retry with backoff vs. fix IAM policy vs. fix input format); handle them distinctly.
- **Streaming without checking `responseStreamingSupported`** — not all models support `InvokeModelWithResponseStream`; verify via `GetFoundationModel` first.
- **Caching dynamic content** — caching user-specific or per-request text defeats the purpose and risks serving stale context; cache only the static prefix.
- **Skipping requirements/audience scoping before building** — the author describes a project that was over-engineered because target-user needs weren't nailed down first, causing costly rework; scope the audience and core interaction before picking a tech stack.

## Code Examples
```python
# Synchronous invocation
model_response = bedrock_runtime.invoke_model(
    body=request_body, modelId=modelId, accept=accept, contentType=contentType)
response_contents = json.loads(model_response['body'].read())
generatedText = response_contents['content']
```

```python
# Streaming invocation with chunked processing
streamed_response = bedrock_runtime.invoke_model_with_response_stream(
    body=request_body, modelId=modelId, accept=accept, contentType=contentType)
stream = streamed_response.get('body')
output = []
if stream:
    try:
        for event in stream:
            chunk = event.get('chunk')
            if chunk:
                chunk_data = chunk.get('bytes')
                if chunk_data:
                    decoded_chunk = chunk_data.decode()
                    chunk_obj = json.loads(decoded_chunk)
                    text = chunk_obj.get("delta", {}).get("text", "")
                    if text.strip():
                        output.append(text)
    except Exception as e:
        print(f'Error processing stream: {str(e)}')
complete_output = ''.join(output)
```

```python
# Exception handling pattern
except botocore.exceptions.ClientError as error:
    if error.response['Error']['Code'] == 'AccessDeniedException':
        print(f"Encountered a permission issue: {error.response['Error']['Message']}")
    else:
        raise error
```

```python
# Stable Diffusion 3.5 Large text-to-image
def create_request_payload(prompt_text, negative_prompt_list, seed):
    payload_data = {"prompt": prompt_text, "negative_prompt": negative_prompt_list, "seed": seed}
    return json.dumps(payload_data)

def execute_model(runtime, payload, model_identifier):
    response = runtime.invoke_model(body=payload, modelId=model_identifier)
    return json.loads(response.get("body").read())

request_payload = create_request_payload(prompt, negative_prompts, seed)
model_identifier = "stability.sd3-5-large-v1:0"
runtime_response = execute_model(bedrock_runtime, request_payload, model_identifier)

image_base64_string = runtime_response["images"][0]
image_1 = Image.open(io.BytesIO(base64.decodebytes(bytes(image_base64_string, "utf-8"))))
image_1.save("data/image_1.png")
```

```python
# LangChain conversational memory across turns (ChatBedrock + RunnableWithMessageHistory)
from langchain_aws.chat_models.bedrock import ChatBedrock
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.messages import SystemMessage, HumanMessage

claude_chat = ChatBedrock(
    model_id="us.anthropic.claude-sonnet-4-20250514-v1:0",
    region_name="us-west-2", credentials_profile_name="mymain",
    beta_use_converse_api=False, max_tokens=500, temperature=1.0, top_p=0.9)

history_store = {}
def get_history(session_id: str):
    if session_id not in history_store:
        history_store[session_id] = InMemoryChatMessageHistory()
    return history_store[session_id]

conversation = RunnableWithMessageHistory(claude_chat, get_session_history=get_history)
config = {"configurable": {"session_id": "user123"}}
first_response = conversation.invoke(
    [SystemMessage(content="You are a top-tier Software Engineering Expert..."),
     HumanMessage(content="What is the best programming language?")], config=config)
followup = conversation.invoke(
    [HumanMessage(content="Why choose that language?")], config=config)
```

- **What it demonstrates**: the full spectrum from raw `invoke_model` calls, to streamed responses with chunk-level error handling, to image generation via Stable Diffusion, to LangChain-managed multi-turn chat memory that avoids manually re-sending the system prompt on every turn.

## Reference Tables
| Endpoint | Plane | Purpose |
|---|---|---|
| `bedrock` | control | model lifecycle: creation, training, deployment |
| `bedrock-runtime` | data | direct inference queries (`InvokeModel`) |
| `bedrock-agent` | control | agent & knowledge-base setup/supervision |
| `bedrock-agent-runtime` | data | invoking agents, querying knowledge bases |

| Mode | When to Use | Pros | Cons |
|---|---|---|---|
| Synchronous | batch tasks, short interactions | simple, one full response | higher perceived latency |
| Streaming | chatbots, real-time, large outputs | lower perceived latency, incremental | more complex client handling |

| HTTP Code | Exception | Cause |
|---|---|---|
| 403 | AccessDeniedException | insufficient IAM permissions |
| 404 | ResourceNotFoundException | resource/model not found |
| 429 | ThrottlingException | rate limit exceeded |
| 429 | ModelNotReadyException | model not fully deployed |
| 504 | ModelTimeoutException | operation timed out |
| 500 | InternalServerException | general AWS-side error |
| 424 | ModelStreamErrorException | streaming data issue |
| 400 | ValidationException | invalid input parameters |
| 402 | ServiceQuotaExceededException | account usage quota exceeded |
| 500 | ModelErrorException | model-side operation problem |

| Claude Parameter | Default | Purpose |
|---|---|---|
| messages | — | input prompt array with role/content |
| temperature | 1.0 | output randomness |
| top_p | 0.999 | nucleus sampling threshold |
| top_k | disabled | candidate token pool size |
| max_tokens | 4096 | output length cap |
| stop_sequences | — | tokens that halt generation |

| Stable Diffusion Model | Function |
|---|---|
| Text to Image | generate images from text descriptions |
| Image to Image | modify existing image via text + `init_image` |
| Image to Image (Masking) | selective region edits, rest transformed per prompt |

## Worked Example
**Food recommender Flask application, end to end**:
1. **Backend setup**: `boto3.client('bedrock-runtime', ...)`, `ConversationBufferMemory()` seeded with a system-style user/AI message pair framing the assistant as a food expert.
2. **Model integration**: `ChatBedrock` with Claude Sonnet 4 (`max_tokens=500, temperature=1.0, top_p=0.9`), wrapped in `RunnableWithMessageHistory` keyed by `session_id` so memory persists across a user's turns without manual transcript management.
3. **Routes**: `@app.route('/')` renders `chat.html`; `@app.route('/suggest_food', methods=['POST'])` reads `user_input` from the JSON body, guards against empty input, invokes the conversation chain, returns JSON.
4. **Front-end**: `chat.html` with a `#chat-window` div, a form (`#chat-form`) posting via Fetch API asynchronously, appending user/AI message divs and auto-scrolling.
5. **Testing**: `curl -X POST http://127.0.0.1:5000/suggest_food -H "Content-Type: application/json" -d '{"user_input": "I love Italian food, any suggestions?"}'`.
6. **Deployment**: HTTPS via ACM+CloudFront, AWS Cognito/IAM auth, ECS/Lambda + ALB for scaling, rate limiting to respect Bedrock quotas.

**Why it works**: it's the smallest possible end-to-end slice — one route, one memory object, one model call — that still demonstrates every layer (auth, model invocation, context retention, UI, deployment) a production app needs, making it a template to extend rather than a toy to discard.

## Key Takeaways
1. Four endpoints split along control-plane/data-plane lines — pick `bedrock-runtime` for direct inference, `bedrock-agent-runtime` for agent/knowledge-base queries.
2. Streaming (`InvokeModelWithResponseStream`) trades implementation complexity for lower perceived latency — verify model support via `GetFoundationModel` first.
3. Prompt caching can cut costs up to 90% and latency up to 85% by caching only the static prompt prefix; cache entries expire after 5 minutes of inactivity.
4. Each of the 10 documented exception types has a distinct cause and remediation — don't collapse them into one catch-all handler.
5. Production cost control is architectural: right-size the model, invoke on-demand (Lambda/EventBridge), rate-limit, and monitor via Cost Explorer/Budgets/CloudWatch.
6. LangChain's `RunnableWithMessageHistory` + `InMemoryChatMessageHistory` eliminates manual transcript management for multi-turn chat context.
7. Flask is the author's pick for production-ready backends (vs. Streamlit for prototyping, React for complex SPAs) due to its lightweight, unopinionated nature pairing well with Python AI integrations.

## Connects To
- **Ch1**: extends the basic `InvokeModel()` mechanics and JSON response structure introduced there.
- **Ch2**: applies the prompt-engineering and parameter-tuning concepts to real API calls (Claude's six inference parameters mirror Ch2's parameter discussion).
- **Ch4**: multimodal models build on the image-generation (Stable Diffusion) foundation laid here.
- **Ch7**: cost optimization and prompt caching here connect to the deeper performance-optimization strategies (Step Functions, CloudFormation) in Ch7.
- **Ch8**: security/IAM practices mentioned here (least privilege, encryption) are expanded fully in the security chapter.
