````markdown
# AI Architect Interview Question

## Topic: Interoperability and API Contracts

---

### Question

> You are designing an ecosystem of AI services (retrieval, reasoning, evaluation, and synthesis) that will be owned by different teams. How do you design the APIs and contracts between services to ensure long-term interoperability, minimal coupling, and safe evolution?

---

### Answer

Designing service contracts for AI systems requires combining good API design principles with explicit data and semantic contracts. The goal is to enable independent teams to evolve implementations without breaking consumers.

---

#### Key Principles

**1. Schema-first contracts**

Define clear input/output schemas (JSON Schema, Protobuf, or Avro). Schemas should include type information, required fields, and semantic descriptions for each field (not just types).

**2. Semantic versioning + deprecation policy**

Treat API changes with semantic versioning. Establish explicit deprecation timelines and graceful fallback rules.

**3. Backward-compatible evolution**

Prefer additive changes (new optional fields) and use feature flags for non-breaking behavioral changes. Avoid renaming or removing fields without an agreed migration plan.

**4. Contract tests**

Use consumer-driven contract tests to ensure changes on provider side don't break consumers. Automate these tests in CI for each integration boundary.

**5. Typed data and canonical vocabularies**

Publish canonical vocabularies for common concepts (e.g., `Document`, `Claim`, `Citation`) with clear semantics. This reduces interpretation drift between teams.

**6. Sidecar metadata & capabilities discovery**

Expose service capabilities (model families supported, max tokens, latency SLOs, deterministic vs. stochastic modes) via a discovery endpoint so consumers can adapt dynamically.

**7. Error and uncertainty contracts**

Standardize how uncertainty, confidence, and failure modes are reported (confidence scores, provenance links, and error codes). Consumers should be able to programmatically interpret these.

**8. Security and rate limits**

Define authentication, authorization scopes, and rate-limiting expectations in the contract. Include required headers and audit-tracing fields.

---

#### Operational Patterns

1. Use an API gateway to manage versions and route requests to correct service versions.
2. Provide SDKs and codegen from schemas to reduce client-side parsing bugs.
3. Maintain a public changelog and a deprecation calendar for all contract changes.
4. Run contract compatibility checks in CI for every provider change and consumer integration.

---

#### What This Tests

- Ability to design robust integration points between teams
- Knowledge of schema/versioning strategies and contract testing
- Understanding of operational and security needs for AI services

````
