# AI Researcher Interview Question

## Topic: Identifying Research Directions

---

### Question

> How do you decide what research problem to work on? Walk me through your process for identifying promising research directions.

---

### Answer

Research direction selection is arguably the most important skill for a researcher—working on the wrong problem means even excellent execution yields limited impact. Here's how I think about it:

---

#### The Three Lenses

I evaluate potential research directions through three lenses:

**1. Importance: Does solving this matter?**

- What's the downstream impact if this problem is solved?
- Who cares about this problem? (Researchers? Practitioners? Society?)
- Is this a bottleneck for other progress?

A problem can be technically interesting but practically irrelevant. I try to avoid "cute" problems that don't connect to anything larger.

**2. Tractability: Can this plausibly be solved now?**

- Do we have the tools, data, and compute to make progress?
- Are there early signs that existing approaches are close?
- What's changed recently that makes this more tractable than before?

Some problems are important but premature—you'll bang your head against the wall with little progress. The best problems are ones where recent developments have opened new angles.

**3. Fit: Am I well-positioned to work on this?**

- Do I have relevant expertise or unique perspective?
- Can I access necessary resources (data, compute, collaborators)?
- Does this build on my existing work and reputation?

Even important, tractable problems might not be right for *me specifically*.

---

#### Signals I Look For

**Gap between theory and practice**

When practitioners are solving problems with heuristics and hacks, there's often room for principled methods. Example: early prompt engineering was all trial-and-error before systematic research on in-context learning.

**Surprising empirical results**

When something works (or doesn't work) unexpectedly, that's a sign our models of the world are incomplete. Understanding "why" often leads to improvements.

**Recurring pain points**

If the same complaint keeps coming up across papers, talks, and conversations, it's probably a real problem worth solving. Example: evaluation methods for generative models.

**Emerging capabilities**

When models start doing new things, there's a window to study and improve those capabilities. Example: the emergence of in-context learning in large language models.

**Interdisciplinary connections**

Problems that connect ML to other fields (cognitive science, linguistics, physics) often have unexplored angles. Different fields have developed intuitions that might transfer.

---

#### My Concrete Process

**1. Immersion**

Read papers, attend talks, and talk to researchers—but also practitioners. I try to understand:
- What are people excited about?
- What are people frustrated by?
- What are people not talking about but should be?

**2. Question generation**

Keep a running list of questions and observations. Not all will be research-worthy, but the habit of questioning builds intuition.

**3. Quick feasibility checks**

Before committing to a direction, I do small experiments:
- Can I reproduce the phenomenon I want to study?
- Are there obvious baselines that already solve this?
- Is the problem well-defined enough to make progress?

**4. Talk to people**

Discuss ideas with colleagues before investing heavily. They might:
- Point out related work I missed
- Identify flaws in my reasoning
- Suggest angles I hadn't considered

**5. Commit and iterate**

Once I've done due diligence, I commit to a direction—but stay flexible. Research rarely goes as planned, and the best results often come from unexpected pivots.

---

#### Red Flags I Avoid

- **Crowded areas with diminishing returns** – When 50 papers all improve BLEU by 0.3%, it's time to work on something else
- **Problems defined only by benchmarks** – If the problem disappears when you change the benchmark, it wasn't a real problem
- **My solution looking for a problem** – Avoid forcing techniques onto applications where they don't fit
- **Hype-driven research** – Popular ≠ important; sometimes the best opportunities are in unfashionable areas

---

#### Example Decision Process

"I'm interested in improving reasoning in language models. Let me evaluate:

**Importance**: High—reasoning is a key limitation and bottleneck for many applications.

**Tractability**: Promising—chain-of-thought and related methods show reasoning can be improved, suggesting there's room for more principled approaches.

**Fit**: Good—I have background in both language models and structured reasoning.

**My angle**: Most work focuses on prompting; I'll investigate how to improve the underlying reasoning capabilities through training, which is less explored."

---

### What This Question Tests

- Strategic thinking about research
- Awareness of the research landscape
- Ability to self-assess and position
- Intellectual taste and judgment
