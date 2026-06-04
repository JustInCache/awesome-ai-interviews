# ML Engineer Interview Question

## Topic: Training Pipeline Design

---

### Question

> You're tasked with setting up training infrastructure for a team that will be running hundreds of experiments per month. What does your ideal training pipeline look like, and what are the critical decisions?

---

### Answer

A good training pipeline isn't just about running jobs—it's about **enabling fast iteration** while maintaining **reproducibility** and **cost efficiency**.

Here's how I'd think through the design:

---

#### Core Components

**1. Experiment Tracking**

Every run needs to capture:
- Hyperparameters (learning rate, batch size, architecture choices)
- Data version (what exact dataset was used)
- Code version (git commit hash)
- Metrics over time (loss curves, eval metrics)
- Artifacts (checkpoints, final model)

Without this, you can't reproduce results or understand what worked.

**2. Configuration Management**

I'd use a config-driven approach where experiments are defined declaratively:

```yaml
model:
  architecture: transformer
  hidden_size: 768
  num_layers: 12
  
training:
  learning_rate: 1e-4
  batch_size: 32
  max_steps: 100000
  
data:
  dataset: v2.3
  preprocessing: standard
```

This makes experiments reproducible and diffable.

**3. Compute Orchestration**

For hundreds of experiments:
- **Job queue** to manage submissions and priorities
- **Auto-scaling** to spin up/down based on demand
- **Preemptible instances** for cost savings (with checkpointing for recovery)
- **Resource limits** per user/project to prevent runaway costs

**4. Data Pipeline**

- **Versioned datasets** so you know exactly what data trained each model
- **Efficient data loading** (don't let I/O bottleneck GPU utilization)
- **Data validation** to catch corrupt or malformed data before training starts

---

#### Critical Decisions

**Decision 1: Where do checkpoints live?**

Checkpoints are large and frequently accessed. I'd use:
- Fast storage (local NVMe) during training
- Durable storage (object storage) for completed runs
- Automatic cleanup policies to manage costs

**Decision 2: How do you handle failed runs?**

Training jobs fail. Good infrastructure:
- Checkpoints frequently (every N steps)
- Auto-resumes from last checkpoint on preemption
- Alerts on unexpected failures (OOM, NaN loss)
- Makes it easy to inspect what went wrong

**Decision 3: How do you compare experiments?**

With hundreds of experiments, you need:
- Dashboards to visualize metrics across runs
- Filtering by hyperparameters, data version, etc.
- Statistical significance testing for close results

---

#### What I'd Build vs. Buy

| Component | Build | Buy/Use |
|-----------|-------|---------|
| Experiment tracking | | Use existing tools (MLflow, W&B, etc.) |
| Config management | Custom for team needs | |
| Job orchestration | | Kubernetes + job scheduler |
| Data versioning | | Existing tools or object storage versioning |

Building experiment tracking from scratch is almost never worth it—the tooling is mature and battle-tested.

---

### Common Failure Modes I'd Design Against

1. **"I can't reproduce that good result from last month"** → Strict versioning of code, data, and configs

2. **"Training is slow and GPUs sit idle"** → Profile data loading, optimize preprocessing

3. **"We're spending too much on compute"** → Preemptible instances, automatic job termination, resource quotas

4. **"Nobody can find anything"** → Good naming conventions, tagging, searchable metadata

---

### What This Question Tests

- Systems thinking about ML infrastructure
- Understanding of the full training lifecycle
- Cost awareness and practical trade-offs
- Experience with real-world failure modes
