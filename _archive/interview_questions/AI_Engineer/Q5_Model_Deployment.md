# AI Engineer Interview Question

## Topic: Model Deployment & Monitoring

---

### Question

> You've trained a great model offline, but it performs poorly in production. How do you approach deploying and monitoring AI models to catch issues early?

---

### Answer

Model deployment isn't just about getting the model running—it's about maintaining performance over time. The key is treating models like any other software component: version control, testing, monitoring, and rollback capability.

---

#### Deployment Strategy

**1. Containerization & Versioning**

- Package models as containers (Docker) with all dependencies
- Version models like code (semantic versioning: 1.2.3)
- Store model artifacts in registry (like Docker Hub or MLflow Model Registry)
- Include metadata: training data version, hyperparameters, performance metrics

**2. Gradual Rollout**

Never go from 0% to 100% traffic:

- **Canary deployment**: 1% → 5% → 25% → 100% traffic
- Compare performance metrics between canary and baseline
- Automatic rollback if metrics degrade beyond threshold

**3. A/B Testing Infrastructure**

- Route traffic to different model versions
- Measure business metrics, not just technical ones
- Statistical significance testing for small changes

---

#### Monitoring Framework

Monitor three levels:

**Level 1: System Health**

- Latency (p50, p95, p99)
- Throughput (requests per second)
- Error rates (4xx, 5xx)
- Resource utilization (CPU, memory, GPU)

**Level 2: Model Performance**

- Prediction accuracy on recent data
- Confidence score distributions
- Feature drift detection
- Output distribution shifts

**Level 3: Business Impact**

- User engagement metrics
- Conversion rates
- Customer satisfaction scores
- Revenue or cost metrics

---

#### Early Warning Systems

**Data Drift Detection**

Models fail when input distribution changes:

- Statistical tests on feature distributions
- Population stability index (PSI)
- Alert when PSI > 0.1 (significant drift)

**Concept Drift Detection**

Model predictions become less accurate over time:

- Monitor prediction accuracy on recent vs. historical data
- Track calibration (confidence vs. actual accuracy)
- Set up alerts for accuracy drops >5%

**Performance Degradation**

- Automated retraining triggers
- Model freshness metrics (how old is the training data?)
- A/B testing for model updates

---

#### Operational Best Practices

**1. Feature Stores**

Centralize feature computation to ensure consistency between training and serving:

- Same code paths for training and inference
- Versioned features
- Data quality monitoring

**2. Model Validation**

Before deployment:

- Unit tests for model loading and prediction
- Integration tests with realistic data
- Performance benchmarks
- Shadow mode testing (run new model alongside old, compare outputs)

**3. Incident Response**

- Runbooks for common issues (high latency, accuracy drops)
- Rollback procedures (can revert to previous model version in minutes)
- Escalation paths for different severity levels

**4. Continuous Improvement**

- Regular model updates (weekly/monthly retraining)
- Performance dashboards for stakeholders
- Feedback loops from production data back to training

---

#### Example Monitoring Dashboard

```
┌─────────────────────────────────────────────────────────────┐
│ Model Performance Dashboard - v2.1.3 (Deployed 2024-01-15) │
├─────────────────────────────────────────────────────────────┤
│ Latency: 95% < 200ms ✓ | Throughput: 500 RPS ✓             │
│ Accuracy: 94.2% (target: >93%) ✓                           │
│ Data Drift: PSI = 0.08 (threshold: 0.1) ✓                  │
│ Feature 'user_age': Distribution stable ✓                  │
│ Business Metric: Conversion +2.3% vs baseline ✓            │
├─────────────────────────────────────────────────────────────┤
│ Alerts: None active                                         │
│ Last Retrained: 2024-01-10 (5 days ago)                     │
│ Next Scheduled: 2024-01-17                                  │
└─────────────────────────────────────────────────────────────┘
```

---

#### Common Failure Modes & Fixes

| Problem | Symptom | Fix |
|---------|---------|-----|
| Model staleness | Gradual accuracy decline | Automated retraining pipeline |
| Data drift | Feature distributions shift | Feature monitoring + alerts |
| Serving skew | Training/serving feature mismatch | Feature store standardization |
| Cold start issues | High latency on model load | Model pre-warming, caching |
| Resource contention | OOM errors, high latency | Horizontal scaling, resource limits |

---

### What This Question Tests

- Production mindset for ML systems
- Understanding of ML lifecycle beyond training
- Monitoring and observability practices
- Risk management and rollback strategies
