# Data Scientist Interview Questions

[← AI Researcher](ai-researcher.md) | [← Back to Home](../README.md)

Scenario-based interview questions for Data Scientist roles in AI-forward organizations.

**Related**: [Evaluation & Testing](../topics/09-evaluation-testing.md) | [AI Safety & Ethics](../topics/10-ai-safety-ethics.md)

---

## What Data Scientists Do

Data Scientists extract insights from data, design experiments, build predictive models, and communicate findings to stakeholders. In AI-forward organizations, they increasingly work alongside LLMs — designing metrics, running A/B tests on generative features, and ensuring data quality pipelines that feed AI systems.

---

## Interview Questions

### Q1: Experimentation Design

> **Your team wants to test whether adding an AI-generated summary to search results increases user engagement. How do you design this experiment?**

**Step 1: Define the primary metric**
- What is "engagement"? Be specific: click-through rate (CTR), session length, return visit rate, conversion?
- Choose one primary metric; everything else is secondary/guardrail.
- Common mistake: optimizing for CTR while degrading session quality.

**Step 2: Define guardrail metrics**
- Metrics that must not degrade: page load time, user satisfaction (CSAT), ad revenue, content quality flags.

**Step 3: Identify the unit of randomization**
- User-level randomization (same user always sees control or treatment): avoids SUTVA violations, correct for user-level decisions.
- Query-level: more power per unit time, but same user can see both experiences — may cause confusion or learning effects.
- Recommendation: user-level for feature experiments; query-level only if the feature has no cross-query memory.

**Step 4: Power analysis**
```
Baseline CTR: 12%
Minimum detectable effect: 1% relative (to 12.12%)
Significance level: α = 0.05
Power: 1 - β = 0.80
Required sample: ~150,000 users per arm
```
Calculate required duration given daily active users. If 100K DAU, need 3+ days minimum (account for weekly seasonality → run 2 full weeks).

**Step 5: Launch plan**
- Start at 5% treatment to detect catastrophic failures early
- Ramp to 50/50 after 48h if no guardrail violations
- Primary metric readout after full power achieved

---

### Q2: Metric Design

> **How do you design metrics for an AI-powered writing assistant? What would you measure?**

Metrics for generative AI features are harder than classic product metrics because quality is subjective and hard to observe directly.

**Tier 1 — Business metrics** (lagging, high confidence):
- User retention: do users who use the AI feature retain better than those who don't?
- Feature adoption: % of eligible users who use the feature at least once, then repeatedly

**Tier 2 — Engagement proxies** (leading, medium confidence):
- Acceptance rate: % of AI suggestions that users keep (without editing)
- Edit distance: how much do users modify AI suggestions? (lower = higher quality OR lower agency)
- Session depth: do users complete more writing tasks per session?

**Tier 3 — Quality signals** (harder to collect, highest signal):
- Explicit feedback: thumbs up/down, star ratings
- Downstream quality: do documents created with AI assistance achieve better business outcomes (e.g., higher email response rates)?

