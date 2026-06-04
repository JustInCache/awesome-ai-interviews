# Data Scientist Interview Question

## Topic: Stakeholder Communication & Data Storytelling

---

### Question

> You've completed a complex analysis with nuanced findings—some good news, some bad, some "it depends." The executive wants a clear recommendation in 5 minutes. How do you handle this?

---

### Answer

This is the core challenge of data science in business: translating analytical complexity into actionable clarity without losing the truth. The key is **hierarchical communication**—lead with the decision, support with evidence, have depth ready if needed.

---

#### The Pyramid Principle

Structure communication like a pyramid:

```
         ┌─────────────────┐
         │  Recommendation │  ← Start here
         └────────┬────────┘
    ┌─────────────┼─────────────┐
    │             │             │
┌───▼───┐    ┌────▼────┐   ┌────▼────┐
│Key     │    │Key      │   │Key      │
│Finding │    │Finding  │   │Finding  │
│   1    │    │   2     │   │   3     │
└───┬────┘    └────┬────┘   └────┬────┘
    │              │              │
 Details       Details        Details
```

Lead with the answer, then provide supporting evidence, then have details ready.

---

#### My 5-Minute Structure

**Opening (30 seconds): The Bottom Line**

"Based on the analysis, I recommend we [action]. This will likely [expected outcome] with [confidence level]."

Don't bury the lead. Executives are busy—give them the answer first.

**Middle (3 minutes): The Key Supporting Points**

Three points maximum. For each:
- What we found
- Why it matters
- How confident we are

"The data shows three things:
1. [Finding that supports recommendation]
2. [Finding with nuance, honestly stated]
3. [Risk or caveat they should know about]"

**Closing (1.5 minutes): The "It Depends" & Next Steps**

"The main uncertainty is [key unknown]. If [scenario A], we should [adjust]. If [scenario B], this recommendation holds.

Proposed next step: [concrete action with timeline]."

---

#### Handling Nuance Honestly

The temptation is to oversimplify. Resist it—but be strategic about *how* you convey complexity.

**Don't say**: "It's complicated, there are many factors, results vary by segment..."
(This sounds like you don't have an answer)

**Do say**: "The recommendation is X. It's strongest for [segment/scenario]. For [other segment], we should watch [metric] and potentially adjust."
(This shows you understand nuance but can still decide)

**Technique: Conditional Recommendations**

Instead of: "Maybe we should do X"
Say: "Do X. If we see [signal] after two weeks, pivot to Y."

This gives clear direction while acknowledging uncertainty.

---

#### Anticipating Questions

Before the meeting, prep for likely questions:

| Question Type | How to Prepare |
|---------------|----------------|
| "How confident are you?" | Have specific confidence ranges and what drives uncertainty |
| "What's the downside risk?" | Know worst-case scenarios and their probability |
| "Why not [alternative]?" | Have 2-3 alternatives pre-analyzed with trade-offs |
| "What would change your mind?" | Know what signals would trigger reconsideration |
| "When will we know if it's working?" | Have measurement plan with timeline |

---

#### Common Mistakes to Avoid

**1. Starting with methodology**

❌ "We collected data from three sources, cleaned it, ran a regression..."
✅ "We should invest in channel X. Here's why..."

Nobody cares about your process until they trust your conclusion.

**2. Presenting all findings equally**

❌ "Finding 1, Finding 2, Finding 3, Finding 4, Finding 5..."
✅ "The key insight is X. We also found Y and Z, which are in the appendix."

Prioritize ruthlessly.

**3. Being defensive about uncertainty**

❌ "I'm not sure, the data is messy, it's hard to say..."
✅ "I'm 70% confident in this. The main risk is [X], and here's how we'd detect it."

Confidence intervals are your friend.

**4. Not having a recommendation**

❌ "Here are the options and trade-offs..."
✅ "I recommend option A. Here's why, and here are the trade-offs."

You're paid to have judgment, not just present data.

---

#### Real Example: Converting a Complex Analysis

**The complex finding**: 

"Segment A shows 23% lift but only 15% of users; Segment B shows 8% lift but is 60% of users; Segment C shows negative results but has data quality issues; overall effect depends heavily on assumptions about cannibalization; long-term effects unclear."

**The executive summary**:

"We should proceed with the feature, targeting Segment A first.

The overall lift is positive, but it's concentrated. Segment A—our power users—shows strong 23% improvement. That's enough to justify launch.

We'll watch Segment B closely. Early signs are positive (8% lift) but we need two more weeks to confirm.

The main risk is cannibalization, which could reduce net impact by up to 40%. I'll have better estimates by [date].

Recommended next step: Launch to Segment A this week, expand to B if metrics hold after two weeks."

---

### What This Question Tests

- Communication under constraints
- Ability to prioritize and synthesize
- Comfort making recommendations with uncertainty
- Executive presence and credibility
