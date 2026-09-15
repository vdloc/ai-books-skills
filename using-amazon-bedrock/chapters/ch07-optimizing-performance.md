# Chapter 7: Optimizing Performance for Foundational Models

## Core Idea
Optimizing foundational-model performance spans four layers — memory/compute efficiency (quantization, pruning, parallelism), evaluation/refinement (RL, automatic + human model evaluation), distributed computing (data/model/hybrid parallelism, elastic scaling, fault tolerance), and orchestration (Step Functions + Lambda + CloudFormation) — illustrated end-to-end through a book-review-generation pipeline.

## Frameworks Introduced
- **Memory optimization techniques**: Pruning (remove unimportant weights — smaller model, possible accuracy drop), Quantization (lower numerical precision FP16/INT8 — big memory/speed gains, slight accuracy loss), Mixed Precision (FP16 forward/back pass + FP32 for some layers — balances speed/accuracy, more complex).
  - How: pruning for models with rarely-used parameters; quantization for inference-heavy tasks at scale; mixed precision when partial precision is acceptable for training/inference.
- **Chinchilla scaling laws**: training tokens should scale ~20:1 with model parameters to maximize performance per unit of compute.
  - When to use: selecting model size and dataset volume for compute-optimal training.
- **Distributed computing spectrum**: Data Parallelism (same model, different data shards, aggregate results) → Model Parallelism (split the model itself across nodes when it exceeds single-node memory) → Hybrid Parallelism (split both simultaneously, for large data AND large models).
  - How: data parallelism when the dataset is the bottleneck; model parallelism when the model itself doesn't fit in memory; hybrid when both are true.
- **RL policy initialization (three methods)**: Random initialization (explore without prior knowledge), Pretrained-model-derived policy, Behavior cloning (mimic expert/another agent via supervised learning).
- **Three RL approaches for content refinement**: Learning from Demonstrations (train on high-quality human examples), Learning from Preferences (use user ratings/feedback as reward signal), Learning from Critiques (retrain based on direct human critique/correction).

## Key Concepts
- **Continuous batching**: dynamically fills GPU with new requests as old ones finish — maximizes utilization, minimal idle time.
- **Speculative batching**: a smaller draft model generates candidate outputs verified by the larger model — fast when draft is correct, costly recomputation when wrong.
- **Sparse attention**: focuses computation on relevant input parts (based on attention scores/token importance) rather than the full sequence — reduces compute for long-text tasks.
- **Prompt caching**: caches stable prompt prefixes (system instructions, few-shot examples) — up to 90% token cost reduction, 85% latency reduction; 5-minute sliding TTL reset on each hit; tracked via `CacheReadInputTokens`/`CacheWriteInputTokens` CloudWatch metrics.
- **Elastic scaling**: automatic compute scale-up/down based on real-time demand — cost-effective during demand spikes/lulls.
- **Checkpointing & replication**: fault-tolerance mechanisms — checkpointing saves periodic state snapshots (revert instead of restart on failure); replication duplicates processes/data across nodes so failures don't halt service.
- **Model evaluation metrics**: Accuracy (RWK — real-world knowledge correctness), Robustness (Word Error Rate on perturbed prompts), Toxicity (detoxify/RealToxicityPrompts), BERTScore (embedding-based F1 vs. reference), NLP-F1 (precision/recall for Q&A).

## Mental Models
- Treat **memory requirements as a back-of-envelope calculation**: each parameter needs ~2 bytes at 16-bit precision, so a 1B-parameter model needs ~2GB, a 175B-parameter model needs ~350GB (parameters alone, excludes activations/gradients) — use this to sanity-check infrastructure planning before deployment.
- **Automatic evaluation first, human evaluation for nuance**: start with automatic evaluations (fast, objective — accuracy/robustness/toxicity) and escalate to human worker evaluation only for subjective qualities (tone, style, "does this feel natural") that automatic metrics can't capture.
- **RL loop is state → action → reward → next state**: an agent (the generation model) takes actions (e.g., "add character analysis"), the environment (user feedback/ratings) returns a reward, and the policy updates to maximize cumulative reward over episodes — this closed loop is the mental model for any Bedrock RL-based refinement.
- **Step Functions as prompt-chaining orchestrator**: each state is one prompt/task (overview → plot → themes → style → synthesis); outputs from earlier states feed into later ones via `ResultPath`, letting complex generation be broken into debuggable, modular, retriable stages.

