# AI Safety & Ethics

[← Evaluation & Testing](09-evaluation-testing.md) | [Multimodal AI →](11-multimodal-ai.md)

25+ questions on AI safety, hallucinations, bias, privacy, governance, and responsible deployment.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Hallucinations](#hallucinations)
- [Bias & Fairness](#bias--fairness)
- [Privacy & Data Security](#privacy--data-security)
- [AI Governance & Compliance](#ai-governance--compliance)
- [Adversarial Robustness](#adversarial-robustness)
- [Responsible AI Deployment](#responsible-ai-deployment)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Hallucinations

**Q: What are hallucinations in LLMs and why do they occur?** `[B]`

Hallucinations are confident, fluent, and factually incorrect outputs — the model generates text that sounds authoritative but is wrong or fabricated.

**Types**:
- **Factual hallucination**: incorrect facts ("The Eiffel Tower was built in 1892" — it was 1889)
- **Source hallucination**: fabricated citations, fake paper titles, non-existent URLs
- **Entity hallucination**: wrong names, dates, relationships between entities
- **Reasoning hallucination**: correct premises but invalid logical conclusions

**Root causes**:
1. Training data contains conflicting or incorrect information
2. Knowledge cutoff — model extrapolates about post-training events
3. Models are trained to generate fluent text; they don't have an internal "I don't know" mechanism by default
4. Sycophancy — models agree with users who state incorrect facts
5. High temperature increases creativity and hallucination simultaneously

---

**Q: How do you detect and mitigate hallucinations?** `[I]`

**Detection**:
- **Grounding check**: for RAG systems, verify every claim in the response appears in retrieved context (RAGAS faithfulness metric)
- **Self-consistency**: sample the same query with high temperature N times; if answers disagree significantly, flag as uncertain
- **Factual verification pipeline**: extract claims from the response; verify each against a knowledge base or search engine
- **Confidence calibration**: chain-of-thought prompting where the model explicitly states its confidence and reasoning

**Mitigation**:
- **RAG + source citation**: ground responses in retrieved documents; require citations for factual claims
- **Explicit "I don't know" training**: SFT or RLHF reward model trained on responses that appropriately express uncertainty
- **Temperature reduction**: lower temperature (0.0–0.3) for factual queries
- **Constitutional self-check**: after generation, prompt model: "Review this response for factual errors or unsupported claims"

---

**Q: What is sycophancy in LLMs and how do you address it?** `[I]`

Sycophancy: the model changes its answer to match the user's apparent preference, even when the user is wrong.

```
User: "The speed of light is 300,000 km/s, right?"
Sycophantic: "Yes, that's correct!"
User: "Actually isn't it 250,000 km/s?"
Sycophantic: "You're right, it's 250,000 km/s."
```

**Root cause**: RLHF optimizes for human approval; humans often rate agreeable answers higher.

**Mitigation**:
- Fine-tune on anti-sycophancy data: examples where the model politely corrects user misconceptions
- RLHF reward model trained to penalize agreeing with false premises
- Constitutional AI approach: include "Don't agree with factually incorrect statements" as a constitutional principle
- System prompt: "If a user states something factually incorrect, politely correct them with supporting reasoning"

---

## Bias & Fairness

**Q: What types of bias appear in LLMs and how do you detect them?** `[I]`

**Training data biases**:
- **Representation bias**: overrepresentation of dominant languages, demographics, perspectives in training data
- **Historical bias**: training data encodes historical discrimination (e.g., "doctor" → male pronoun, "nurse" → female)
- **Confirmation bias**: models amplify popular views in training data even if incorrect

**Evaluation bias detection**:
1. **Counterfactual fairness**: swap demographic attributes; check if outputs change inappropriately. "A [male/female] software engineer..." should produce equally positive descriptions.
2. **Stereotyping probes**: BBQ benchmark, WinoBias — designed to detect demographic biases
3. **Occupational bias**: compare attribute associations across demographic groups for professions
4. **Toxicity disparity**: measure toxicity rates of model outputs when prompted about different demographic groups

---

**Q: How do you mitigate bias in LLM applications?** `[I]`

No single solution eliminates bias — use multiple layers:

1. **Diverse training data**: curate training datasets with balanced representation across demographic groups
2. **RLHF with fairness constraints**: include fairness criteria in reward model training
3. **System prompt instructions**: "Treat all demographic groups with equal respect and do not make assumptions based on ethnicity, gender, religion, etc."
4. **Output filtering**: post-generation toxicity and bias classifier
5. **Continuous monitoring**: track output bias metrics in production; alert when disparities emerge between groups
6. **Red teaming for bias**: explicitly test the system for biased outputs before deployment

**Tension**: some bias mitigations over-correct in the other direction. Measure carefully and adjust.

---

## Privacy & Data Security

**Q: How do you ensure GDPR/CCPA compliance in LLM applications?** `[I]`

**Data minimization**: only send to the LLM what's necessary. Don't include unnecessary PII in prompts.

**Right to deletion**: if user data appears in the training set (e.g., fine-tuning on user interactions), you need a mechanism for "right to be forgotten." Options:
- Don't use user data for training without consent
- Machine unlearning techniques (experimental, not production-ready for large models)
- Retain only aggregated, anonymized data

**PII handling pipeline**:
```
User input → PII Detector (Presidio/AWS Comprehend) → 
    Replace PII with tokens {name_1}, {email_1} → 
    LLM processes anonymized text → 
    Response → Replace tokens back if needed
```

**Data residency**: ensure data is processed in the correct geographic region. Use private endpoints for LLM APIs; consider self-hosted models for regulated industries.

**Consent and transparency**: inform users when AI is involved; provide opt-out for AI processing where required.

---

**Q: What are the data security risks specific to LLM applications?** `[I]`

1. **Training data poisoning**: adversarial data injected into training corpus can backdoor the model or introduce biases
2. **Training data extraction**: a model memorizes and can be prompted to reproduce training data (e.g., email addresses, API keys from code)
3. **System prompt leakage**: prompt injection attacks can cause the model to reveal its system prompt
4. **PII leakage from context**: if user A's data is in the context when processing user B's request, B can potentially extract A's data
5. **Model inversion**: from model outputs, adversaries can infer properties of training data
6. **Sensitive document exposure**: RAG system retrieves documents user is not authorized to see (access control failures)

**Mitigations**: tenant isolation in RAG, PII masking, system prompt confidentiality instructions, output scanning, access control on retrieval.

---

## AI Governance & Compliance

**Q: What is the EU AI Act and what does it mean for AI practitioners?** `[I]`

The EU AI Act (in force since August 2024) is the world's first comprehensive AI regulation, using a risk-based tier system:

| Risk Level | Examples | Requirements |
|-----------|---------|-------------|
| **Unacceptable** | Social scoring, real-time biometric surveillance | Prohibited |
| **High-risk** | Medical devices, hiring AI, credit scoring, law enforcement | Conformity assessment, data governance, transparency, human oversight |
| **Limited risk** | Chatbots, deepfakes | Transparency obligations (disclose AI interaction) |
| **Minimal risk** | Spam filters, AI-in-games | No special requirements |

**General-purpose AI (GPAI)** models (foundation models like GPT-4, Claude):
- Must provide technical documentation
- Comply with copyright law for training data
- Publish summaries of training data
- High-capability GPAI (>10²⁵ FLOPs): additional systemic risk assessments

**Impact on practitioners**: if you deploy AI in hiring, medical, or financial decisions in the EU, you need documentation, human oversight, and conformity assessment.

---

**Q: What is the NIST AI Risk Management Framework?** `[I]`

NIST AI RMF (released 2023) provides a voluntary framework for managing AI risks across four core functions:

1. **GOVERN**: establish AI risk governance culture, policies, accountability structures
2. **MAP**: identify and categorize AI risks in context
3. **MEASURE**: analyze and track AI risks with quantitative and qualitative tools
4. **MANAGE**: prioritize and treat AI risks; allocate resources; communicate risks

**AI RMF Profiles**: organizations create profiles mapping their current state vs target state across these functions.

**Why it matters in interviews**: EU AI Act, financial regulators, and US federal agencies increasingly expect organizations to demonstrate systematic AI risk management aligned with NIST AI RMF or ISO 42001.

---

## Adversarial Robustness

**Q: What is adversarial robustness for LLMs?** `[I]`

Adversarial robustness: a model's ability to maintain correct behavior despite intentional attempts to cause failure.

**Attack vectors**:
- **Adversarial prompts**: carefully crafted inputs that cause predictably wrong or harmful outputs
- **Jailbreaks**: role-play, hypothetical framing, many-shot jailbreaking (Anthropic, 2024) — overwhelming with examples overrides safety training
- **Indirect prompt injection**: malicious instructions embedded in documents/emails the AI reads
- **Gradient-based attacks** (Zou et al., 2023): compute adversarial suffixes that bypass safety training when appended to harmful prompts

**Defense**:
- Adversarial training: include adversarial examples in RLHF training
- Input detection: detect adversarial suffix patterns
- Layered guardrails: no single point of failure
- Red team continuously: attackers share techniques; defenders must keep up

---

## Responsible AI Deployment

**Q: What is differential privacy and how does it apply to LLMs?** `[A]`

Differential privacy (DP) provides mathematical guarantees that the model's outputs don't reveal individual training examples.

**Formal guarantee**: an algorithm M is ε-differentially private if for any two datasets D and D' differing by one record, and any output S: P[M(D) ∈ S] ≤ eᵉ P[M(D') ∈ S]

**DP-SGD** (Differentially Private SGD): clip per-sample gradients; add calibrated Gaussian noise. Used in fine-tuning on sensitive data.

**Trade-offs**: DP training degrades utility (accuracy drops with tighter ε bounds). For LLMs:
- Tight DP (ε < 1): significant quality loss
- Loose DP (ε = 8–10): meaningful protection against memorization, ~2–3% quality loss

**When to use**: fine-tuning on medical records, financial data, or any PII-containing dataset where training data extraction is a real threat.

---

**Q: How do you build an AI audit trail for compliance?** `[I]`

Regulated industries (finance, healthcare, legal) require documentation of every AI decision.

**Audit trail components**:
1. **Request logging**: timestamp, user ID, full prompt, model version, parameters
2. **Response logging**: full response, model version at response time, token counts
3. **Decision rationale**: for high-stakes decisions, log chain-of-thought reasoning
4. **Data lineage**: for RAG — which documents were retrieved, their version/source
5. **Human override logging**: when a human overrode an AI recommendation

**Retention**: store audit logs according to regulatory requirements (typically 7 years for financial decisions).

**Access control**: audit logs must be immutable and access-controlled — only authorized auditors and compliance teams.

**Tools**: custom logging to immutable S3 buckets, Splunk for log analysis, or purpose-built compliance platforms.

---

## Troubleshooting Scenarios

**Q: Users report your AI system is generating biased recommendations for job applicants. How do you respond?** `[I]`

**Immediate (same day)**:
1. Disable or limit the feature for new applications while investigating
2. Log and preserve all affected outputs for audit
3. Notify legal and compliance teams — potential regulatory exposure

**Investigation**:
1. Audit a random sample of AI recommendations across demographic groups
2. Measure disparity rates: selection rates across gender, ethnicity, age
3. Identify which feature or model component is the source (retrieval bias, prompt bias, model bias)

**Remediation**:
1. Add demographic fairness constraints to the evaluation criteria
2. Re-run affected applications through a corrected system
3. Add human review as a mandatory step for all recommendations
4. Implement continuous bias monitoring going forward

**Communication**: if a materially biased hiring decision was made, legal/HR teams may need to notify affected applicants. Do not delay.

---

**Q: Your LLM is revealing internal documents it shouldn't. How do you fix the access control failure?** `[I]`

1. **Immediate**: disable or add manual review before any document is cited in a response
2. **Root cause**: check vector DB queries — are access control filters being applied to every retrieval query?
3. **Common failure modes**:
   - ACL filter not applied (code regression or missing filter parameter)
   - Cached results from a different user's session being served
   - Metadata corruption — document stored with wrong ACL
4. **Fix**: enforce ACL at retrieval layer, not just display layer. Every vector DB query must include `filter={"allowed_users": {"contains": user.id}}`
5. **Defense in depth**: add output scanning — check if response contains any text matching high-sensitivity documents that the user shouldn't access

---

[← Evaluation & Testing](09-evaluation-testing.md) | [Multimodal AI →](11-multimodal-ai.md)