**Anti-metrics** (things you don't want to inadvertently optimize):
- "Acceptance rate" can be gamed by making suggestions so generic users accept them without reading
- "Session length" could increase because users are confused, not productive

**Recommendation**: use a combination of acceptance rate (proxy quality), retention (proxy value), and periodic qualitative user research to calibrate whether proxies are tracking real value.

---

### Q3: Causal Inference

> **You observe that users who use your AI feature have 30% higher LTV. How do you determine if the AI feature *caused* this?**

Correlation ≠ causation. Users who adopt new features are typically power users with higher LTV regardless.

**Approaches to establish causality**:

**1. A/B test** (gold standard): randomly assign access to the feature. If LTV difference persists under randomization, it's causal. Problem: you may already be past the A/B test window.

**2. Instrumental variables**: find a variable that affects treatment (feature access) but affects LTV only through the feature. Example: users who happened to see a feature tutorial email (random assignment of email) are more likely to try the feature.

**3. Difference-in-differences (DiD)**: compare LTV growth of adopters vs. non-adopters before and after adoption. Controls for pre-existing differences if parallel trends assumption holds.

**4. Regression discontinuity**: if the feature was rolled out based on a threshold (e.g., users with account age > 30 days got early access), compare users just above/below the threshold.

**5. Propensity score matching**: match adopters to non-adopters with similar observed characteristics; compare LTV of matched pairs. Controls for observed confounders — doesn't control for unobserved.

**What to report**: "The AI feature is associated with 30% higher LTV. A/B test evidence suggests X% of this is causal; the remainder may reflect selection effects."

---

### Q4: Stakeholder Communication

> **You've found that your AI recommendation system has a significant racial bias, but fixing it will reduce overall recommendation accuracy by 3%. How do you present this to leadership?**

This is a values and ethics question as much as a data question.

**Framework for the conversation**:

**1. Present the facts clearly, without burying the lede**:
- "Our system shows a statistically significant disparity in recommendation quality across racial groups: [Group A] receives recommendations with 15% lower click-through rate despite equivalent preferences."
- "Fixing this costs 3% overall accuracy."

**2. Contextualize the trade-off**:
- Who is harmed by the bias? Quantify it: "Affects approximately 2M users."
- What is the cost of inaction? Legal risk (disparate impact law), reputational risk, trust damage.
- 3% accuracy reduction: what does this translate to in business terms?

**3. Present options, not just problems**:
- Option A: Fix fully — fairness-constrained model, -3% accuracy
- Option B: Partial mitigation — post-processing calibration, -1% accuracy, reduces disparity by 60%
- Option C: Demographic-specific models — separate models per group, complexity cost
- Option D: Do nothing — with explicit documentation of the risk you're accepting

**4. Make a recommendation**: based on legal risk analysis and company values, recommend Option A or B with clear rationale.

**5. Don't let leadership "not decide"**: present a deadline by which action must be taken. Inaction is a decision with consequences.

---

### Q5: Data Quality

> **Your AI model's performance has degraded in production. You suspect a data quality issue. How do you investigate?**

Data quality issues are the most common cause of production model degradation, yet often the last thing investigated.

**Systematic investigation**:

**1. Define "degraded"**: which metric, by how much, since when? Get precise before investigating.

**2. Check data pipeline health**:
- Are all expected data sources arriving? (Missing upstream tables, failed ETL jobs)
- Are row counts as expected? (Sudden drop = missing data; sudden spike = duplicate records)
- Are null rates stable? (Increase in nulls = upstream schema change)

**3. Check feature distributions**:
- Compare mean, std, min, max, and quantiles for each feature between baseline (last known good period) and current
- Look for: new categories in categorical features; out-of-range values in numerical features; sudden 0s or NaN spikes

**4. Check label quality**:
- Has label generation logic changed? (Business rule update in source system)
- Has label distribution shifted? (Class imbalance change)

**5. Check for leakage**:
- Were any features accidentally using future information (leakage from new data pipeline)?

**Tools**: Great Expectations, dbt tests, custom monitoring with Evidently AI.

**Quick win**: add automated data quality gates to the training and serving pipelines. Most data quality issues that reach production do so because there's no automated check to catch them.

---

### Q6: Advanced Analytics for AI Features

> **How do you measure the long-term impact of an AI feature that only shows value over weeks or months?**

Short-term A/B tests don't capture long-term learning effects (users get better with the tool over time) or habit formation.

**Approaches for long-term impact measurement**:

**1. Holdout groups** (most rigorous): designate a permanent holdout (5–10% of users) who never receive the feature. Compare this holdout to the treatment group at 30, 60, 90, 180 days. Expensive (permanently withholding a potentially valuable feature from some users) but gives true causal long-term effect.

**2. Survival analysis**: time-to-event modeling for metrics like "days until churn." Compare survival curves between feature users and non-users. Can be applied retrospectively.

**3. Cohort analysis**: track weekly/monthly metric cohorts for users who adopted the feature at different times. Look for consistent LTV lifts across adoption cohorts (vs. one-time novelty effect).

**4. Time series causal inference (Causal Impact)**: use Google's CausalImpact package to model what the metric would have been without the feature launch, based on pre-launch trends. Good for market-level rollouts without clean controls.

**5. Intent-to-treat analysis**: measure impact on all users who were *offered* the feature, not just those who used it. Avoids selection bias in "feature users vs. non-users" comparisons.

---

### Q7: A/B Testing for Generative AI Features

> **What are the unique challenges of A/B testing generative AI features compared to traditional product features?**

Generative AI features break several standard A/B testing assumptions:

**Challenge 1: Non-binary outcomes**
Traditional features: clicked/didn't click. Generative features: quality exists on a continuous spectrum. Need quality evaluation (LLM-as-judge, human raters) not just engagement metrics.

**Challenge 2: Delayed quality signals**
Users may not know immediately if an AI response was accurate. Quality feedback comes later (did the recommendation work? Was the summary correct?). Experiment duration must accommodate the feedback loop.

**Challenge 3: Novelty effect**
Users often engage more with new AI features because they're novel, not because they're valuable. Run experiments for 4+ weeks to see past the novelty bump.

**Challenge 4: Network effects**
In collaborative features (AI-assisted team documents), treatment of one user can affect control users. Unit of randomization must be the collaboration unit (team/org), not the individual user.

**Challenge 5: Quality vs. engagement decoupling**
High engagement metrics can be achieved by AI that's addictive, sycophantic, or generates low-quality but compelling content. Add quality evaluation as a primary metric, not just engagement.

**Best practice**: define quality metrics (human eval, LLM-as-judge scores on sampled outputs) alongside engagement metrics in the experiment spec before running.

---

### Q8: Label Noise and Weak Supervision

> **Your training dataset has significant label noise — crowdworkers disagreed on 30% of examples. How do you handle this?**

Label noise at 30% is significant and will degrade model quality if not addressed. Strategies:

**1. Measure, don't assume**:
- Compute inter-annotator agreement (Cohen's κ, Krippendorff's α)
- Identify systematic disagreements: is noise random, or concentrated in specific categories/edge cases?

**2. Data cleaning approaches**:
- **Majority vote**: for each example, use the label that ≥3/5 annotators agreed on; discard examples with no majority
- **Confidence filtering**: train an initial model; examples the model is confidently wrong about are likely mislabeled → review those first
- **Cleanlab**: automated label error detection using cross-validated model predictions

**3. Weak supervision (Snorkel)**:
- Encode domain expertise as labeling functions (heuristics, regex, knowledge bases)
- Combine multiple weak signals with a generative model to produce probabilistic labels
- Train on probabilistic labels — model learns to be uncertain where labels are uncertain

**4. Loss function robustness**:
- Symmetric cross-entropy loss: more robust to label noise than standard CE
- Bootstrap loss: replace noisy labels with model's own predictions once confidence exceeds threshold
- Confident learning: only train on examples where model's predicted label agrees with the noisy label

**5. Evaluate carefully**: with noisy labels, standard held-out test accuracy is misleading if the test set also has noise. Clean a small test set manually (200–500 examples) for evaluation.

---

## Preparation Checklist

For Data Scientist interviews, be ready to discuss:

- [ ] A/B test design: randomization, power analysis, guardrail metrics
- [ ] Causal inference: DiD, IV, propensity score matching
- [ ] Statistical significance and effect sizes
- [ ] Metric design and proxy metric risks
- [ ] Data quality monitoring and anomaly detection
- [ ] Communicating trade-offs to non-technical stakeholders
- [ ] Label noise and weak supervision techniques
- [ ] SQL and Python for data analysis

---

[← AI Researcher](ai-researcher.md) | [← Back to Home](../README.md)
