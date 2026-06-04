# AI Engineer Interview Question

## Topic: Data Pipeline Engineering

---

### Question

> Your AI system relies on real-time data streams, but the data quality is inconsistent—missing values, duplicates, late arrivals. How do you design data pipelines that ensure reliable AI performance?

---

### Answer

Data pipelines for AI systems are different from traditional ETL. The goal isn't just "clean data"—it's "data that enables reliable model predictions." You need to think about data as a service that models depend on.

---

#### Pipeline Design Principles

**1. Data as a Contract**

Treat data like an API contract:

- Define schemas explicitly (field types, required vs. optional)
- Version data schemas (breaking changes require model retraining)
- Validate data quality at ingestion
- Monitor contract compliance over time

**2. Layered Architecture**

```
┌─────────────────┐
│   Raw Data      │  ← Ingestion layer
│   (untrusted)   │
└─────────┬───────┘
          │
┌─────────▼───────┐
│   Validation    │  ← Quality gates
│   & Cleaning    │
└─────────┬───────┘
          │
┌─────────▼───────┐
│   Feature       │  ← Feature engineering
│   Engineering   │
└─────────┬───────┘
          │
┌─────────▼───────┐
│   Serving       │  ← Model-ready data
│   (cached)      │
└─────────────────┘
```

Each layer has specific responsibilities and failure modes.

---

#### Handling Real-Time Challenges

**Late Arrivals & Out-of-Order Events**

- **Watermarks**: Define "lateness" thresholds (events arriving >5 minutes late are considered late)
- **Triggering**: Use event-time windows instead of processing-time
- **State management**: Handle late data by updating previous aggregations

**Missing Values & Data Gaps**

- **Default values**: Sensible defaults for missing data (0 for counts, mean for continuous)
- **Imputation strategies**: Statistical imputation, forward-fill, model-based
- **Graceful degradation**: Models that can handle missing features

**Duplicates & Inconsistencies**

- **Deduplication**: Unique keys, time-based deduplication
- **Conflict resolution**: Last-write-wins, merge strategies
- **Data quality metrics**: Track duplicate rates, alert when > threshold

---

#### Quality Assurance Framework

**1. Data Validation**

At each pipeline stage:

- Schema validation (required fields present, correct types)
- Range checks (values within expected bounds)
- Cross-field validation (age > 0, dates make sense)
- Statistical checks (distribution drift detection)

**2. Monitoring & Alerting**

- Data freshness (how old is the latest data?)
- Completeness (what % of expected records arrived?)
- Accuracy (sample validation against ground truth)
- Latency (end-to-end pipeline delay)

**3. Automated Recovery**

- **Circuit breakers**: Stop processing if data quality drops below threshold
- **Fallback data**: Use historical aggregates when real-time data fails
- **Graceful degradation**: Reduce model confidence when data quality is poor

---

#### Feature Engineering at Scale

**Online vs. Offline Features**

- **Offline**: Pre-computed features (user lifetime value, historical averages)
- **Online**: Real-time features (current session behavior, recent interactions)

**Consistency Challenges**

- **Training-serving skew**: Features computed differently in training vs. production
- **Feature stores**: Centralized feature computation and serving
- **Versioning**: Feature definitions versioned with models

**Performance Optimization**

- **Materialized views**: Pre-compute expensive features
- **Streaming aggregations**: Real-time statistics (rolling averages, counts)
- **Caching**: Hot features cached for low-latency access

---

#### Example: E-commerce Recommendation Pipeline

```
Raw Events → Validation → Deduplication → Feature Computation → Model Serving

Validation Rules:
- user_id: required, valid format
- product_id: required, exists in catalog
- timestamp: within last 24 hours
- price: > 0, < $10,000

Feature Computation:
- user_total_spend: sum of past purchases
- product_popularity: views in last 7 days
- user_product_similarity: collaborative filtering score
- session_features: current session length, items viewed

Quality Gates:
- Alert if >5% events fail validation
- Alert if feature computation latency >100ms
- Alert if data freshness >10 minutes
```

---

#### Operational Excellence

**1. Testing Strategy**

- **Unit tests**: Individual pipeline components
- **Integration tests**: End-to-end data flow
- **Chaos engineering**: Simulate data quality issues
- **Canary deployments**: Test pipeline changes on subset of data

**2. Documentation & Ownership**

- **Data lineage**: Track data from source to model prediction
- **SLA definitions**: Data freshness, completeness, accuracy SLAs
- **Runbooks**: Procedures for common issues (data source down, schema changes)

**3. Cost Optimization**

- **Incremental processing**: Only recompute what's changed
- **Tiered storage**: Hot data in fast storage, cold data archived
- **Resource scaling**: Auto-scale based on data volume

---

#### Common Pitfalls

- **Over-engineering**: Simple problems don't need complex pipelines
- **Under-monitoring**: Data quality issues discovered too late
- **Tight coupling**: Models that break when data schema changes
- **Performance bottlenecks**: Feature computation that can't scale
- **Lack of testing**: Pipeline bugs discovered in production

---

### What This Question Tests

- Data engineering for ML systems
- Reliability and quality assurance
- Real-time data processing challenges
- Production data pipeline design
