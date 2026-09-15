# Chapter 5: Fine-Tuning Foundational Models on AWS

## Core Idea
Fine-tuning adapts a pretrained model's weights to a specific domain/task via labeled JSONL datasets — use it when you need specialized jargon handling, rigid/consistent output formats, or lower inference cost from shorter prompts; use prompt engineering or RAG instead when pretrained knowledge already covers the domain, iteration speed matters, or you lack sufficient domain data.

## Frameworks Introduced
- **Fine-tune vs. prompt-engineer decision framework**: Fine-tune for Specialized Domains (legal/medical/technical jargon), Repeated/Consistent Outputs (fixed schema/classification tags), or Inference Cost Savings (shorter prompts at scale). Stick with prompt engineering for General Knowledge Tasks, Rapid Iteration needs, or Limited Data availability.
  - How: before committing to fine-tuning, check if the failure mode is "wrong knowledge" (fine-tune/RAG) vs. "wrong instruction-following" (prompt engineer first, cheaper to fix).
- **AWS deployment complexity/control spectrum**: Bedrock (managed, minimal setup, less control) → SageMaker (moderate complexity, prebuilt algorithms + custom code, more control) → EKS (full infrastructure control, highest complexity).
  - How: use Bedrock for quick fine-tuning with minimal infra management (content generation, summarization, dialogue systems); use SageMaker for full ML lifecycle control and complex custom pipelines; use EKS only when you need to self-manage everything.
- **Seven-step fine-tuning setup process**: Install libraries → Import libraries/init variables → Create IAM role & policy → Write instruction & iterate over data → Define `data_transform` & process partitions → Convert & upload datasets → Create fine-tuning job (+ provisioned throughput + monitor + invoke + clean up).

