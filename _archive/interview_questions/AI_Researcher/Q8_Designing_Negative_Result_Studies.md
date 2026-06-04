````markdown
# AI Researcher Interview Question

## Topic: Designing and Reporting Negative Results

---

### Question

> Negative or null results are common but underreported. How would you design, run, and report experiments whose primary outcome is a negative result, while ensuring scientific rigor and useful lessons for the community?

---

### Answer

Negative results are valuable when documented rigorously. The design should emphasize pre-registration, clear hypotheses, and reproducible artifacts so that the community benefits from what was learned.

---

#### Experimental Design

1. Pre-register hypotheses and experimental protocol, including datasets, metrics, and stopping rules.
2. Ensure sufficient statistical power to detect meaningful effects; report power analysis and confidence intervals.

#### Execution & Reproducibility

1. Use deterministic seeds where possible and publish code, seeds, and environment manifests.
2. Run ablations to confirm the robustness of the null finding across reasonable hyperparameter ranges.

#### Reporting

1. Report negative findings transparently: exact setup, hyperparameters, datasets, and metrics.
2. Provide analysis of potential causes and boundary conditions where the approach might still work.
3. Share artifacts (data splits, minimal repro) so others can verify or extend.

---

#### What This Tests

- Commitment to rigorous empirical science
- Ability to design reproducible experiments
- Judgment about when a negative result is informative vs. inconclusive

````
