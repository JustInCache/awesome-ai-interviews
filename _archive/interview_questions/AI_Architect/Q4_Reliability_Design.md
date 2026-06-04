# AI Architect Interview Question

## Topic: Reliability & Failure Handling

---

### Question

> You're designing an AI system that will be used for high-stakes decisions (financial, medical, legal). How do you architect for reliability, and what failure modes do you design against?

---

### Answer

High-stakes AI systems require a fundamentally different architecture mindset. The goal isn't just "works most of the time"—it's "fails safely always." Every component needs to answer: "What happens when this breaks?"

---

#### The Reliability Hierarchy

In high-stakes systems, reliability has layers:

**Level 1: Availability** - System responds to requests

**Level 2: Correctness** - Responses are accurate

**Level 3: Confidence Calibration** - System knows when it's uncertain

**Level 4: Safe Failure** - When wrong, fails in least harmful way

Most AI systems optimize for Levels 1-2. High-stakes systems must nail Levels 3-4.

---

#### AI-Specific Failure Modes

Traditional software has predictable failure modes. AI systems have additional, often subtle, failure patterns:

**1. Silent Confidence Failures**

The model is wrong but confident. Traditional error handling doesn't catch this—the system thinks it succeeded.

Mitigations:
- Uncertainty quantification (confidence scores that actually mean something)
- Out-of-distribution detection
- Multiple model ensemble with disagreement detection
- Human review triggers for edge cases

**2. Distribution Shift**

The world changes, model assumptions break. Performance degrades gradually without explicit errors.

Mitigations:
- Continuous monitoring of prediction distributions
- Alerting on statistical drift
- Regular retraining pipelines
- Canary deployments with holdback groups

**3. Adversarial Manipulation**

Bad actors craft inputs to manipulate outputs (jailbreaks, prompt injection, adversarial examples).

Mitigations:
- Input validation and anomaly detection
- Rate limiting and abuse detection
- Output filtering for sensitive domains
- Audit trails for all decisions

**4. Cascading Hallucinations**

One wrong output feeds into another component, compounding errors.

Mitigations:
- Validation gates between components
- Ground truth anchoring where possible
- Circuit breakers that halt cascades
- Independent verification paths

---

#### Architecture Patterns for High-Stakes AI

**Pattern 1: Defense in Depth**

Multiple independent checks, any of which can reject or flag:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Input     │     │    Main     │     │   Output    │
│ Validation  │ ──► │    Model    │ ──► │ Validation  │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       └───────────────────┴───────────────────┘
                           │
                    ┌──────▼──────┐
                    │ Independent │
                    │  Verifier   │
                    └─────────────┘
```

No single component failure leads to bad output.

**Pattern 2: Confidence-Gated Output**

System behavior changes based on confidence:

| Confidence Level | Action |
|-----------------|--------|
| High (>95%) | Return result directly |
| Medium (70-95%) | Return with caveats, suggest verification |
| Low (<70%) | Escalate to human, or decline to answer |

This requires well-calibrated confidence—which is hard but essential.

**Pattern 3: Human-in-the-Loop Checkpoints**

For truly high-stakes decisions:

```
AI Analysis → Human Review → Action
              ↑            ↓
              └── Feedback Loop
```

The AI assists and accelerates, but humans make final calls on consequential decisions.

**Pattern 4: Audit Trail Everything**

Every decision must be explainable after the fact:
- Full input (what did the system see?)
- Model version and configuration
- All intermediate reasoning
- Confidence scores
- Final output
- Timestamp and request metadata

This enables post-hoc analysis when things go wrong.

---

#### Operational Reliability

Beyond architecture, operational practices matter:

**Deployment Safety**:
- Canary deployments (1% → 10% → 100%)
- Automatic rollback on metric degradation
- Feature flags to disable AI components
- Fallback to rules-based systems

**Monitoring**:
- Not just uptime—prediction quality metrics
- Drift detection on inputs and outputs
- Latency budgets with alerting
- Error rate segmented by input characteristics

**Incident Response**:
- Runbooks for AI-specific failures
- Kill switches for AI components
- Communication templates for stakeholders
- Post-mortems that include model behavior analysis

---

#### Regulatory & Compliance Considerations

High-stakes domains often have requirements:

**Explainability**: Can you explain why a decision was made?
- Use inherently interpretable models where possible
- Maintain decision logs with reasoning
- Generate human-readable explanations

**Auditability**: Can regulators review the system?
- Version control for models and training data
- Documented validation procedures
- Clear lineage from data to decision

**Bias & Fairness**: Does the system treat groups equitably?
- Regular fairness audits
- Monitoring for disparate impact
- Mitigation strategies documented

**Data Privacy**: Is sensitive data protected?
- Minimize data retention
- Anonymization where possible
- Clear consent and data handling policies

---

#### Example: Medical Diagnosis Support System

**Safety Architecture**:

1. **Input validation**: Verify all required fields, flag unusual values
2. **Primary model**: Generate differential diagnosis with confidence
3. **Second opinion model**: Independent model for comparison
4. **Disagreement detection**: Alert if models disagree significantly
5. **Known limitation filter**: Check against documented failure cases
6. **Confidence gate**: High confidence → suggest diagnosis; Low confidence → "recommend specialist review"
7. **Audit logging**: Full record for each case
8. **Human override**: Physician makes final diagnosis

**Failure handling**:
- Any component fails → Degrade to "unable to assist, please use standard protocols"
- Never present uncertain output as certain
- Always frame as "decision support," not "diagnosis"

---

### What This Question Tests

- Understanding of AI-specific failure modes
- Safety-first architecture mindset
- Knowledge of operational reliability practices
- Awareness of regulatory and ethical considerations
- Ability to design defense-in-depth systems