## Anti-patterns
- **Applying full parallelism complexity to workloads that don't need it** — data parallelism alone suffices when the dataset (not the model) is the bottleneck; don't reach for hybrid parallelism by default.
- **Skipping automatic evaluation and going straight to human evaluation** — automatic evaluation is faster and covers objective metrics; use human evaluation only when automatic results reveal insufficiency, not as the default first pass.
- **Caching dynamic user input** — like Ch3, cache only stable prefixes (system prompts, few-shot examples); caching per-request dynamic content wastes the cache and risks stale reuse.
- **Ignoring CORS and IAM configuration for evaluation datasets stored in S3** — prompt datasets referenced in evaluation jobs need correctly configured bucket access or the evaluation job fails silently/partially.

## Code Examples
```json
// Step Functions state machine invoking Bedrock directly (no Lambda needed)
{
  "Comment": "State machine for invoking Bedrock using the Messages API",
  "StartAt": "Bedrock InvokeModel",
  "States": {
    "Bedrock InvokeModel": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0",
        "Body": {
          "anthropic_version": "bedrock-2023-05-31",
          "messages": [{"role": "user", "content": "Provide a brief book review on The Great Gatsby."}],
          "max_tokens": 300, "temperature": 0.5, "top_p": 0.9
        }
      },
      "End": true,
      "OutputPath": "$.Body.content[0].text"
    }
  }
}
```

```yaml
# CloudFormation: Lambda + Step Functions for chained book review generation
BookReviewStateMachine:
  Type: AWS::StepFunctions::StateMachine
  Properties:
    RoleArn: !GetAtt LambdaExecutionRole.Arn
    DefinitionString: !Sub |
      {
        "StartAt": "Create Brief Overview",
        "States": {
          "Create Brief Overview": {
            "Type": "Task", "Resource": "${MyLambdaFunction.Arn}",
            "Parameters": {"prompt": "Provide a brief overview..."},
            "ResultPath": "$.Create_Brief_Overview", "Next": "Provide Plot Details"
          },
          "Synthesize the Review": {
            "Type": "Task", "Resource": "${MyLambdaFunction.Arn}",
            "Parameters": {
              "prompt.$": "States.Format('Synthesize a complete review... Overview: {}\n\nPlot: {}\n\n...', $.Create_Brief_Overview.body[0].text, $.Provide_Plot_Details.body[0].text)"
            },
            "End": true
          }
        }
      }
```

```python
# Lambda handler invoked by each Step Functions stage
def handler(event, context):
    client = boto3.client('bedrock-runtime')
    prompt = event['prompt']
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 500,
        "messages": [{"role": "user", "content": [{"type": "text", "text": prompt}]}]
    })
    response = client.invoke_model(
        modelId='anthropic.claude-3-sonnet-20240229-v1:0',
        contentType='application/json', accept='application/json', body=body)
    model_output = json.loads(response['body'].read())
    return {'statusCode': 200, 'body': model_output.get('content', 'No review generated.')}
```

```jsonl
// Custom prompt dataset format (automatic evaluation)
{"referenceResponse": "Jazz", "category": "Music Genres", "prompt": "New Orleans is widely regarded as the birthplace of"}
// Claude-specific format (Human/Assistant structure required)
{"prompt": "Human: Can you explain mindful meditation? Assistant:", "category": "Wellness", "referenceResponse": "Mindful meditation is..."}
```

- **What it demonstrates**: direct Bedrock invocation from Step Functions (no Lambda intermediary needed for simple cases), a full CloudFormation-defined multi-stage prompt-chaining pipeline (Lambda + Step Functions), and the dataset formatting conventions for automatic model evaluation (generic vs. Claude-specific Human/Assistant structure).

## Reference Tables
| Technique | Description | Pros | Cons |
|---|---|---|---|
| Pruning | remove unimportant weights | smaller model/memory | possible accuracy drop |
| Quantization | lower precision (FP16/INT8) | big memory/speed gains | slight accuracy loss |
| Mixed Precision | FP16 + selective FP32 | balance speed/accuracy | more complex |

| Batching Method | How It Works | Pros | Cons |
|---|---|---|---|
| Continuous | fills GPU with new requests as old finish | max GPU utilization | variable requests → partial batching |
| Speculative | draft model proposes, main model verifies | reduces main-model load when correct | high recompute cost if draft wrong |

