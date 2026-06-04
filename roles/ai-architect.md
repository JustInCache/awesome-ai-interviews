# AI Architect Interview Questions

[← ML Engineer](ml-engineer.md) | [AI Researcher →](ai-researcher.md)

Scenario-based interview questions for AI Architect roles — system-level thinking, technical strategy, and organizational leadership.

**Related**: [AI System Design](../topics/07-ai-system-design.md) | [AI Safety & Ethics](../topics/10-ai-safety-ethics.md) | [LLMOps](../topics/08-llmops-production.md)

---

## What AI Architects Do

AI Architects design the overall AI strategy and technical architecture for an organization. They make high-stakes technology decisions (build vs. buy, monolith vs. microservices, cloud vs. on-prem), define governance and safety standards, ensure compliance, and align AI investments with business outcomes.

---

## Interview Questions

### Q1: Multi-Agent Consistency

> **You're architecting a multi-agent system where three specialized agents must collaborate to produce a consistent output. How do you prevent contradictions between agents?**

Multi-agent contradiction is an emergent problem — each agent is internally consistent, but their outputs conflict because they operate with different context or make different assumptions.

**Architectural solutions**:

**1. Shared context store with versioned state**: every agent reads from and writes to a single shared state object. Before an agent commits a conclusion, it reads existing state for conflicts.

```python
# Shared state with lock
with state.lock():
    existing = state.get("conclusion_X")
    if existing and conflicts(existing, new_conclusion):
        arbitrate(existing, new_conclusion)
    else:
        state.set("conclusion_X", new_conclusion)
```

**2. Explicit role boundaries**: each agent has a defined input schema, output schema, and domain scope. Agent A handles financial data; Agent B handles risk assessment. Their outputs don't overlap by design.

**3. Orchestrator pattern**: a coordinator agent sequences agent calls and manages dependencies. If Agent B's output requires Agent A's output, enforce that ordering. No parallel independent derivation of the same conclusion.

**4. Verifier agent**: a dedicated review agent receives all outputs, identifies contradictions, and arbitrates based on explicit priority rules (most recent, highest confidence, or domain-specific precedence).

**5. Output contracts (Pydantic schemas)**: structured outputs make contradictions programmatically detectable and resolvable rather than requiring LLM interpretation.

---

### Q2: Scaling Strategy

> **Your AI system serves 1,000 users today. Walk me through your strategy to scale it to 1 million users.**

**Current state (1K users)**: single region, direct API calls, minimal caching, monolithic application.

**Scale stages**:

**Phase 1 → 10K users**:
- Add Redis for semantic caching (30–50% LLM call reduction)
- Add rate limiting per user/tenant
- Separate LLM inference service from application service
- Add basic observability (latency, error rate, cost per request)

**Phase 2 → 100K users**:
- Multi-instance deployment with load balancer
- Async processing queue (Celery/Redis) for non-real-time operations
- Tiered model routing: simple queries → cheap/fast model, complex → premium model
- Vector DB cluster (dedicated, not embedded)
- CDN for static assets

**Phase 3 → 1M users**:
- Multi-region deployment for geographic latency
- Tenant isolation: enterprise customers get dedicated namespaces; free tier on shared infrastructure
- Self-hosted fine-tuned models for high-volume use cases (unit economics)
- Global load balancing with health-based routing
- Chaos engineering and game days before each major scale milestone

**Cost modeling at each phase**: LLM cost typically dominates (60–80% of infrastructure cost). Every architectural decision should include a cost-per-query analysis.

---

### Q3: Build vs. Buy Decisions

> **Your company wants to add an AI-powered customer support chatbot. Do you build on GPT-4 or use a vendor solution like Intercom/Zendesk AI? How do you decide?**

**Framework for build vs. buy**:

| Dimension | Build (API/Open Source) | Buy (Vendor Solution) |
|-----------|------------------------|----------------------|
| **Time to value** | 4–12 weeks | 1–2 weeks |
| **Customization** | Full control | Limited to vendor features |
| **Data ownership** | Full ownership | Data leaves your infrastructure |
| **Cost at scale** | Lower per-query, higher upfront | Predictable but potentially high at scale |
| **Compliance** | You control data residency | Vendor certifications (SOC 2, HIPAA) |
| **Maintenance** | Your team owns it | Vendor handles updates |

**When to buy**: time-to-market is critical; team lacks ML expertise; regulatory requirements met by vendor certifications; volume is low.

**When to build**: high data sensitivity/compliance requirements; need deep customization; volume justifies investment; existing ML team; competitive differentiation from the AI capability itself.

**Hybrid approach (often optimal)**: use vendor for initial launch; instrument heavily to understand usage patterns; migrate to custom build for highest-ROI features once product-market fit is established.

**Questions to ask before deciding**:
- What's the expected query volume in year 1? Year 3?
- Do we have proprietary data that can differentiate our quality?
- What's the regulatory requirement for data handling?
- Does this capability become a competitive moat?

---

### Q4: Reliability Design

> **Design an AI system that maintains 99.9% availability even when the primary LLM provider has an outage.**

99.9% = 8.7 hours downtime/year. Achieving this with a single LLM provider is impossible — OpenAI, Anthropic, and others have had outages exceeding this budget.

**Multi-provider failover architecture**:

```
Request → LLM Router
    ├─ Primary: OpenAI GPT-4o
    ├─ Secondary: Anthropic Claude 3.5
    └─ Tertiary: Azure OpenAI (different region)

Router logic:
- Health check every 30s per provider
- Circuit breaker: open after 5 failures in 60s
- Automatic failover in <100ms
- Log which provider served each request
```

**Provider compatibility layer**: abstract LLM calls behind a provider-agnostic interface. All providers implement the same interface. Switching providers = changing config, not code.

**Response consistency**: different providers may produce slightly different outputs. Define an output schema (Pydantic model) and validate all responses against it. Provider differences are masked at the interface boundary.

**Degraded mode**: if all providers are down, serve:
- Cached responses for common queries
- Scripted fallback responses for known intents
- Transparent "temporarily unavailable" message rather than silent failure

**Chaos testing**: periodically inject provider failures in staging to verify failover works; include in load tests.

---

### Q5: AI Ethics and Governance

> **You're deploying an AI hiring tool. What governance and ethics framework do you put in place?**

Hiring AI operates in a high-risk regulated space (EEOC, EU AI Act, NYC Local Law 144). A governance failure here creates legal liability and reputational harm.

**Governance framework**:

**1. Pre-deployment**:
- Disparate impact analysis: measure selection rates across protected groups (race, gender, age, disability)
- Third-party audit: independent firm audits for bias before launch
- Legal review: confirm compliance with applicable employment law
- Explainability requirement: every AI-assisted decision must have a human-reviewable rationale

**2. Access controls**:
- AI provides ranked candidates with rationale — humans make final decisions
- AI cannot be sole decision-maker for any hire
- All AI recommendations logged with candidate ID, rationale, score

**3. Monitoring in production**:
- Track pass rates by demographic group weekly
- Alert if any group's pass rate falls >4/5ths of the highest group's rate (EEOC 4/5 rule)
- Monthly bias reports to legal and HR leadership

**4. Candidate transparency**:
- Inform candidates AI was used in evaluation
- Provide opt-out path where legally required
- Enable candidates to request human review of AI decision

**5. Audit trail**:
- Immutable log of all AI recommendations with model version, features used, timestamp
- Retention: 7 years (EEOC requirement)

---

### Q6: Future-Proofing AI Architecture

> **How do you design an AI architecture that won't be obsolete in 2 years given the rapid pace of AI advancement?**

AI advances fast; architectural decisions made today will be stressed by models with 10× capability in 2 years. Key principles for future-proofing:

