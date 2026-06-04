````markdown
# Data Scientist Interview Question

## Topic: A/B Testing for Generative Features

---

### Question

> You're launching a generative feature (e.g., an AI assistant suggestion) that changes user workflows. How would you design an A/B test to measure success, handle novelty effects, and ensure you capture long-term utility and harms?

---

### Answer

Designing A/B tests for generative features requires careful metric selection, experiment duration planning, and monitoring for unintended consequences.

---

#### Metrics

1. Primary business metrics: task completion rate, time-to-task-completion, conversion or retention impact.
2. Safety/quality metrics: rate of flagged outputs, user-reported satisfaction, incidence of harmful suggestions.
3. Engagement metrics: feature usage frequency, session length (interpret cautiously).

#### Experiment Design

1. Use randomized assignment with user-level bucketing to avoid leakage across sessions.
2. Run a pre-experiment calibration period to collect baseline behavior and detect novelty/adaptation effects.
3. Implement burn-in and cooldown windows to separate short-term novelty from steady-state effects.

#### Handling Novelty Effects

1. Monitor metric trajectories over time and analyze early vs. late cohorts separately.
2. Consider sequential testing with pre-specified analysis points and correction for peeking.

#### Safety and Post-hoc Analysis

1. Run subgroup analyses to detect disparate impacts across user segments.
2. Maintain a rapid rollback mechanism if safety metrics deteriorate.

---

#### What This Tests

- Ability to design experiments that capture both immediate and sustained effects
- Skill in balancing business metrics with safety and fairness concerns

````
