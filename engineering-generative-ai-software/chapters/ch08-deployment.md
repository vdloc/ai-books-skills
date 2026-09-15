# Chapter 8: Deployment of Generative AI Software

## Core Idea
Cloud deployment of generative AI software happens at four distinct levels (app, capability/model, service/data, infrastructure) each trading off control for convenience; separately, model size has a large, sometimes counterintuitive effect on output consistency and must be benchmarked, not assumed.

## Frameworks Introduced
- **Cloud deployment levels** (Fig 8.1, from highest to lowest abstraction):
  1. **App-level** (web app / container / microservice): you package the whole product; platform handles OS/DB/runtime. Best for chatbots/image generators aimed at end users.
  2. **Capability-level** (expose a trained model via REST/wasm): you provide raw model access (like OpenAI's API); clients build their own apps on top. Best when you want to monetize model access itself.
  3. **Service-level** (expose data, not a trained model): e.g., Google BigQuery — clients train their own models on your data. Best for data-collection businesses (self-driving, image corpora, language corpora).
  4. **Infrastructure-level**: provide a pre-configured OS/compute platform (remote-desktop-like); clients bring their whole stack. Best when clients have competence but need scalable compute.
  - How to choose: match the level to how much control vs. convenience your customer segment needs — SaaS-like end users want app-level; developers/researchers want capability or service level; power users with their own stack want infrastructure level.
- **Develop-deploy-measure cycle** (Fig 8.2): develop → deploy to cloud platform → monitor/measure at runtime → plan next version → repeat. Standard CI/CD-aligned loop applied to GenAI products.
- **Branching strategy for CI/CD** (Fig 8.3): developer branches → feature branch (integration-leader-reviewed PRs) → main branch (clean, always-releasable) → release branch (auto-deployed to cloud on every commit).

## Key Concepts
- **SaaS vs. PaaS split**: SaaS = thin browser client, all logic/data server-side (e.g., Google Docs); PaaS = client runs most of the stack, uses only basic cloud services (e.g., a hosted DB).
- **ONNX Runtime for embedded deployment**: export a model with `torch.onnx.export` (needs dummy input to trace tensors), then load with `onnxruntime.InferenceSession` on a resource-constrained device (Raspberry Pi-class); requires manually handling token-ID↔text translation and the autoregressive generation loop yourself (no HuggingFace `generate()` convenience).
- **NanoLLM (Nvidia Jetson)**: purpose-built small-form-factor GPU computers that can run up to ~70B-parameter models locally (slowly) — the fallback tier above Raspberry-Pi-class embedded devices when task complexity outgrows ONNX Runtime's edge capability.
- **BLEU/ROUGE model-size benchmarking finding**: comparing same-architecture models at different sizes (SmolLM, Qwen3 families) shows *low and inconsistent* similarity scores between size variants (avg. BLEU ≈0.35, ROUGE ≈0.2 for SmolLM; best case 0.74 for Qwen3) — bigger models do **not** reliably converge toward more similar answers; size alone doesn't predict output consistency.

## Mental Models
- Before choosing a deployment level, ask "what am I actually selling — a finished experience, model access, data access, or raw compute?" — the answer maps directly to one of the four cloud deployment levels.
- Don't assume "bigger model = strictly better/more-similar answers" — the book's own BLEU/ROUGE experiments show scores scattered with no clean size-correlation; benchmark your specific candidate models on your specific task before committing to a size tier.

## Anti-patterns
- **Assuming ONNX-exported models can use the same high-level `generate()` APIs as PyTorch/Transformers**: ONNX Runtime deployment requires manually writing the token-by-token generation loop (tokenize → run session → argmax next token → concatenate → repeat until EOS) — budget engineering time for this.
- **Choosing model size using only "bigger is better" intuition**: the chapter's own benchmark shows the second-smallest and largest models in a family can be *less* similar to each other than expected — always run BLEU/ROUGE (or task-specific) comparisons on candidate sizes before locking in a choice.
- **Treating cloud deployment as risk-free**: the book flags that cloud services introduce hacking/data-interception risk during transit and storage — local/embedded deployment is the mitigation when privacy is paramount, at the cost of needing careful model-size selection.

## Worked Example
Deploying a capability (the `decBERTa` model from Ch1) to Azure ML, end to end:
1. `az ml workspace create` → `az ml model create --name declBERTa --path ./decBERTa` → register providers (`Microsoft.MachineLearningServices`, `Microsoft.Cdn`, `Microsoft.PolicyInsights`) → `az ml online-endpoint create`.
2. `conda.yml` declares dependencies (`torch`, `transformers`, `azureml-inference-server-http`); `environment.yaml` builds the image on top of `mcr.microsoft.com/azureml/minimal-ubuntu20.04-py38-cpu-inference`.
3. A scoring script defines `init()` (loads model/tokenizer from `AZUREML_MODEL_DIR`) and `run(raw_data)` (tokenizes input, forward-passes, returns JSON prediction).
4. `deployment.yaml` links model + environment + scoring script + instance type/count; `az ml online-deployment create -f deployment.yaml` deploys it, then `az ml online-endpoint update --traffic "declberta-deployment=100"` routes traffic.
5. Invocation: fetch a bearer token via `az ml online-endpoint get-credentials`, then `curl -X POST https://declberta-endpoint...score -H "Authorization: Bearer <token>" -d '{"text": "int i = <mask>"}'`.
This shows the full "capability-level" pattern: more setup steps than app-level (Section 8.3.1's three-command web-app deploy), but finer-grained reusability for downstream integrators.

## Key Takeaways
1. Pick a cloud deployment level (app / capability / service / infrastructure) based on what you're actually offering customers — each level trades your control for their convenience differently.
2. Link your repo to the cloud provider's CI/CD to get automatic release-branch deployment — manual `az webapp up`-style deployment is fine for prototypes only.
3. ONNX Runtime enables embedded/edge deployment (Raspberry Pi-class) but forces you to hand-roll the autoregressive generation loop — budget for this engineering cost.
4. Nvidia Jetson + NanoLLM is the escalation path when embedded ONNX Runtime devices are insufficient but full cloud deployment is undesirable (privacy/cost).
5. Never assume larger models produce more mutually consistent output — benchmark BLEU/ROUGE (or a task-specific metric) across your actual candidate model sizes; the book's own data shows inconsistent, non-monotonic results.

## Connects To
- **Ch5**: cloud deployment levels are a concrete instantiation of the microservice/client-server architecture discussed there.
- **Ch6**: ONNX portability techniques from Ch6 are what make embedded deployment in this chapter possible.
- **Ch9**: the API design conventions (versioning, response codes) apply directly to any capability-level deployment from this chapter.
