# Data Scientist Interview Question

## Topic: Causal Inference

---

### Question

> The marketing team ran a promotional campaign and sales went up 15%. They want to claim the campaign caused the increase. What questions do you ask, and how do you assess whether the causal claim is valid?

---

### Answer

This is the classic "correlation vs. causation" problem, but in a business context where people desperately want to claim causation. My job is to be the skeptic who helps us understand what we actually know.

---

#### The Key Question: What Would Have Happened Without the Campaign?

Causal inference is fundamentally about counterfactuals. The campaign "caused" the 15% increase only if sales would have been 15% lower without it.

We can never observe the counterfactual directly—but we can try to estimate it.

---

#### My Initial Questions

**1. What was the comparison?**

- "Sales went up 15%" compared to what?
- Last month? Last year same period? A forecast?
- The baseline matters enormously.

**2. What else changed?**

- Was this a seasonal period? (Holiday shopping, back-to-school, etc.)
- Did competitors change anything? (Price changes, stockouts?)
- Any external events? (News, weather, economic shifts?)
- Did we change anything else? (Pricing, product availability, website?)

Multiple changes make attribution nearly impossible.

**3. Who was exposed to the campaign?**

- Everyone, or a specific segment?
- How were people selected? (Random? Geographic? Self-selected?)
- Selection method determines what inferences we can draw.

**4. What does the trend look like?**

- Was sales growth accelerating before the campaign?
- Did it spike and return to baseline, or shift to a new level?
- A spike that immediately reverts suggests promotion pull-forward, not new demand.

---

#### Hierarchy of Causal Evidence

**Level 1: Randomized Experiment (Gold Standard)**

Was this structured as an A/B test?
- Random assignment to treatment/control
- Compare outcomes between groups
- Difference is causal effect

If yes: We have strong causal evidence (assuming proper execution).

**Level 2: Natural Experiment**

Did something create pseudo-random variation?
- Staggered rollout across regions
- Technical issues that affected some users
- Policy changes with clear cutoffs

If yes: We might be able to construct valid comparisons.

**Level 3: Observational with Controls**

Can we identify comparable non-exposed groups?
- Similar customers who weren't exposed
- Similar time periods without campaigns
- Statistical matching to create comparison groups

If yes: We can estimate effects, but with more assumptions.

**Level 4: Before/After Only**

We only have "before campaign" vs. "after campaign."

This is the weakest evidence. Many things could explain the change.

---

#### Diagnostic Analyses I'd Run

**Timing analysis**: Does the sales increase align precisely with campaign timing? Or did it start before, suggesting other causes?

**Effect heterogeneity**: Is the effect consistent across segments? If the campaign ran on social media but sales increased equally among non-social-media users, something's off.

**Dose-response**: Did more exposure lead to more effect? If heavy-exposure regions show same lift as light-exposure regions, the campaign might not be the cause.

**Parallel trends**: Before the campaign, were treatment and comparison groups trending similarly? If not, the comparison is invalid.

**Placebo tests**: Can I find "effects" in periods or groups where there shouldn't be any? If yes, my methodology is flawed.

---

#### Honest Assessment Framework

| Evidence Level | Confidence | Language to Use |
|----------------|------------|-----------------|
| Proper RCT | High | "The campaign caused a 15% increase" |
| Natural experiment | Medium-High | "Evidence suggests the campaign drove ~15% lift" |
| Observational with controls | Medium | "The campaign is associated with 15% higher sales, likely contributing to the increase" |
| Before/after only | Low | "Sales increased 15% during the campaign period, but we cannot isolate the campaign's contribution" |

---

#### The Conversation with Marketing

"Great question—let's dig into what we can confidently claim.

The 15% increase is real, and it coincides with the campaign. But to claim the campaign *caused* it, we need to rule out other explanations.

Here's what I found:
- [Timing analysis result]
- [Comparison with control group / similar periods]
- [Effect consistency check]

Based on this, I'd say [confidence level] that the campaign contributed [estimated amount]. The honest range is probably [X% to Y%].

For future campaigns, if we set up a proper holdout group, we could get much cleaner measurement. Want me to design that?"

---

### What This Question Tests

- Causal reasoning and counterfactual thinking
- Healthy skepticism without being obstructive
- Knowledge of causal inference methods
- Ability to communicate uncertainty to stakeholders
