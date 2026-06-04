# Data Scientist Interview Question

## Topic: Experimentation & A/B Testing

---

### Question

> You ran an A/B test for a new recommendation algorithm. The test shows a 3% lift in click-through rate, but revenue is flat. How do you interpret this, and what do you recommend?

---

### Answer

This is a classic "metric divergence" situation—different metrics telling different stories. Before recommending anything, I need to understand **why** clicks went up but revenue didn't follow.

---

#### Step 1: Validate the Results Are Real

First, I'd sanity-check the data:

- **Is the test properly randomized?** Check for bias in user assignment.
- **Is the sample size sufficient?** 3% lift needs enough data to be statistically significant.
- **Are we measuring the same time period?** Revenue might lag clicks.
- **Any technical issues?** Logging bugs, bot traffic, etc.

Assuming the results are valid, the interesting question is interpretation.

---

#### Step 2: Understand the Mechanism

Clicks up, revenue flat. Several possible explanations:

**Hypothesis 1: Lower-quality clicks**

The algorithm drives clicks on lower-value items. Users are clicking more, but on cheaper products or content that doesn't convert to purchases.

How to test: Look at the *value* of clicked items, not just click count. Check add-to-cart rate and purchase rate per click.

**Hypothesis 2: Cannibalization**

Users are clicking on recommendations instead of higher-value items they would have found through search or navigation.

How to test: Look at total user journey. Are users who click recommendations making fewer high-value discoveries elsewhere?

**Hypothesis 3: Engagement without intent**

The algorithm is good at generating curiosity clicks but not purchase-intent clicks. Users browse more but don't buy more.

How to test: Segment by user intent signals. Are high-intent users (previous purchasers, cart-havers) also showing the click lift?

**Hypothesis 4: Saturation effects**

More clicks, but users have a fixed budget. They're now spreading the same spend across more, cheaper items.

How to test: Look at basket size and items per order.

---

#### Step 3: Dig Into the Data

I'd build a diagnostic dashboard:

| Metric | Control | Treatment | Delta |
|--------|---------|-----------|-------|
| Click-through rate | 5.0% | 5.15% | +3% |
| Revenue per session | $2.50 | $2.50 | 0% |
| Revenue per click | $50 | $48.54 | -3% |
| Avg item value clicked | $75 | $65 | -13% |
| Conversion rate | 2.0% | 2.0% | 0% |
| Items per order | 2.1 | 2.3 | +10% |

This hypothetical data tells a story: users are clicking cheaper items, buying more of them, but spending the same total. The algorithm is driving volume, not value.

---

#### Step 4: Form a Recommendation

My recommendation depends on business context:

**If the goal is engagement and retention:**
"Ship it. More clicks means more user engagement with recommendations. Revenue parity means we're not hurting the business. Over time, engaged users may become more valuable."

**If the goal is revenue growth:**
"Don't ship as-is. We've optimized for the wrong metric. We should either:
- Add revenue-weighted signals to the algorithm
- Change the test metric to revenue per session
- Explore a multi-objective approach that balances engagement and value"

**If we're unsure about long-term effects:**
"Extend the test. Short-term revenue might be flat, but increased engagement could drive long-term customer lifetime value. Let's run for 4-6 weeks and look at retention metrics."

---

#### The Conversation with Stakeholders

I'd present this as a decision, not just a data dump:

"The new algorithm successfully drives more engagement—users are clicking 3% more. However, they're clicking on lower-value items, so revenue stays flat.

This gives us a choice:
1. **Ship for engagement** – if we believe engagement drives long-term value
2. **Iterate on the algorithm** – if short-term revenue matters
3. **Run a longer test** – if we need more data on downstream effects

My recommendation is [X], because [reasoning]. But I want to hear your perspective on what we're optimizing for."

---

### What This Question Tests

- Ability to interpret nuanced results (not just "significant or not")
- Diagnostic thinking—multiple hypotheses, ways to test each
- Business acumen—connecting metrics to outcomes
- Stakeholder communication—framing decisions, not just data
