````markdown
# AI Engineer Interview Question

## Topic: Inference Observability, Metrics, and Fallbacks

---

### Question

> When deploying inference services for language models at scale, what observability signals should you collect, and how do you design reliable fallback strategies for degraded models or infrastructure issues?

---

### Answer

Observability is critical for detecting issues early and triggering safe fallbacks. Collect signals spanning performance, quality, and user-impact metrics and design multi-layered fallbacks.

---

#### Signals to Collect

**Performance:** latency p95/p99, CPU/GPU utilization, queue lengths, error rates

**Quality:** perplexity (where applicable), model confidence distribution, freshness of retrieved evidence, hallucination detectors

**User Impact:** rate of human escalations, task success rate, time-to-first-byte for user requests

**Data & Drift:** input distribution skews, feature drift, input length histograms

**Safety:** rate of blocked outputs, policy violations, false positives in filters

---

#### Fallback Strategies

1. Graceful degradation of model size: route to smaller, cheaper models with calibrated response formats when capacity is constrained.
2. Cached responses: serve previously computed results for repeat queries or high-confidence templates.
3. Read-only fallback: when synthesis models are down, provide retrieved evidence and suggest the user verify rather than produce novel responses.
4. Circuit breakers and throttling: prevent overload cascading across services.

---

#### Automation & Alerts

1. Define SLOs and automated remediation playbooks (restart, scale-out, switch-model).
2. Use anomaly detection on combined signals to trigger human review for silent quality degradation.

---

#### What This Tests

- Ability to instrument ML services effectively
- Designing pragmatic fallbacks that preserve user trust
- Understanding of operational SRE-style practices adapted for AI workloads

````
