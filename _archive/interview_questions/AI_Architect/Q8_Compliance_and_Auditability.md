````markdown
# AI Architect Interview Question

## Topic: Compliance, Auditability, and Traceability

---

### Question

> For regulated domains (finance, healthcare, legal), how do you design an AI architecture that meets compliance and audit requirements while preserving model flexibility and innovation? Walk through data handling, model decisions, and logging strategies.

---

### Answer

Balancing compliance and innovation requires building auditability and traceability into the architecture from the start. You need controls around data, decision provenance, and human oversight that don't block iteration.

---

#### Data Handling

1. Data lineage and consent: record source, consent status, retention policy, and transformation steps for every dataset used in training and inference.
2. Minimization and purpose limitation: store only required fields for compliance purposes and redact or tokenize sensitive attributes when possible.

#### Model Decisions & Explainability

1. Produce structured explanations for key decisions (features used, confidence, evidence links) for each high-risk output.
2. Use a hybrid approach: combine a deterministic rules layer for critical checks with learned models for flexible behavior.

#### Logging & Audit Trails

1. Immutable audit logs: store inputs, model version, random seeds (where relevant), post-processing steps, and output with timestamps in an append-only store.
2. Provenance pointers: logs should include references to external evidence (document IDs, retrieval timestamps) not full documents to avoid duplication while enabling traceability.

#### Governance & Controls

1. Model governance board and approval workflows for model changes that affect regulated behavior.
2. Automated policy checks in CI/CD for model changes (data drift, fairness metrics, PII leakage detectors).

---

#### Runtime Protections

1. Human-in-the-loop gating for high-risk decisions with clear escalation paths.
2. Policy enforcement layer to block or flag outputs that violate compliance rules using a combination of classifiers and deterministic checks.

---

#### What This Tests

- Practical architecture for compliance and auditability
- Understanding of data governance, explainability, and runtime controls
- Ability to balance regulatory needs with product velocity

````
