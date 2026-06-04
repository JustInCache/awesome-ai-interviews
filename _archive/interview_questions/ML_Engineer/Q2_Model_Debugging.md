# ML Engineer Interview Question

## Topic: Debugging Model Performance

---

### Question

> Your model performs great on the validation set but poorly in production. Walk me through how you'd diagnose and fix this.

---

### Answer

This is the classic **train-serve skew** problem—one of the most common and frustrating issues in ML. The gap between offline metrics and online performance usually comes from one of a few root causes.

---

#### Step 1: Confirm the Problem is Real

Before diving deep, I'd verify:

- **Are we measuring the same thing?** Offline accuracy vs. online click-through rate might not be directly comparable.
- **Is there enough production data?** Small sample sizes can make things look worse than they are.
- **Did something else change?** New UI, different user population, seasonal effects?

---

#### Step 2: Systematic Diagnosis

I'd check these in order, from most common to least:

**1. Data Distribution Shift**

The most common cause. Production data looks different from training data.

How to detect:
- Compare feature distributions between training and production
- Look at prediction confidence—lower confidence often signals OOD data
- Check for new categories, missing values, or range violations

Example: Model trained on data from 6 months ago, but user behavior has shifted.

**2. Feature Computation Differences**

The features computed at training time don't match what's computed at serving time.

Common culprits:
- **Time-based features** calculated differently (training uses batch, serving uses real-time)
- **Aggregations** computed over different windows
- **Missing data handling** differs between pipelines
- **Numerical precision** differences (float32 vs float64)

How to detect:
- Log features at serving time and compare to training features for the same examples
- Unit tests that verify feature parity

**3. Preprocessing Inconsistencies**

Tokenization, normalization, or encoding done differently.

Examples:
- Text lowercased in training but not in serving
- Different tokenizer versions
- Categorical encoding applied in wrong order

**4. Label Leakage in Training**

The model learned patterns that aren't available at prediction time.

Example: Using a feature that's computed *after* the event you're predicting.

This shows up as suspiciously good offline metrics that don't transfer.

**5. Serving Infrastructure Issues**

Less common but worth checking:
- Model loaded incorrectly (wrong version, corrupted weights)
- Input parsing errors (JSON field ordering, encoding)
- Timeout causing partial processing

---

#### Step 3: Build Debugging Infrastructure

To diagnose efficiently, you need:

1. **Feature logging** – Record the exact features used for each prediction
2. **Prediction logging** – Save inputs, outputs, and confidence scores
3. **Shadow mode** – Run new models alongside production without serving results
4. **Offline replay** – Ability to re-run production traffic through the training pipeline

---

#### Step 4: Fixing the Issue

Once diagnosed, fixes depend on the cause:

| Problem | Fix |
|---------|-----|
| Distribution shift | Retrain on recent data, continuous training pipeline |
| Feature skew | Unify feature computation, use feature stores |
| Preprocessing mismatch | Single preprocessing module shared by train/serve |
| Label leakage | Audit feature timelines, proper temporal validation |

---

### A Real Example I'd Walk Through

"Let's say I found that a 'user_activity_score' feature has different distributions:

- Training: mean=0.5, std=0.2
- Production: mean=0.3, std=0.4

I'd investigate: Is this a computation difference, or real distribution shift?

If computation: Fix the serving pipeline to match training.
If real shift: Either retrain with recent data, or make the model more robust to this feature's distribution."

---

### What This Question Tests

- Systematic debugging approach
- Knowledge of train-serve skew causes
- Understanding of ML infrastructure requirements
- Practical experience with production ML systems