**1. Provider abstraction**: never couple tightly to a specific model or provider. Use an adapter pattern where model upgrades are a config change. This alone enables keeping pace with frontier models without code rewrites.

**2. Evaluation-first**: invest heavily in evaluation infrastructure before model infrastructure. Evals enable you to safely upgrade models; without evals, you can't know if a model change helped or hurt.

**3. Data moat**: collect and curate proprietary data from day one. Models improve, but your unique data remains a differentiator. The architecture should make it easy to use your data to fine-tune future models.

**4. Modular pipelines**: loose coupling between components (retrieval, generation, evaluation, memory). When a better embedding model ships, swap just the embedding module. When a better reranker ships, swap just the reranker.

**5. Technology bets vs. principles**: bet on principles (RAG as a pattern, agents as an architecture), not specific frameworks (LangChain versions come and go). Frameworks change; patterns persist.

**6. Continuous benchmarking**: run your evaluation suite on new model versions as they're released. When a new model improves your key metrics by >5% at acceptable cost, upgrade. The evaluation infrastructure is what enables this.

---

### Q7: Interoperability and API Design

> **How do you design AI system APIs to allow interoperability with future AI models and tools?**

**Principle**: design APIs to the OpenAI standard where possible. The ecosystem has converged on OpenAI-compatible APIs for LLM serving.

**API design principles**:

1. **OpenAI-compatible chat completions format**: even if using Anthropic or a self-hosted model, expose an OpenAI-compatible API. Clients can switch providers by changing base_url.

2. **Streaming support**: all LLM endpoints should support streaming (Server-Sent Events). Non-streaming is a subset of streaming; design for streaming first.

3. **Tool/function calling standardization**: use the OpenAI function calling schema as the standard for tool definitions. Most major models now support this format.

4. **Versioned API**: `v1/`, `v2/` prefixes. Breaking changes require a new version. Maintain the previous version for 6 months.

5. **Model routing as configuration**: the model name in the API request should be a logical name (e.g., `"smart-model"`, `"fast-model"`) that maps to a physical model in a routing config. Swapping the underlying model doesn't require client code changes.

---

### Q8: Compliance and Auditability

> **How do you ensure an AI system deployed in financial services meets regulatory compliance requirements?**

Financial AI is subject to: model risk management guidelines (SR 11-7), GDPR/CCPA, SEC/FINRA requirements for explainability, and emerging AI-specific regulations.

**Compliance architecture**:

**Model documentation (SR 11-7)**:
- Model inventory: every AI model registered with use case, developer, owner, validation status
- Model cards: training data, intended use, limitations, performance by demographic group
- Validation: independent model validation team reviews each model before production

**Explainability**:
- Regulatory decisions (credit, trading) must be explainable. LLMs' black-box nature is a problem.
- Use structured decision frameworks: LLM → structured JSON output → rule-based explanation template
- SHAP/LIME explanations for traditional ML components in hybrid systems

**Audit trail (immutable)**:
- Every AI decision logged: input, output, model version, timestamp, user ID, confidence
- Stored in append-only, access-controlled storage (AWS S3 with Object Lock)
- Retention: 7+ years as required

**Change management**:
- Model updates go through formal change management: UAT, validation, sign-off
- Prompt changes treated as model changes — require same review
- Rollback capability for any production model version

---

## Preparation Checklist

For AI Architect interviews, be ready to discuss:

- [ ] Multi-region, multi-provider resilient architectures
- [ ] Build vs. buy trade-off analysis
- [ ] AI governance frameworks (NIST AI RMF, EU AI Act)
- [ ] Cost modeling and unit economics for AI systems
- [ ] Compliance requirements in regulated industries
- [ ] Multi-agent system design patterns
- [ ] Organizational AI strategy (center of excellence vs. embedded)
- [ ] Technical roadmap planning under uncertainty

---

[← ML Engineer](ml-engineer.md) | [AI Researcher →](ai-researcher.md)
