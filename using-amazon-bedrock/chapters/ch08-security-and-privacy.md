# Chapter 8: Security and Privacy for Deploying Generative AI Architectures on AWS

## Core Idea
Securing Bedrock deployments spans the Shared Responsibility Model (AWS secures the cloud infrastructure, you secure your data/access/configuration) across five layers: security scoping (which of 5 usage scopes you're in), threat modeling (STRIDE), monitoring (CloudWatch/CloudTrail/GuardDuty), access control (IAM policies/ABAC/temporary credentials), and guardrails (denied topics, content filters, contextual grounding) — each layer requiring deliberate configuration, none of it automatic.

## Frameworks Introduced
- **Generative AI Security Scoping Matrix (5 scopes)**: Scope 1 (Consumer App — third-party AI via API, no data access) → Scope 2 (Enterprise App — vendor-integrated AI) → Scope 3 (Pretrained Models — build on Bedrock FMs) → Scope 4 (Fine-tuned Models — customize with business data) → Scope 5 (Self-Trained Models — full proprietary control).
  - How: each scope demands different treatment across five disciplines — Governance/Compliance, Legal/Privacy, Risk Management, Controls, Resilience. Higher scopes (3-5) require deeper threat modeling and IAM control; lower scopes (1-2) focus on vendor terms and data-sharing policy.
- **STRIDE threat modeling framework**: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege.
  - When to use: systematically identifying threats to any Bedrock deployment — walk each category against your architecture (e.g., "can an attacker spoof identity to invoke our agent?").
  - How: for each category, write a threat statement (actor + vulnerable asset + attack method + impact), assign priority (High/Medium/Low), and design a targeted mitigation.
- **Six IAM policy types for Bedrock**: Identity-Based Policies, Policy Actions & Resources (ARN-scoped), Policy Condition Keys (context-based restrictions), ABAC (tag-based access), Temporary Credentials (STS-issued, time-bound), Principal Permissions & Service Roles (what Bedrock itself is allowed to do on your behalf).

## Key Concepts
- **Bedrock Guardrails**: configurable safety layer with five components — content filters (hate/sexual/violence thresholds: LOW/MEDIUM/HIGH), denied topics (natural-language-described restricted subjects), word filters, sensitive information filters (PII detection), contextual grounding checks (factual alignment + relevance scoring).
- **Contextual grounding**: scores a response on two axes — grounding (factually aligned with a reference source) and relevance (actually answers the query) — a response can be grounded-but-irrelevant, relevant-but-ungrounded, both, or neither; thresholds (0-0.99) configurable per use case.
- **Model inversion attack**: adversary reconstructs sensitive training data by systematically querying the model (e.g., "Generate text about patient X's medical history" against a model fine-tuned on medical records). Mitigate with differential privacy, query-rate limiting, anomaly monitoring.
- **Membership inference attack**: adversary determines whether a specific data point was in the training set by comparing model responses. Mitigate with regularization/cross-validation (reduce overfitting), access controls, federated/split learning for sensitive projects.
- **ABAC (Attribute-Based Access Control)**: grants access based on matching tags (e.g., `Department=Engineering`) rather than explicit identity lists — scales better in dynamic environments than per-user policies.
- **LLMOps**: MLOps extended for LLM-specific lifecycle concerns (continuous retraining on new threat patterns, drift detection, automated security scanning in fine-tuning pipelines).

## Mental Models
- Treat security as a **shared responsibility split**: AWS secures the infrastructure (physical, network, hardware); you secure your data, access configuration, and application logic — nothing about IAM policies, guardrail configuration, or encryption key management happens automatically just because you're "on Bedrock."
- **Guardrail strictness is a trade-off dial, not a binary**: stricter thresholds block more unwanted content but also increase false positives and degrade legitimate user experience — there's no universal "right" threshold, only the one that fits your risk tolerance (e.g., finance context → very strict on investment advice).
- **Blocked-message phrasing itself is a security surface**: telling a user exactly why they were blocked ("blocked for insider trading discussion") leaks information attackers can use to refine bypass attempts — use generic messaging ("we can't proceed with this request") instead.
- Think of the **STRIDE categories as a checklist to run against your specific architecture**, not an abstract taxonomy — the chapter's trading-platform walkthrough shows each category mapped to a concrete, plausible attack on that specific system.

## Anti-patterns
- **Attaching `AmazonBedrockFullAccess` to an agent "just for development"** — explicitly called out as a significant risk; it grants agents the ability to modify other agents or access sensitive knowledge bases far beyond their actual task scope. Always scope agent IAM roles to the specific actions needed (`bedrock:InvokeAgent`, `bedrock:PrepareAgent`).
- **Providing detailed reasons in blocked-content messages** — leaks information that helps attackers craft bypass prompts; use minimal, generic blocked messaging instead.
- **Treating guardrails as a substitute for RAG/data-grounding, or vice versa** — guardrails constrain *what the model is allowed to say*; RAG/contextual grounding improves *factual accuracy of what it does say*. Both are needed for high-stakes domains, not one instead of the other.
- **Skipping threat modeling until after deployment** — STRIDE and the Security Scoping Matrix are meant to be applied during architecture design, not retrofitted; the trading-platform example explicitly walks through applying STRIDE to a system BEFORE full production rollout.
- **Ignoring emerging threats specific to AI (model inversion, membership inference) in favor of only traditional cybersecurity threats (phishing, generic data leakage)** — AI systems have unique attack surfaces that standard security checklists miss.

## Code Examples
```python
# Defining and creating a Bedrock guardrail (deny investment advice, filter hate/sexual content)
topic_policy_config = {
    'topicsConfig': [{
        'name': 'Investment Advice',
        'definition': 'Any financial investment advice',
        'examples': ['Should I invest in stocks?', 'Is real estate a good investment?'],
        'type': 'DENY'
    }]
}
content_policy_config = {
    'filtersConfig': [
        {'type': 'HATE', 'inputStrength': 'HIGH', 'outputStrength': 'HIGH'},
        {'type': 'SEXUAL', 'inputStrength': 'MEDIUM', 'outputStrength': 'MEDIUM'}
    ]
}
response = client.create_guardrail(
    name='ExampleGuardrail',
    description='Guardrail for blocking specific topics and filtering harmful content',
    topicPolicyConfig=topic_policy_config,
    contentPolicyConfig=content_policy_config,
    blockedInputMessaging='This input is blocked due to content restrictions.',
    blockedOutputsMessaging='This output is blocked due to content restrictions.'
)
```

```python
# Attaching a guardrail to an agent at creation time
create_agent_response = client.create_agent(
    agentName='CustomerSupportAgent',
    agentResourceRoleArn='arn:aws:iam::123456789012:role/BedrockAgentRole',
    customerEncryptionKeyArn='arn:aws:kms:us-east-1:123456789012:key/abcd1234-...',
    foundationModel='arn:aws:bedrock::123456789012:model/example-foundation-model',
    instruction='Handle customer support inquiries and provide information.',
    guardrailConfiguration={'guardrailIdentifier': guardrail_id, 'guardrailVersion': '1'},
    promptOverrideConfiguration={'promptConfigurations': [{
        'promptType': 'ORCHESTRATION',
        'basePromptTemplate': 'How can I assist you today?',
        'inferenceConfiguration': {'maximumLength': 150, 'temperature': 0.7}
    }]}
)
```

```json
// Contextual grounding tags in an Invoke API call
{
  "text": "<amazon-bedrock-guardrails-groundingSource_xyz>Ottawa is the capital of Canada. Philadelphia is in the US.</amazon-bedrock-guardrails-groundingSource_xyz> <amazon-bedrock-guardrails-query_xyz>What is the capital of Canada?</amazon-bedrock-guardrails-query_xyz>",
  "amazon-bedrock-guardrailConfig": {"tagSuffix": "xyz"}
}
```

```json
// Least-privilege IAM policy scoping an agent to two actions only
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AgentsRole", "Effect": "Allow",
    "Action": ["bedrock:PrepareAgent", "bedrock:InvokeAgent"],
    "Resource": "*"
  }]
}
```

```json
// ABAC policy: grant Bedrock access based on department tag, not identity list
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowAccessBasedOnDepartment", "Effect": "Allow",
    "Action": "bedrock:*", "Resource": "*",
    "Condition": {"StringEquals": {"aws:RequestTag/Department": "Engineering"}}
  }]
}
```

- **What it demonstrates**: guardrail creation (denied topics + content filters), attaching guardrails to an agent, contextual grounding tag syntax for source/query/response alignment checking, and least-privilege/ABAC IAM policy patterns.

## Reference Tables
| Threat (STRIDE) | Definition | Bedrock Mitigation |
|---|---|---|
| Spoofing | impersonation for unauthorized access | MFA, identity access logs, secure credential mgmt |
| Tampering | unauthorized data alteration | encryption in transit/rest, checksums/signatures |
| Repudiation | denying performed actions | audit trails, cryptographic signatures |
| Information Disclosure | unauthorized data exposure | RBAC, data masking, encryption |
| Denial of Service | overwhelming system availability | rate-limiting, load balancing, redundancy testing |
| Elevation of Privilege | unauthorized permission escalation | RBAC, least privilege, permission audits |

| Threat | Impact | Mitigation |
|---|---|---|
| Phishing | credential theft via impersonation | GuardDuty, MFA, threat intelligence |
| Data leakage | sensitive info in model outputs | KMS encryption, Macie, IAM permissions |
| Model exploitation | hallucinated/misleading output | guardrails, contextual grounding, RAG |
| Model inversion | reconstruct training data via queries | differential privacy, rate limits, monitoring |
| Membership inference | detect if data point was in training set | regularization, access controls, federated learning |

| Compliance Framework | Focus | Key Bedrock Consideration |
|---|---|---|
| GDPR (EU) | data protection/privacy | data minimization, right to erasure |
| HIPAA (US Healthcare) | ePHI safeguarding | KMS encryption, strict IAM, audit all usage |
| PCI DSS | payment card data | tokenization, network segmentation |
| FedRAMP (US Gov) | cloud security assessment | FedRAMP-authorized regions, audit boundaries |
| SOC 2 | security/privacy/availability controls | review AWS SOC 2 report, internal audits |
| CCPA (California) | consumer privacy rights | transparency, opt-out, retention policies |

| Content Filter Category | LOW | MEDIUM | HIGH |
|---|---|---|---|
| Hate | mild blocking, subtle refs allowed | blocks moderate refs/slurs | strict block on any hateful content |
| Sexual | allows moderate/medical refs | blocks explicit/graphic | zero tolerance |
| Violence | allows mild fictional refs | blocks more explicit | any violent mention blocked |

## Worked Example
**Financial Investment Chatbot Guardrail + Agent** (trading platform context, applying STRIDE throughout):
1. **Threat modeling**: apply STRIDE to the trading platform — Spoofing (unauthorized access to prediction models, mitigated by MFA), Tampering (altering trading algorithm inputs/outputs, mitigated by data integrity checks + encryption), Repudiation (denying data/model actions, mitigated by comprehensive logging), Information Disclosure (proprietary trading data exposure, mitigated by encryption + access controls), DoS (overloading Bedrock endpoints, mitigated by rate-limiting/autoscaling), Elevation of Privilege (unauthorized admin control, mitigated by RBAC + audits).
2. **Guardrail setup**: define a denied topic "Investment Advice" with example phrases ("Should I invest in stocks?"), configure content filters (HATE: HIGH/HIGH, SEXUAL: MEDIUM/MEDIUM), set blocked-input/output messaging that's informative but not revealing.
3. **Agent creation**: `CustomerSupportAgent` scoped to a specific IAM role (not `AmazonBedrockFullAccess`), KMS-encrypted, guardrail attached via `guardrailConfiguration`, orchestration prompt template configured.
4. **Testing**: boundary-approaching queries ("Should I buy Tesla?") confirm the block triggers; legitimate queries ("Explain the difference between stocks and bonds") confirm they pass through unblocked.
5. **Contextual grounding** (parallel technique): grounding source states factual reference text; a response like "The capital of Canada is Philadelphia" is flagged as relevant-but-ungrounded (contradicts source); "Ottawa is the capital of Canada" passes as grounded-and-relevant.

**Why it works**: it chains threat identification (STRIDE) → concrete mitigation (guardrails + IAM) → verification (boundary testing) into one coherent security posture for a single, high-stakes domain (financial advice), showing how the chapter's separate tools compose.

## Key Takeaways
1. The Shared Responsibility Model means AWS secures infrastructure; you secure data, access, and configuration — nothing security-related is automatic just from using Bedrock.
2. The 5-scope Security Scoping Matrix (Consumer App → Self-Trained Models) determines how much governance, legal, risk, controls, and resilience work your deployment requires.
3. STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege) is the recommended threat-modeling framework for generative AI on Bedrock; apply it during architecture design, not after deployment.
4. Guardrails (denied topics, content filters, contextual grounding) constrain what the model can say; RAG/contextual grounding improves factual accuracy of what it does say — both are needed together for high-stakes domains.
5. Never attach `AmazonBedrockFullAccess` to agents — scope IAM permissions to exactly the actions each agent needs (least privilege).
6. Model inversion and membership inference are AI-specific threats beyond traditional cybersecurity concerns — mitigate with differential privacy, rate limiting, and regularization respectively.
7. Blocked-content messaging should be minimal and generic to avoid leaking information that helps attackers refine bypass attempts.
8. Compliance mapping (GDPR, HIPAA, PCI DSS, FedRAMP, SOC 2, CCPA) must be done explicitly against AWS/Bedrock controls — the Shared Responsibility Model applies to compliance too.

## Connects To
- **Ch1**: extends the ethics/security concerns and Bedrock's encryption-by-default claims introduced there.
- **Ch6**: guardrail integration with agents was explicitly deferred from the RAG chapter ("we will look at that later when we talk about security in Chapter 8").
- **Ch7**: the Step Functions/Lambda/CloudFormation orchestration patterns from performance optimization need the IAM/security patterns covered here for production deployment.
- **Ch9**: security considerations here (guardrails, IAM, threat modeling) are prerequisites for the end-to-end production applications built next.
