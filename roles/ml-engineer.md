# ML Engineer Interview Questions

[← AI Engineer](ai-engineer.md) | [AI Architect →](ai-architect.md)

Scenario-based interview questions for Machine Learning Engineer roles.

**Related**: [Fine-Tuning](../topics/05-fine-tuning.md) | [AI Infrastructure](../topics/12-ai-infrastructure.md) | [LLMOps](../topics/08-llmops-production.md)

---

## What ML Engineers Do

ML Engineers focus on building, training, and deploying models. They bridge the gap between research (AI Researchers) and production (AI Engineers). Core skills: training pipelines, distributed training, model optimization, feature engineering, experiment tracking, reproducible infrastructure.

---

## Interview Questions

### Q1: Training Pipeline Design

> **Design a production ML training pipeline for a classification model that needs to be retrained weekly on fresh data.**

**Pipeline stages**:

```
Data Ingestion → Validation → Feature Engineering → Training → Evaluation → Registry → Deploy
```

**Data validation** (critical, often skipped):
- Schema validation: correct columns, types, expected ranges
- Distribution checks: compare feature distributions to training baseline; alert on >2σ drift
- Label quality: check label distribution; flag suspicious imbalance changes
- **If validation fails: block pipeline, alert on-call**

**Feature engineering**:
- Deterministic transformations (no randomness) for reproducibility
- Store feature pipeline as a versioned artifact (pickle or ONNX)
- Use the same feature pipeline at training and serving (training-serving skew is a top failure mode)

**Training**:
- Hyperparameters from last best config (with optional Optuna sweep for major retrains)
- MLflow / W&B for experiment tracking — log all parameters, metrics, artifacts

**Evaluation gate**: don't deploy if:
- Test set AUC drops >2% vs previous deployed model
- Calibration error (ECE) exceeds threshold
- Performance on any monitored demographic subgroup drops >5%

**Deployment**: canary deploy via feature flag; monitor for 24h before full rollout; automated rollback if production metrics degrade.

---

### Q2: Model Debugging

> **Your model's accuracy was 94% last week, now it's 82%. You haven't changed the code. What's happening?**

**Hypothesis: data distribution shift (most common cause)**

**Debugging playbook**:

1. **Verify the accuracy drop is real** — is it a metric computation bug? Check logging code and sampling logic first.

2. **Check input data distribution** — compare feature distributions between last week and current week. Look for:
   - New feature values never seen in training (out-of-distribution inputs)
   - Missing values suddenly appearing in a previously complete feature
   - Upstream data source change (different encoding, different business logic)

3. **Check label distribution** — has the class balance shifted? If positive class went from 10% → 2%, apparent accuracy can drop significantly even if the model is the same.

4. **Segment analysis** — break down performance by time, user segment, region. Is the drop uniform or concentrated in one slice? (Concentrated = upstream data issue for that slice)

5. **Check model serving** — are you serving the right model version? Was a dependency updated that changed preprocessing behavior?

6. **Check for data pipeline bugs** — a join that changed cardinality, a timestamp timezone issue, a feature being filled with NaN due to a schema change upstream.

**Most common culprits**: upstream data schema change, feature encoding change, label definition change in the business system feeding your labels.

---

### Q3: Distributed Training

> **You need to train a 7B parameter model. Your single A100 80GB can almost hold it but training OOMs. How do you handle this?**

A 7B model in FP32 = 28 GB weights + 28 GB gradients + ~84 GB optimizer states (Adam) = 140+ GB. Doesn't fit on a single A100 80GB.

**Solution options**:

**Option 1 — BF16 training** (try first): BF16 weights + gradients = 14+14 = 28 GB. Adam states in FP32 = 56 GB. Still tight but might fit with activation checkpointing.

**Option 2 — Gradient checkpointing**: recompute activations during backward pass instead of storing them. ~30% training slowdown; reduces activation memory from O(layers) to O(√layers).