## Key Concepts
- **Epoch**: one full pass through the training dataset; more epochs can improve fit but risks overfitting (monitor via F1-score, confusion matrix).
- **Batch size**: number of examples per parameter-update step; smaller = faster convergence but more variable updates, larger = more stable but slower/more memory.
- **Learning rate**: controls magnitude of weight updates per step; fine-tuning typically uses a conservative rate (e.g., 0.0001) to refine without erasing pretrained knowledge.
- **Learning rate warmup steps**: gradually ramps the learning rate up from small to target value over N initial steps, preventing early-training divergence.
- **Instruction-based fine-tuning**: trains the model to follow specific instruction/output pairs — improves task adherence (e.g., Q&A, structured generation) beyond raw domain knowledge transfer.
- **Provisioned model throughput**: the *only* way to serve a custom/fine-tuned model on Bedrock (on-demand pricing doesn't apply); billed by model units × duration — can cost thousands/month if left running, must be explicitly deleted.
- **PEFT (Parameter-Efficient Fine-Tuning)**: techniques like LoRA, adapters, prompt tuning that update <1% of model parameters instead of full weights — cuts GPU time/storage while enabling multiple tuned variants without duplicating full checkpoints.
- **Mixed-precision training (FP16/BF16)**: halves memory footprint vs. FP32, accelerates GPU compute with minimal numerical-stability impact; combine with ZeRO-stage optimizers to fine-tune 30B+ models on modest GPU setups.

## Mental Models
- Treat fine-tuning as **"lock in a format/domain at the parameter level"** rather than "make the model smarter" — it's most valuable when you need *consistency* (exact JSON schema, fixed classification tags) that prompting alone can't guarantee run-to-run.
- The **dataset is the product**: quality/relevance/size of the JSONL dataset directly bounds fine-tuning outcomes — 500-1,000 examples for simple tasks, 10,000+ for complex ones, with strict format consistency (uniform prompt/completion style) and no dataset-format inconsistency (which silently breaks training jobs).
- **Cost discipline is structural, not optional**: provisioned throughput is the only serving path for custom models and bills continuously — the author explicitly frames "clean up resources" as a required last step, not an afterthought.

## Anti-patterns
- **Fine-tuning when pretrained knowledge already suffices** — wastes data-curation and compute effort where a well-crafted prompt would solve it faster and cheaper.
- **Skipping JSONL format validation** — missing `prompt`/`completion` keys, wrong encoding (must be UTF-8), or malformed syntax silently breaks fine-tuning jobs; validate before submitting.
- **Leaving provisioned throughput running after testing** — this is explicitly called out as capable of costing "thousands of U.S. dollars per month" if not decommissioned; always tear down `provisioned_model_id`, S3 buckets, and IAM roles/policies after the exercise.
- **Full-model fine-tuning by default when PEFT would suffice** — updating all weights costs far more GPU time/storage than LoRA/adapters/prompt tuning for equivalent task performance in most cases.

## Code Examples
```python
# Creating a fine-tuning job on Bedrock (Llama 3.3 + GovReport dataset)
hyper_parameters = {
    "epochCount": "2",
    "batchSize": "1",
    "learningRate": "0.00005",
}
training_data_config = {"s3Uri": s3_train_uri}
validation_data_config = {"validators": [{"s3Uri": s3_validation_uri}]}
output_data_config = {"s3Uri": f's3://{bucket_name}/outputs/output-{custom_model_name}'}

bedrock.create_model_customization_job(
    customizationType="FINE_TUNING",
    jobName=customization_job_name,
    customModelName=custom_model_name,
    roleArn=customization_role,
    baseModelIdentifier="meta.llama3-3-70b-instruct-v1:0",
    hyperParameters=hyper_parameters,
    trainingDataConfig=training_data_config,
    validationDataConfig=validation_data_config,
    outputDataConfig=output_data_config
)
```

```python
# Provisioning throughput to serve the custom model (required — no on-demand for custom models)
provisioned_model_id = bedrock.create_provisioned_model_throughput(
    modelUnits=1,
    provisionedModelName='provisioned_model_fine_tuning_1',
    modelId='<INSERT_CUSTOM_MODEL_ID_HERE>'
)['provisionedModelArn']
```

```python
# Dataset formatting: JSONL prompt/completion pairs
instruction = '''Below is an instruction which describes a task, paired with an input which will provide further context. Write a response that appropriately completes the request. instruction: Summarize the report provided below.

input: '''

datapoints_train = []
for data in dataset['train']:
    temp_dict = {}
    temp_dict['prompt'] = instruction + data['report']
    temp_dict['completion'] = 'response:\n' + data['summary']
    datapoints_train.append(temp_dict)
```

```python
# Filtering data points by length and truncating to a target sample size
def data_transform(data_points, num_data, max_data_length):
    lines = []
    for data in data_points:
        if len(data['prompt'] + data['completion']) <= max_data_length:
            lines.append(data)
    random.shuffle(lines)
    lines = lines[:num_data]
    return lines

train = data_transform(datapoints_train, 5000, 10000)
```

```python
# Cleanup — always run this after testing to avoid runaway provisioned-throughput costs
bedrock.delete_provisioned_model_throughput(provisionedModelId=provisioned_model_id)
objects = s3_client.list_objects(Bucket=bucket_name)
if 'Contents' in objects:
    for obj in objects['Contents']:
        s3_client.delete_object(Bucket=bucket_name, Key=obj['Key'])
s3_client.delete_bucket(Bucket=bucket_name)
iam.detach_role_policy(RoleName=role_name, PolicyArn=policy_arn)
iam.delete_role(RoleName=role_name)
```

- **What it demonstrates**: the complete fine-tuning lifecycle — job creation with hyperparameters, mandatory provisioned throughput for serving, dataset formatting into prompt/completion JSONL pairs, length-filtering, and the mandatory cleanup sequence to avoid runaway billing.

## Reference Tables
| Dataset Type | Use Case | Required Format |
|---|---|---|
| Text-to-Text | Summarization, translation | JSONL with `prompt` and `completion` |
| Text-to-Image | Image generation | JSONL with `image-ref` (S3 URI) and `caption` |

| Hyperparameter | Effect | Example Value |
|---|---|---|
| epochCount | passes through full dataset | 2-3 |
| batchSize | examples per update | 1-16 |
| learningRate | update magnitude | 0.00001-0.0001 (conservative) |
| warmup steps | gradual LR ramp-up | ~500 |

| Platform | Complexity | Control | Best For |
|---|---|---|---|
| Bedrock | Low | Low | quick fine-tune, minimal infra mgmt |
| SageMaker | Medium | Medium | full ML lifecycle, custom code/algorithms |
| EKS | High | High | fully self-managed custom deployments |

## Worked Example
**Fine-tuning Llama 3.3 on the GovReport Summarization dataset** (full pipeline):
1. **Setup**: install boto3/awscli/botocore, langchain, jsonlines, datasets, pandas, matplotlib; restart kernel.
2. **IAM & S3**: create an S3 bucket (`bedrock-fine-tuning-custom-{region}-{account}`), an IAM role (`AmazonBedrockFineTuningCustomRole`) trusted by `bedrock.amazonaws.com`, and an access policy scoped to that bucket (least privilege).
3. **Dataset prep**: load `ccdv/govreport-summarization` (17,517 train / 973 validation / 973 test rows), prepend an instruction template to each report, format as `{"prompt": ..., "completion": ...}`, filter to ≤10,000 characters, shuffle and cap at 5,000 train / 999 validation / 10 test examples, convert to JSONL, upload to S3.
4. **Fine-tuning job**: `bedrock.create_model_customization_job` with `base_model_id="meta.llama3-3-70b-instruct-v1:0"`, hyperparameters `epochCount=2, batchSize=1, learningRate=0.00005`.
5. **Provisioned throughput**: once job status is "Completed," provision throughput (`modelUnits=1`) — the only way to serve a custom model.
6. **Invoke & evaluate**: send a test prompt (`max_gen_len=300, temperature=0.5, top_p=0.5`) — the fine-tuned model correctly summarizes a FOIA-policy report, contrasting FBI (mail-only) vs. CDC (electronic-only) request handling, cut off at the 200-token generation limit.
7. **Cleanup**: delete provisioned throughput, empty and delete the S3 bucket, detach and delete the IAM role/policy.

**Why it works**: it's a complete, cost-aware reference implementation — every AWS resource created in step 2-5 has an explicit teardown in step 7, modeling the discipline required when provisioned throughput bills continuously.

## Key Takeaways
1. Fine-tune for specialized domains, rigid output consistency, or inference cost savings; prefer prompt engineering for general knowledge, rapid iteration, or limited data.
2. The four key hyperparameters — epoch count, batch size, learning rate, warmup steps — directly control training dynamics; fine-tuning typically uses conservative learning rates to avoid erasing pretrained knowledge.
3. Bedrock, SageMaker, and EKS trade simplicity for control along a clear spectrum — pick based on how much infrastructure management your team wants to own.
4. Datasets must be JSONL-formatted (prompt/completion for text-to-text, image-ref/caption for text-to-image); validate format before submission to avoid silent training failures.
5. Provisioned throughput is mandatory for serving custom/fine-tuned models — it's the primary cost risk in this chapter and must be explicitly torn down after use.
6. PEFT techniques (LoRA, adapters, prompt tuning) update <1% of parameters, dramatically cutting training cost/time versus full-model fine-tuning for most use cases.
7. Cleanup (provisioned throughput, S3, IAM) is not optional — it's presented as the final, mandatory step of every fine-tuning workflow.

## Connects To
- **Ch1**: extends the model-selection and provisioned-throughput-vs-on-demand pricing concepts introduced there.
- **Ch4**: multimodal fine-tuning (text-to-image datasets) builds on the image-generation groundwork from the multimodal chapter.
- **Ch6**: fine-tuning is positioned as an alternative to RAG — "fine-tuning is the best option when you require deeper model adaptation that can't be solved by providing external data."
- **Ch7**: performance evaluation metrics (perplexity, loss) mentioned here are covered in depth in the optimization chapter.
- **Ch9**: CloudFormation/CDK for automated resource teardown (mentioned here for cleanup) is expanded in the end-to-end applications chapter.
