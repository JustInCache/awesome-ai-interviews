````markdown
# ML Engineer Interview Question

## Topic: Reproducible Pipelines Across Environments

---

### Question

> How do you ensure that a training pipeline is reproducible across developer machines, CI, and production, including data, code, and environment differences? Provide practical tooling and safeguards.

---

### Answer

Reproducibility requires controlling three axes: code, data, and environment. Build deterministic pipelines, provenance tracking, and environment capture.

---

#### Practical Measures

1. Environment capture: containerize with pinned base images and explicit dependency manifests (`requirements.txt`, `pyproject.toml`, or `environment.yml`).
2. Data versioning: store dataset snapshots or use content-addressed storage; record dataset hashes with each experiment.
3. Deterministic training: fix random seeds, control nondeterministic ops (e.g., GPU nondeterminism flags), and record hyperparameters and RNG state.
4. Provenance & metadata: log model artifacts, code commit SHA, data hash, and environment spec for every run.

#### Tooling Suggestions

1. Use workflow managers (Airflow, Kubeflow, Dagster) for orchestration and reproducible DAGs.
2. Use experiment tracking (MLflow, Weights & Biases) and artifact stores (S3, GCS) with immutable paths.

---

#### What This Tests

- Practical engineering to guarantee reproducibility
- Familiarity with industry tools and deterministic ML practices

````
