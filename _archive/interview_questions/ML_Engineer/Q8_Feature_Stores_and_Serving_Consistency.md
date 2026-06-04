````markdown
# ML Engineer Interview Question

## Topic: Feature Stores and Training/Serving Consistency

---

### Question

> Explain how you would design a feature store and serving architecture to ensure training/serving consistency, low-latency feature access, and manageable feature evolution.

---

### Answer

Feature consistency is a common source of production failures. The architecture should separate compute for offline feature materialization and online serving, and enforce clear APIs and lineage.

---

#### Core Components

1. Offline store: batch features materialized for training (parquet/warehouse) with timestamps and versioned feature tables.
2. Online store: low-latency key-value store (Redis, DynamoDB) for serving features at inference time.
3. Feature registry: central catalog with definitions, transformations, owners, and lineage.

#### Consistency Strategies

1. Ensure features are computed with identical logic for offline and online paths by sharing transformation code (e.g., feature SDKs or compiled transformations).
2. Use event-time joins and watermarking to avoid leakage; always materialize features with explicit cut-off timestamps.

#### Feature Evolution

1. Version features, not just code: roll out new feature versions alongside old ones and run shadow traffic to compare.
2. Deprecation and monitoring: track feature importance and drop features safely when unused.

---

#### What This Tests

- Understanding of feature engineering at scale
- Designing consistent training/serving pipelines
- Knowledge of operational trade-offs for latency and correctness

````
