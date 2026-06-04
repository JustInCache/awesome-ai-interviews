````markdown
# Data Scientist Interview Question

## Topic: Handling Label Noise and Weak Supervision

---

### Question

> Many annotation sources are noisy or weak. Describe methods to model label noise, combine weak supervision signals, and still produce reliable metrics and models for production.

---

### Answer

Noisy labels are common; the right approach combines probabilistic modeling of noise, aggregation strategies, and robust evaluation.

---

#### Modeling & Aggregation

1. Use label-modeling frameworks (e.g., Snorkel) to learn accuracies and correlations of weak sources and produce probabilistic labels.
2. Calibrate and threshold probabilistic labels before training; consider training with label uncertainty via loss weighting.

#### Robust Training Techniques

1. Noise-robust loss functions (e.g., generalized cross-entropy) or sample reweighting based on estimated label reliability.
2. Semi-supervised approaches: use a small trusted labeled set to guide training and validation.

#### Evaluation Practices

1. Maintain a held-out high-quality test set for unbiased evaluation.
2. Report metrics with confidence intervals and run sensitivity analyses across noise-level assumptions.

---

#### What This Tests

- Ability to combine weak supervision sources effectively
- Knowledge of robust training and evaluation under noisy labels

````
