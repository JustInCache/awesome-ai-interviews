# Data Scientist Interview Question

## Topic: Metric Design & Trade-offs

---

### Question

> The product team wants to measure "user satisfaction" for an AI feature. How do you approach designing a metric for something this ambiguous?

---

### Answer

"User satisfaction" is a goal, not a metric. My job is to translate this fuzzy objective into something measurable, valid, and actionable—while being honest about what we can and can't capture.

---

#### Step 1: Unpack What "Satisfaction" Means

I'd start by asking questions:

- **What triggered this request?** Are users complaining? Are we launching something new? Is this for goal-setting?
- **What decisions will this metric drive?** Prioritization? Launch/no-launch? Performance reviews?
- **What does a "satisfied" user look like behaviorally?** How would we recognize one?
- **What's the time horizon?** Are we measuring session-level satisfaction or long-term happiness?

The answers shape what kind of metric makes sense.

---

#### Step 2: Map the Metric Landscape

There's no single "satisfaction" metric—there's a family of options with different trade-offs:

**Direct Metrics (Ask Users)**

| Metric | Pros | Cons |
|--------|------|------|
| Thumbs up/down on responses | Easy to collect, specific | Low response rate, selection bias |
| Post-session survey (1-5 stars) | More context, less biased | Interrupts UX, still low response |
| NPS-style survey | Standard, benchmarkable | Noisy, hard to act on |

**Behavioral Proxies (Observe Users)**

| Metric | Pros | Cons |
|--------|------|------|
| Task completion rate | Objective, measurable | Doesn't capture quality of experience |
| Session length | Easy to measure | Ambiguous: long = engaged or stuck? |
| Return usage | Shows ongoing value | Lagging indicator, many confounds |
| Abandonment rate | Clear negative signal | Doesn't capture positive satisfaction |
| Regenerate/edit rate | Shows immediate dissatisfaction | Might also indicate feature discovery |

**Composite Metrics**

Combine multiple signals into a single score. More robust, but harder to interpret and debug.

---

#### Step 3: My Recommendation Framework

I'd propose a **layered approach**:

**Primary Metric: Task Success Rate**

Did the user accomplish what they came to do? This requires defining "success" for the feature, but it's the most direct measure of value delivered.

For an AI assistant: Did the user accept the answer, or did they need to go elsewhere?

**Secondary Metrics: Quality Signals**

- **Explicit feedback rate** – What % of users give any feedback (not just positive)?
- **Negative signal rate** – Regenerate, edit, abandon, or report rates
- **Follow-up query rate** – Do users need to ask clarifying questions?

**Guardrail Metrics: Watch for Harm**

- **Error rate** – How often does the feature fail completely?
- **Support ticket rate** – Are users escalating to humans?

---

#### Step 4: Address the Validity Question

Any satisfaction metric needs validation:

**Does it correlate with what we expect?**
- If we deliberately make the feature worse, does the metric go down?
- Do users who rate highly also behave like satisfied users (return, recommend)?

**Is it robust to gaming?**
- If we optimize hard for this metric, could we make the product worse while improving the number?
- Example: Optimizing for session length might make us add friction.

**Does it capture enough signal?**
- If only 2% of users give feedback, is that sample representative?
- What's the variance? Can we detect meaningful changes?

---

#### Step 5: Set Expectations

I'd be clear about what the metric can and can't do:

**It can:**
- Give directional signal about whether changes help or hurt
- Identify major problems quickly
- Enable comparison across time periods or user segments

**It can't:**
- Capture everything about satisfaction
- Tell you *why* users feel a certain way
- Replace qualitative research for deep understanding

---

#### The Conversation with Product

"Here's what I propose:

**Primary metric**: Task success rate—whether users accomplished their goal without leaving or escalating.

**Supporting metrics**: Explicit feedback rate, regenerate rate, and return usage.

This gives us a measurable proxy for satisfaction. But let's be clear: no single metric captures 'satisfaction.' I'd also recommend quarterly user interviews to catch things metrics miss.

What matters most for your decision-making: fast signal on new changes, or long-term trend tracking?"

---

### What This Question Tests

- Ability to translate ambiguous goals into concrete metrics
- Understanding of metric properties (validity, bias, gaming)
- Trade-off thinking across different measurement approaches
- Stakeholder communication—managing expectations, framing options