**Option 3 — DeepSpeed ZeRO Stage 2** (single GPU doesn't help, but if you add a second GPU):
- Stage 1: shard optimizer states (4× memory reduction)
- Stage 2: also shard gradients (8× reduction)
- Stage 3: also shard parameters (N× reduction)

**Option 4 — LoRA/QLoRA**: if this is fine-tuning (not pre-training), use QLoRA — 4-bit base model (~14 GB) + LoRA adapters in BF16 (~1 GB) = fits comfortably on one A100.

**Option 5 — Tensor Parallel (2 GPUs)**: split weight matrices across 2 GPUs; each GPU holds half the model. Requires NVLink for low-latency GPU-to-GPU communication.

For most production fine-tuning: **QLoRA is the right answer** — single GPU, high quality, production-ready.

---

### Q4: Hyperparameter Optimization

> **You have a model that needs tuning but HPO experiments take 6 hours each. How do you efficiently find good hyperparameters?**

**Inefficient approaches**:
- Grid search: exhaustive, exponential in parameter count — impractical
- Random search: better than grid, but wastes budget on poor configurations

**Efficient approaches**:

**1. Bayesian Optimization (Optuna, Hyperopt)**: builds a probabilistic model of the objective function; suggests configurations likely to improve on existing results. 10–50× more sample-efficient than random search.

```python
import optuna

def objective(trial):
    lr = trial.suggest_loguniform("lr", 1e-5, 1e-2)
    weight_decay = trial.suggest_loguniform("wd", 1e-4, 1e-1)
    # ... train and return validation metric

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=30)
```

**2. Early stopping with successive halving (SHA/Hyperband)**: run many configurations; kill the bottom 50% after N steps; repeat. Allocates budget to promising configurations.

**3. Warm-starting**: start HPO from the best known configuration of a similar problem (transfer HPO knowledge).

**4. Proxy tasks for fast iteration**: validate on a 10% data subset first; only run full 6-hour experiments for top-3 configs.

**5. Knowledge-based search space pruning**: use domain knowledge to narrow search space (e.g., learning rate almost always works in [1e-5, 1e-3] for fine-tuning LLMs).

---

### Q5: Model Compression

> **Your model inference is too expensive. How do you compress it without unacceptable quality loss?**

**Compression techniques in order of complexity**:

**1. Quantization** (try first — easiest):
- Post-training quantization (PTQ): INT8 with calibration dataset, ~1% quality loss
- GPTQ/AWQ: 4-bit quantization with ~2-3% quality loss, 4× memory reduction
- QAT (Quantization-Aware Training): best quality for extreme compression, requires retraining

**2. Knowledge distillation**: train a smaller "student" model to mimic the larger "teacher" model. 
- Use teacher logits as soft labels (temperature-scaled probabilities) for richer training signal
- Student can achieve 90–95% of teacher quality at 10–30% of the size

**3. Structured pruning**: remove entire neurons, attention heads, or layers.
- LLM Surgeon, SliceGPT: identify and remove least-important components
- 20–30% parameter reduction with 1–3% quality loss

**4. Unstructured pruning**: zero out individual weights (SparseGPT). Requires sparse matrix operations hardware support for actual speedup.

**Practical recommendation**: quantization (GPTQ/AWQ for LLMs, PTQ for traditional ML) gives 4× memory reduction and 1.5–2× speedup at minimal quality cost — best ROI.

---

### Q6: Feature Engineering

> **You're adding new features to an existing model. How do you ensure you don't introduce training-serving skew?**

Training-serving skew is when features are computed differently at training time vs. inference time — a leading cause of production model degradation.

**Prevention**:

**1. Shared feature pipeline**: the exact same code that computes features at training runs at serving. Not "same logic" — the same artifact.
```
train.py: features = FeaturePipeline().fit_transform(train_data)
serve.py: features = FeaturePipeline.load("v1.2").transform(live_data)
```

**2. Feature stores** (Feast, Tecton, Hopsworks): centralized registry where training retrieves features with `get_historical_features()` and serving retrieves with `get_online_features()`. Same feature definitions, different data stores.

**3. Unit tests for feature transformations**: assert that a known input produces the expected feature vector; run in CI.

**4. Distribution monitoring in production**: log feature distributions in serving; compare to training baseline daily; alert on >3σ drift.

**5. Offline-online consistency checks**: for a sample of inference requests, compute features both ways; assert they match (within float tolerance).

---

### Q7: Reproducible Pipelines

> **How do you ensure that a model training run can be exactly reproduced 6 months later?**

Reproducibility requires pinning every source of variation:

**1. Code versioning**: training script committed to Git; record commit SHA in experiment metadata.

**2. Dependency pinning**: `requirements.txt` with exact versions (`torch==2.1.0` not `torch>=2.0`). Better: Docker image with locked environment.

**3. Data versioning**: data must be immutable or versioned. Options:
- DVC (Data Version Control): version large data files like Git
- Store exact data snapshot with timestamp; use point-in-time queries for time-partitioned data
- Log the S3 paths and their ETag (content hash) used for training

**4. Seed management**: set seeds for all random sources:
```python
random.seed(42); np.random.seed(42); torch.manual_seed(42)
torch.backends.cudnn.deterministic = True
```
Note: deterministic mode may reduce performance — acceptable for reproducibility builds.

**5. Hyperparameter logging**: log all hyperparameters explicitly; do not use config files that can be changed without tracking.

**6. Artifact storage**: save trained model, preprocessor, feature pipeline, and training metrics together as a versioned bundle in MLflow/W&B artifact registry.

**Test of reproducibility**: after 6 months, check out the commit, load the exact Docker image, run training with the logged hyperparameters on the versioned data — metrics should be within floating-point noise.

---

### Q8: Feature Stores and Serving Consistency

> **What is a feature store and when do you need one?**

A feature store is a centralized platform that manages feature definitions, computation, storage, and serving for ML models.

**Components**:
- **Feature registry**: centralized definitions with metadata (type, description, owner, lineage)
- **Offline store**: historical features for training (data warehouse, Parquet files)
- **Online store**: low-latency feature serving for inference (Redis, DynamoDB)
- **Feature pipelines**: computation jobs that keep offline and online stores in sync

**When you need a feature store**:
- Multiple models sharing the same features (avoid duplicated computation)
- Feature reuse across teams (catalog and discovery)
- Training-serving skew problems (shared definitions prevent discrepancies)
- Point-in-time correctness for time-series features (prevent future data leakage in training)

**Popular options**: Feast (open source), Tecton (managed), Hopsworks, Databricks Feature Store, Vertex AI Feature Store.

**Simple alternative**: for smaller teams, a shared library with feature transformation code + unit tests, with training using the offline computation and serving using the same library, achieves 80% of the benefit without the operational overhead.

---

## Preparation Checklist

For ML Engineer interviews, be ready to discuss:

- [ ] Training pipeline design and CI/CD for ML
- [ ] Debugging model degradation (data drift, training-serving skew)
- [ ] Distributed training (FSDP, DeepSpeed ZeRO)
- [ ] Hyperparameter optimization (Optuna, Hyperband)
- [ ] Model compression (quantization, distillation, pruning)
- [ ] Feature engineering and feature stores
- [ ] Experiment tracking (MLflow, W&B)
- [ ] Model evaluation and A/B testing

---

[← AI Engineer](ai-engineer.md) | [AI Architect →](ai-architect.md)
