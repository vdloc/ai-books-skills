# Chapter 2: Setting Up Your Environment

## Core Idea
Before writing any Bedrock code, you need three things configured correctly: an AWS account with Bedrock model access requested, an IAM user/role scoped to least-privilege Bedrock policies, and a working SDK/CLI setup — the Model Playground is the fastest way to validate all three before committing to code.

## Frameworks Introduced
- **Essential IAM Policies for Amazon Bedrock**: the author's minimum policy set for a working Bedrock dev environment.
  - When to use: setting up any new AWS account/IAM user for Bedrock work, including CI/CD service roles.
  - How: attach `AmazonBedrockFullAccess` (or a scoped-down custom policy for production) plus supporting policies for the surrounding stack — `AmazonS3FullAccess` (for storing prompts/outputs/fine-tuning data), `AWSLambdaFullAccess` (for serverless invocation), `CloudWatchLogsFullAccess` (for observability), and `IAMReadOnlyAccess` for auditing. Production environments should narrow these to resource-scoped policies rather than the *FullAccess managed policies used for learning.
- **Playground-first validation workflow**: don't debug SDK auth issues by iterating on code.
  - When to use: right after requesting model access, before writing the first SDK call.
  - How: (1) open the Bedrock console → Model Playground, (2) request access to target foundation models (not automatic — must be explicitly granted per model), (3) run a prompt in Text/Chat/Image playground mode and confirm output, token counts, and latency look right, (4) only then move to Boto3/CLI, reusing the same model ID and region.

## Key Concepts
- **IAM (Identity and Access Management)**: AWS's access-control service; Bedrock access is governed by IAM policies attached to users/roles, not a separate Bedrock-specific auth system.
- **Model access request**: a Bedrock-specific step — even with full IAM permissions, each foundation model must be individually granted access in the console (or via API) before it can be invoked; this is a common first-time blocker.
- **Boto3**: the AWS SDK for Python; `boto3.client('bedrock')` is the control-plane client (list/manage models), `boto3.client('bedrock-runtime')` is the data-plane client used for actual inference calls — a distinction the book treats as important to avoid confusing "administrative" and "runtime" API surfaces.
- **Model Playground (Text/Chat/Image)**: the console UI for interactively testing prompts against a chosen model without writing code; supports single-shot and multi-turn (Chat mode) modes and exports a working API request (JSON) once you're satisfied with a prompt.
- **Provisioned vs on-demand access**: referenced here in the context of cost — Bedrock is excluded from the AWS Free Tier, so token usage costs apply from the first call, unlike some other AWS ML services (e.g. SageMaker's free tier).

## Mental Models
- Treat the **bedrock vs bedrock-runtime client split** as a permissions and mental-model boundary: control-plane operations (listing/managing model access) use one client, inference calls use another — mixing them up is a common source of `AccessDenied`-looking errors that are actually "wrong client for this operation."
- Use the **Playground's "Export as JSON"** feature as your first code scaffold — it generates a verified-working API request shape, removing guesswork about parameter names/structure when writing the equivalent Boto3 call.
- Think of **IAM policy scoping as a dial, not a switch**: start broad (*FullAccess) to unblock learning, then narrow to per-resource/per-model policies before anything touches production — the book presents this as a deliberate progression, not a one-time decision.

## Anti-patterns
- **Skipping the explicit model-access request step**: assuming IAM `AmazonBedrockFullAccess` is sufficient — Bedrock still requires per-model access grants in the console, a frequent first-run failure mode the book calls out explicitly.
- **Using the root AWS user for day-to-day Bedrock development**: the book's security best-practice section is explicit that the root user should be reserved for account-level tasks only, with an IAM user (or role) created immediately for all Bedrock work.
- **Assuming Bedrock is covered by AWS Free Tier**: unlike Amazon SageMaker (which has a free tier), Bedrock usage is billed from the first token — budget/cost alerts should be set up in this chapter, not deferred.

## Code Examples
```python
import boto3
from botocore.exceptions import ClientError, NoCredentialsError

# Specify region explicitly
region_name = 'us-east-1'
try:
    # Create Bedrock client (control plane — list/manage models)
    bedrock = boto3.client(
        service_name='bedrock',
        region_name=region_name
    )
    # List foundation models
    ...
except (ClientError, NoCredentialsError) as e:
    ...

# Separate client for actual inference calls (data plane)
bedrock_runtime = boto3.client(service_name='bedrock-runtime')
```
- **What it demonstrates**: the book's canonical two-client pattern — `bedrock` for control-plane/admin operations (e.g. listing available models), `bedrock-runtime` for the actual inference calls used in every later chapter's code samples.

## Reference Tables
| Setup Step | Purpose |
|---|---|
| Create/verify AWS account | Root access to provision resources |
| Create IAM user + attach policies | Least-privilege access (AmazonBedrockFullAccess + supporting S3/Lambda/CloudWatch/IAM policies) |
| Request Bedrock model access (console) | Per-model opt-in required even with full IAM permissions |
| Configure AWS CLI credentials | Local dev / scripting access via Access Key ID + Secret |
| Install AWS SDK (Boto3 for Python, or JS/Java/.NET/Go SDKs) | Programmatic invocation |
| Explore Model Playground (Text/Chat/Image) | Validate model access and prompt shape before writing code |

## Worked Example
The book walks through a first console inference: navigate to the Bedrock console → Model Playground → select a pretrained model you've been granted access to → enter a prompt (e.g. "Explain quantum computing in simple terms" or, for the image playground, "Draw a cat playing guitar") → review the output alongside input/output token counts and latency → use "Export as JSON" to capture the exact API request shape, which becomes the template for the first Boto3 `invoke_model` call in later chapters.

## Key Takeaways
1. Bedrock access requires two separate opt-ins: IAM permissions AND an explicit per-model access request in the console — missing the second is the most common first-time blocker.
2. Use `boto3.client('bedrock')` for control-plane operations and `boto3.client('bedrock-runtime')` for actual inference — conflating them causes confusing permission errors.
3. Validate any new setup in the Model Playground before writing SDK code; use its JSON export as a working request template.
4. Bedrock has no free tier — token costs apply immediately, so set up cost monitoring/alerts as part of initial setup, not as an afterthought.
5. Start with broad IAM policies to unblock learning, but treat narrowing to least-privilege as a required step before production, not optional hardening.

## Connects To
- **Ch1**: applies the model-selection guidance from Ch1 when choosing which model(s) to request access to first.
- **Ch4**: the Boto3 `bedrock-runtime` client pattern set up here is used directly to build the first Generative AI solution.
- **Ch6**: IAM/cost setup here is revisited under production scaling and cost-optimization concerns.