| Metric | Meaning | Calculation |
|---|---|---|
| Accuracy (RWK) | real-world knowledge correctness | compares output to reference (T-REx) |
| Robustness | resilience to small input changes | Word Error Rate on perturbed prompts |
| Toxicity | harmful/offensive content | detoxify / RealToxicityPrompts |
| BERTScore | closeness to reference summaries | embedding-based F1 |
| NLP-F1 | Q&A precision/recall | exact/partial match vs. reference |

| Task Type | Built-in Datasets | Key Metrics |
|---|---|---|
| Text Generation | BOLD, RealToxicityPrompts, TREX, WikiText2 | Accuracy (RWK), Robustness (WER), Toxicity |
| Summarization | Gigaword | BERTScore, Robustness, Toxicity |
| Q&A | BoolQ, Natural Questions, TriviaQA | NLP-F1, F1/deltaF1, Toxicity |
| Classification | (consult docs) | Accuracy, Robustness |

## Worked Example
**Book Review Generation Pipeline** (illustrates the full chapter end-to-end):
1. **Memory/compute concerns**: generating a review involves multiple sub-tasks (overview, plot, themes, style critique, synthesis) — each consuming memory for intermediate outputs; Bedrock applies pruning/quantization automatically for managed deployments.
2. **RL refinement**: Episode 1 — agent generates a bare summary, gets reward -1 (too shallow). Episode 2 — adds character analysis, reward +2, cumulative reward 1. Episode 3 — adds thematic analysis, reward +3, cumulative reward 4 — demonstrating iterative quality improvement via feedback.
3. **Evaluation**: automatic evaluation job on Claude 3 Sonnet using RealToxicityPrompts (toxicity) + TREX (accuracy/robustness) datasets; human evaluation job on Llama 3.1 405B using Coherence and Accuracy Likert-scale (1-5) ratings from a "Bring Your Own Work Team."
4. **Distributed computing**: data parallelism processes different book reviews across nodes concurrently; model parallelism splits the LLM itself (e.g., one segment for plot, another for style) when the model exceeds single-node memory.
5. **Orchestration**: a Step Functions state machine chains five Lambda-invoked Bedrock calls — Overview → Plot Details → Themes → Style Critique → Synthesis (final stage combines all four prior outputs into a cohesive five-paragraph review via `States.Format`).
6. **Infrastructure as code**: the entire pipeline (Lambda function, IAM roles, Step Functions state machine) is defined in a single CloudFormation template, deployed via `aws cloudformation create-stack`, with `Outputs` exposing the Lambda and Step Function ARNs for external integration.

**Why it works**: using ONE running example (book reviews) across memory optimization, RL, evaluation, parallelism, and orchestration shows how these normally-separate concerns compose into a single production pipeline, rather than reading as disconnected technique catalog entries.

## Key Takeaways
1. Memory requirements scale predictably (~2 bytes/parameter at 16-bit precision) — use this for infrastructure planning before deployment.
2. Start with automatic model evaluation (fast, objective metrics); escalate to human worker evaluation only for subjective qualities automatic metrics can't capture.
3. Choose the parallelism strategy that matches your actual bottleneck: data parallelism for large datasets, model parallelism for models exceeding node memory, hybrid when both are true.
4. Prompt caching (5-min sliding TTL, cache only stable prefixes) can cut costs up to 90% and latency up to 85% — monitor via `CacheReadInputTokens`/`CacheWriteInputTokens`.
5. Step Functions can invoke Bedrock directly (no Lambda needed) for simple single-call workflows, or orchestrate Lambda-mediated multi-stage prompt chains for complex generation pipelines.
6. CloudFormation (or CDK/Terraform) ensures consistent, auditable, rollback-capable infrastructure deployment for generative AI workloads — treat infrastructure as code, not manual console clicks, for anything beyond one-off experiments.
7. Fault tolerance in distributed LLM workloads relies on checkpointing (periodic state snapshots) and replication (duplicated processes/data) to survive node failures without full restarts.

## Connects To
- **Ch2**: extends prompt caching concepts introduced in the prompt-engineering chapter.
- **Ch3**: extends the cost-optimization and prompt-caching foundations from the Bedrock API chapter.
- **Ch5**: RL-based refinement here parallels but differs from fine-tuning — RL adapts behavior via reward signals, fine-tuning adapts weights via labeled examples.
- **Ch8**: security/access-control considerations for the Step Functions/Lambda/CloudFormation pipeline are covered next.
- **Ch9**: the Step Functions + Lambda + CloudFormation orchestration pattern here is the direct foundation for the end-to-end application architectures in Ch9.
