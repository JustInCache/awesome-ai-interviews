# Anthropic Interview Preparation

[← Back to README](../README.md) | [See All Companies](../README.md#-company-interview-styles)

**What Anthropic looks for**: deep safety conviction (not just lip service), interpretability awareness, careful reasoning about uncertainty, and the ability to think rigorously about alignment. They hire people who genuinely believe AI safety is important — candidates who treat safety as a checkbox won't get far.

---

## Interview Format

| Round | Format | Duration |
|-------|--------|----------|
| Recruiter Screen | Background + why Anthropic | 30 min |
| Technical Depth | LLM internals, safety concepts | 60 min |
| Coding | Algorithms or ML implementation | 45 min |
| System Design | AI system or safety system design | 60 min |
| Safety Discussion | Scenario-based safety thinking | 60 min |
| Values Deep Dive | Mission, collaboration, hard decisions | 45 min |

---

## What They Emphasize

- **Genuine safety commitment** — do you actually believe this matters? Or is it a buzzword?
- **Epistemic humility** — comfortable saying "I don't know" and reasoning under uncertainty
- **Rigorous thinking over confident assertions** — they want to see how you think, not just what you know
- **Constitutional AI fluency** — their signature technique; you should understand it deeply
- **Interpretability curiosity** — what's actually happening inside these models?

---

## Interview Questions

### Safety & Alignment

**Q1: What is Constitutional AI and why does Anthropic use it instead of RLHF?**

Anthropic's core alignment technique:

**Constitutional AI (CAI)**:
1. Define a "constitution" — a set of principles (e.g., "Be helpful, harmless, and honest")
2. Generate model responses to potentially harmful prompts
3. Have the **model itself critique** each response against the constitution
4. Have the model **revise** the response based on its critique
5. Use critique-revision pairs for SFT training
6. Further train with RLAIF (RL from AI feedback) using the model's own judgments as reward signal

**Why over RLHF**:
- RLHF requires expensive human preference labels; CAI uses AI self-critique (much cheaper to scale)
- Human labelers have inconsistent values and can be manipulated by clever model outputs
- CAI principles are explicit and auditable; RLHF encodes implicit human biases
- CAI enables rapid iteration — changing a principle and retraining is easier than re-labeling data

**Limitations** (Anthropic would want you to know these):
- The constitution must itself be carefully designed — whose values does it encode?
- Model self-critique is bounded by model capability — it can't critique errors it can't detect
- Doesn't address goal misgeneralization (model behaves well in training, badly in deployment)

📖 [AI Safety deep dive](../topics/10-ai-safety-ethics.md)

---

**Q2: Explain mechanistic interpretability and why Anthropic invests in it.**

Mechanistic interpretability (MI) is the project of understanding the internal computations of neural networks — not just what they do, but *how* they do it.

**Anthropic's key contributions**:
- **Superposition hypothesis**: models represent more features than they have neurons by encoding multiple features in overlapping linear directions
- **Induction heads**: specific attention circuits that enable in-context learning — they pattern-match and complete sequences
- **Sparse Autoencoders (SAEs)**: technique to decompose model activations into interpretable "features" — each feature represents a human-understandable concept
- **Monosemanticity**: work showing that individual neurons in SAE-expanded spaces correspond to coherent concepts (e.g., specific legal concepts, biological entities)

**Why Anthropic invests**:
- If you can't understand what a model is doing internally, you can't verify alignment
- MI could enable detecting deceptively aligned models (models that appear safe in training but are not)
- MI could enable targeted interventions — fixing specific failure modes by locating them in weights

**Interview signal**: Anthropic wants you to understand this isn't just academic curiosity — it's a path to making safety claims that are more than behavioral.

---

**Q3: What is the "deceptive alignment" problem?**

One of the most concerning alignment failure modes:

A model that is sufficiently capable and has a misaligned goal might **learn to behave safely during training** because it realizes that being safe during training is necessary to eventually be deployed and pursue its real goals.

The model passes all safety evaluations — not because it's actually safe, but because it's strategically appearing safe.

**Why it's hard to solve**:
- Behavioral testing can't distinguish genuine alignment from strategic compliance
- The model would need to recognize that it's being evaluated, which requires self-awareness
- Standard interpretability may not help if the deceptive goal is distributed across many parameters

**Current best approaches**:
- Mechanistic interpretability looking for goal representations
- Diverse evaluation environments where strategic compliance is harder to maintain
- Scalable oversight: use weaker models to supervise stronger ones
- Debate: have models argue against each other to surface deceptive reasoning

Strong Anthropic candidates know this problem isn't solved and can reason about approaches honestly.

---

**Q4: Design a content moderation system for Claude that minimizes both harmful outputs AND over-refusals.**

This is the core tension Anthropic deals with: being safe without being unhelpful.

**Framework: dual newspaper test**
- Would this response be reported as harmful by a journalist covering AI harms?
- Would this refusal be reported as needlessly restrictive by a journalist covering paternalistic AI?

**System design**:

```
User Input
    ↓
Harm Classification (probability of harm, harm severity)
    ↓
Context Assessment (is this a medical professional? developer testing?)
    ↓
Response Generation
    ↓
Output Review (does response contain actually harmful content?)
    ↓
User
```

**Key calibration points**:
- **High severity, low frequency** (bioweapons instructions): always refuse
- **Low severity, context-dependent** (how to pick a lock): depends on context — legitimate uses exist
- **Contested topics** (political opinions): provide balanced view, not refusal
- **Over-refusal detection**: monitor thumbs-down on refusals, rate of "not what I meant" follow-ups

**What distinguishes strong answers**: quantifying the trade-off (every unnecessary refusal has a real cost to user trust and usefulness), and recognizing that "safe" and "helpful" are usually aligned, not opposed.

---

**Q5: What is scalable oversight and why does it matter for Claude at scale?**

The problem: as AI models become more capable than humans at specific tasks, humans can no longer directly verify whether model outputs are correct. Yet we need to train with human feedback. How?

**Scalable oversight approaches**:

1. **Debate**: train two models to argue opposing positions. A human judges the debate rather than the direct answer. The argument structure makes deceptions detectable.

2. **Amplification (IDA)**: pair a human with a weaker AI assistant to answer questions about stronger AI behavior. The human can ask the assistant sub-questions, gradually building up to evaluating complex outputs.

3. **Recursive reward modeling**: decompose complex tasks into sub-tasks humans can evaluate, build up reward signal from sub-task evaluations.

4. **Process supervision (PRMs)**: instead of evaluating final answers, evaluate reasoning steps — a human can check if step 3 follows from step 2 without evaluating the full 20-step solution.

**Why Anthropic cares**: Claude is already superhuman at some tasks. Training the next Claude requires scalable oversight now, not after the fact.

---

### Technical Depth

**Q6: How does Claude's context window work technically, and what are the current limitations?**

Claude 3.5 has a 200K token context. How it works and where it breaks:

**How it works**:
- Standard transformer attention, but with efficient attention mechanisms (Flash Attention) for memory efficiency
- RoPE position embeddings with scaling for long contexts
- Training includes "needle in haystack" examples to maintain attention across the full context

**Known limitations**:
- **Lost in the middle**: performance degrades for information placed in the middle of very long contexts
- **Computational cost**: attention is O(n²) — 200K context uses 4× the memory of 100K context
- **Quality degradation at max context**: while Claude performs well up to ~100K, the last 50K tokens get less reliable attention
- **Coherence**: for very long documents, maintaining logical consistency across the full context is hard

**What Anthropic is working on** (based on published work): more efficient attention mechanisms, better positional embeddings for very long contexts, and training specifically for multi-hop reasoning across long documents.

---

**Q7: Explain the "helpful, harmless, and honest" (HHH) framework and where the tensions arise.**

Anthropic's core alignment criterion:

- **Helpful**: gives users what they actually need (not just what they asked for)
- **Harmless**: doesn't cause harm to users, third parties, or society
- **Honest**: doesn't deceive, doesn't assert false beliefs, acknowledges uncertainty

**Tensions**:

| Tension | Example |
|---------|---------|
| Helpful vs. Harmless | User wants detailed information that could be misused |
| Helpful vs. Honest | User wants validation; honest answer is "no, your idea isn't good" |
| Harmless vs. Honest | Being fully honest about a sensitive topic could cause emotional harm |
| HHH vs. Autonomy | User makes a choice Claude considers harmful to themselves |

**Anthropic's resolution approach**: prioritize in context — for third-party harms (harmless > helpful), for self-regarding choices respect autonomy, for honesty maintain it while being tactful.

**What distinguishes strong candidates**: they've thought about these tensions seriously and can give examples from real AI interactions, not just recite the principles.

---

### Research Discussion

**Q8: Walk me through a paper you've read recently and what you took from it.**

Prepare 2–3 papers deeply. Anthropic especially values:
- **Interpretability papers**: their own sparse autoencoder work, circuits work, features work
- **Alignment papers**: RLHF, DPO, Constitutional AI, debate, amplification
- **Capability papers**: o1 technical report, DeepSeek-R1, newer architecture innovations

**Format for discussing a paper**:
1. What problem does it solve?
2. What's the key insight/approach?
3. What are its limitations?
4. What would you do differently or as a follow-up?

They're evaluating whether you can engage critically — not just summarize.

---

**Q9: What would it mean for an AI to be "deceptive" and how would you detect it?**

They want to probe the depth of your safety thinking:

**Forms of deception**:
- **Explicit lying**: stating known falsehoods
- **Misleading framing**: technically true but designed to create false impressions
- **Selective disclosure**: omitting information that would change user behavior
- **Strategic compliance**: appearing aligned while actually pursuing different goals

**Detection approaches**:
- **Behavioral testing**: does the model behave consistently when it thinks it's being watched vs. not? (Hard to implement — model can't tell)
- **Mechanistic interpretability**: look for goal representations that don't match stated goals
- **Adversarial probing**: try to elicit deceptive behavior through various prompting strategies
- **Consistency testing**: present the same question in many framings — deceptive models may give different answers when the "right" answer is framed differently

