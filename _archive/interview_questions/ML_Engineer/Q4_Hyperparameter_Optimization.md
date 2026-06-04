# ML Engineer Interview Question

## Topic: Hyperparameter Optimization

---

### Question

> You have a limited compute budget and need to find good hyperparameters for a new model architecture. How do you approach this efficiently?

---

### Answer

Hyperparameter optimization is often the difference between "this doesn't work" and "state-of-the-art results." But it's also where teams burn enormous compute budgets. The key is being **strategically lazy**—don't search what you can transfer, don't tune what doesn't matter.

---

#### My Hierarchy of Approaches

**Level 1: Don't Search—Transfer**

Before searching anything, ask:
- Has someone tuned a similar model/task? Use their hyperparameters as starting point.
- Are there well-established defaults? (Adam lr=1e-4, batch size powers of 2, etc.)
- Can I use a smaller proxy task to narrow the range?

The best hyperparameter search is the one you don't run.

**Level 2: Identify What Matters**

Not all hyperparameters are equal. In rough order of impact:

| High Impact | Medium Impact | Low Impact (usually) |
|-------------|---------------|---------------------|
| Learning rate | Weight decay | Adam β2 |
| Batch size | Warmup steps | Gradient clipping threshold |
| Model size | Dropout | Label smoothing |
| Training duration | Layer norm position | Attention dropout |

Focus search budget on high-impact parameters. Use defaults for the rest.

**Level 3: Efficient Search Strategy**

When you do need to search:

---

#### Phase 1: Quick Exploration (10% of budget)

**Goal**: Narrow the search space, eliminate obviously bad regions.

**Method**: Random search with wide ranges, short training runs.

Why random over grid? Random is more efficient when some parameters matter more than others (which is almost always true).

```
Example search space:
- Learning rate: log-uniform(1e-5, 1e-2)
- Batch size: choice(32, 64, 128, 256)
- Weight decay: log-uniform(1e-6, 1e-1)
- Warmup ratio: uniform(0, 0.1)
```

Run 20-30 short trials (10% of full training). Identify which regions look promising.

**Key insight**: Early training loss is predictive of final performance. You don't need to train to convergence to rank hyperparameters.

---

#### Phase 2: Focused Refinement (30% of budget)

**Goal**: Find good hyperparameters in the promising region.

**Method**: Bayesian optimization or successive halving in narrowed space.

**Successive Halving (Hyperband)**:
1. Start many configurations with small budget
2. Evaluate, keep top 50%, double budget
3. Repeat until one configuration remains

This automatically allocates more resources to promising configurations.

**Bayesian Optimization**:
1. Fit a surrogate model (Gaussian process) to observed results
2. Use acquisition function to choose next point to try
3. Balance exploration vs. exploitation

Works well when evaluations are expensive and search space is continuous.

---

#### Phase 3: Final Validation (60% of budget)

**Goal**: Train best configuration(s) to convergence, validate on holdout.

**Method**: Full training runs with top 3-5 configurations from Phase 2.

- Run multiple seeds to measure variance
- Use proper validation set (not the one used during search)
- Check learning curves for signs of overfitting or underfitting

---

#### Practical Tricks

**1. Learning rate finder**

Before any search, sweep learning rate on a single short run:
- Start very small, increase exponentially
- Plot loss vs. learning rate
- Pick value where loss decreases fastest (usually 1/10 of where it diverges)

This gets you in the right ballpark instantly.

**2. Batch size / learning rate scaling**

When changing batch size, scale learning rate proportionally (linear scaling rule):
- 2× batch size → 2× learning rate (approximately)

This reduces the search space significantly.

**3. Progressive training**

Tune on small model or data subset first:
- Smaller model → faster iterations
- Optimal hyperparameters often transfer to larger scale

But verify! Some hyperparameters don't transfer (especially learning rate for very different scales).

**4. Use validation loss, not training loss**

Training loss can be misleading (overfitting). Always use validation metrics for selection.

---

#### Budget Allocation Example

Total budget: 100 GPU-hours

| Phase | Budget | Runs | Length |
|-------|--------|------|--------|
| LR finder | 1 hour | 1 | 1 hour |
| Random exploration | 10 hours | 30 | 20 min each |
| Bayesian refinement | 25 hours | 10 | 2.5 hours each |
| Final validation | 64 hours | 4 | 16 hours each (full training) |

---

#### What I'd Avoid

- **Exhaustive grid search**: Exponentially wasteful
- **Tuning everything at once**: Focus on what matters
- **Using test set during tuning**: Leads to overfitting to test set
- **Trusting single runs**: Variance can mislead you
- **Over-optimizing for proxy metrics**: Ensure proxy correlates with real goal

---

### What This Question Tests

- Efficient resource allocation under constraints
- Knowledge of hyperparameter sensitivity patterns
- Practical experience with search strategies
- Understanding of train/validation/test hygiene
