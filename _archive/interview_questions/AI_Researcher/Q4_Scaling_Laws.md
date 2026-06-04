# AI Researcher Interview Question

## Topic: Scaling Laws & Resource Allocation

---

### Question

> You have a fixed compute budget for training a new model. How do you think about the trade-off between model size, dataset size, and training duration? What role do scaling laws play in your decisions?

---

### Answer

This is one of the most important practical questions in modern ML research. Scaling laws give us a principled way to make these decisions—but they're tools, not oracles. Understanding their assumptions and limitations is as important as knowing the formulas.

---

#### The Core Trade-off

Given fixed compute C, you can train:
- A larger model for fewer steps
- A smaller model for more steps
- Any point along this frontier

The question: What allocation gives the best model?

---

#### What Scaling Laws Tell Us

**The Chinchilla insight** (simplified):

For compute-optimal training, model size N and dataset size D should scale together:
- D ∝ N (roughly 20 tokens per parameter)

If you train a 10B parameter model, you want ~200B tokens.

**The practical implication**:

Most models before ~2022 were undertrained relative to their size. Chinchilla showed that smaller models trained longer often beat larger models trained less.

But this is for *compute-optimal* training—minimizing loss for a given compute budget.

---

#### When Scaling Laws Don't Apply Directly

**1. Inference cost matters**

Scaling laws optimize for training compute, but you might care about inference:
- A 7B model is much cheaper to deploy than 70B
- You might accept higher training cost for a smaller, faster model

**2. You're not training from scratch**

If you're fine-tuning or starting from a pretrained model:
- Different compute allocation is optimal
- Less data might be needed (knowledge is already there)

**3. Data is limited**

Scaling laws assume you have enough data. If data is limited:
- You might need to repeat data (with diminishing returns)
- Smaller models that don't overfit might be better

**4. You need specific capabilities**

Scaling laws predict average loss, not specific capabilities:
- Some capabilities emerge at specific scales
- Task-specific performance might not follow smooth scaling

---

#### My Decision Framework

**Step 1: Define the objective clearly**

What are you actually optimizing?

| Objective | Implication |
|-----------|-------------|
| Best loss for fixed training compute | Follow Chinchilla scaling |
| Best model for fixed inference budget | Train smaller model longer |
| Best model for specific tasks | May need capability-aware scaling |
| Best model for limited data | Smaller model to avoid overfitting |

**Step 2: Estimate the frontier**

Use scaling laws to predict performance at different configurations:

For compute budget C:
- Config A: 7B model, 200B tokens → predicted loss L1
- Config B: 13B model, 100B tokens → predicted loss L2
- Config C: 3B model, 500B tokens → predicted loss L3

Where is the sweet spot? Scaling laws give you these predictions.

**Step 3: Account for your specific constraints**

Adjust based on practical considerations:
- Do you have enough data?
- What's your inference budget?
- How long can training take? (Wall-clock time constraints)
- What infrastructure do you have? (Memory limits, GPU types)

**Step 4: Validate with small-scale experiments**

Scaling laws are approximations. Before committing full budget:
- Run smaller experiments to verify scaling trends hold
- Check if your specific task follows expected patterns
- Identify any anomalies early

---

#### A Practical Example

**Scenario**: 10,000 GPU-hours budget, goal is best general-purpose model.

**Scaling law analysis**:
- Compute-optimal would suggest ~13B model with ~260B tokens
- This would take X hours on our infrastructure

**Constraints check**:
- We have 500B tokens available ✓
- 13B fits on our GPUs ✓
- But inference cost: We need to serve this cheaply ✗

**Revised decision**:
- Train a 7B model on 350B tokens (over-training relative to compute-optimal)
- Accept ~5% worse training loss
- Get 2x cheaper inference and 2x faster iteration

This is explicitly trading training efficiency for deployment efficiency—scaling laws help quantify the trade-off.

---

#### What I'd Caution Against

**1. Blind faith in scaling laws**

They're empirical fits with assumptions:
- Trained on specific data distributions
- Measure specific metrics (usually perplexity)
- May not hold outside training distribution

**2. Ignoring emergent capabilities**

Some capabilities appear suddenly at scale. If you need a specific capability, you might need to overshoot what scaling laws suggest.

**3. Forgetting about data quality**

Scaling laws assume fixed data quality. Better data can beat more data.

**4. Not validating locally**

Run pilot experiments. Your specific setup might deviate from published scaling laws.

---

### What This Question Tests

- Deep understanding of scaling laws and their assumptions
- Ability to translate theory into practical decisions
- Awareness of real-world constraints beyond pure optimization
- Research maturity—knowing when rules of thumb break down