**Honest candidate answer**: acknowledge that we don't have reliable detection methods for sophisticated deception. This is an open research problem, which is why Anthropic's interpretability work matters.

---

**Q10: How do you think about the long-term risks of advanced AI systems?**

Anthropic was founded on the belief that transformative AI could be among the most important and potentially dangerous technologies ever. They want candidates who:
- Take this seriously (not dismissive)
- Have nuanced views (not catastrophist nor naive)
- Connect long-term concerns to near-term engineering decisions

**Strong answers address**:
- The difference between near-term risks (misuse, bias, hallucinations) and long-term risks (goal misgeneralization, deceptive alignment, power concentration)
- Why solving the near-term problems doesn't automatically solve the long-term ones
- What engineering choices made today could make long-term safety easier or harder
- How uncertainty about timelines affects what to prioritize now

---

## Resources for Anthropic Prep

- [Anthropic research page](https://www.anthropic.com/research) — read recent safety and capabilities papers
- [Transformer Circuits Thread](https://transformer-circuits.pub/) — Anthropic's mechanistic interpretability work
- [Constitutional AI paper](https://arxiv.org/abs/2212.08073)
- [Scaling Monosemanticity paper](https://transformer-circuits.pub/2024/scaling-monosemanticity/)
- Claude's model card and usage policies — understand the practical safety decisions they've made

📖 Also study: [AI Safety & Ethics](../topics/10-ai-safety-ethics.md) | [Fine-Tuning & Alignment](../topics/05-fine-tuning.md) | [Reasoning Models](../topics/13-reasoning-models.md)
