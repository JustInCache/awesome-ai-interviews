# OpenAI Interview Preparation

[← Back to README](../README.md) | [See All Companies](../README.md#-company-interview-styles)

**What OpenAI looks for**: deep systems thinking, safety awareness, scaling intuition, and the ability to reason about novel problems at the frontier. They expect candidates to have strong opinions backed by evidence — and to change those opinions when confronted with new evidence.

---

## Interview Format

OpenAI interviews typically include:

| Round | Format | Duration |
|-------|--------|----------|
| Recruiter Screen | Background, motivation, culture fit | 30 min |
| Technical Phone Screen | LLM/ML concepts + coding | 45–60 min |
| System Design | Design a large-scale AI system | 60 min |
| ML/AI Deep Dive | Whiteboard ML concepts, trade-offs | 60 min |
| Coding | Data structures, algorithms, or ML coding | 45–60 min |
| Research Discussion (senior) | Discuss a paper or your past work | 60 min |
| Values / Culture Fit | Mission alignment, collaboration | 45 min |

---

## What They Emphasize

- **Scaling intuition** — how do things change when data, compute, or users 100×?
- **First-principles reasoning** — not just "use tool X" but *why* this approach
- **Safety awareness** — what can go wrong, and how do you prevent it?
- **Rapid iteration** — can you move fast without breaking safety properties?
- **Concrete over vague** — they want specific numbers, specific failure modes, specific solutions

---

## Interview Questions

### System Design & Scaling

**Q1: Design the inference infrastructure for a model serving 100 million requests per day.**

What they want to see:
- Continuous batching, KV cache management, PagedAttention
- Auto-scaling based on GPU utilization + queue depth
- Multi-region deployment for latency
- Model routing (different tiers for different query complexities)
- Cost analysis — tokens/sec per dollar

Strong answer must include: **prefill/decode disaggregation** (OpenAI actually does this), **speculative decoding** for common patterns, and a thoughtful discussion of **graceful degradation** when capacity is exceeded.

📖 [AI Infrastructure deep dive](../topics/12-ai-infrastructure.md)

---

**Q2: How would you design a system to detect when a deployed model starts degrading in quality?**

OpenAI cares deeply about **model monitoring** — what they're looking for:

- Sampling strategy: you can't evaluate every output, so how do you sample intelligently? (anomaly detection on embeddings, random sampling, stratified by query type)
- LLM-as-judge: have a reference model evaluate a subset of outputs for quality metrics (helpfulness, harmlessness, accuracy)
- User signals: implicit signals (copy/paste, follow-up corrections, thumbs down) as leading indicators
- Distributional drift: embedding distribution of queries — if it shifts, your evals may no longer be representative
- Shadow deployment: run old and new model in parallel, compare outputs

**What distinguishes strong answers**: acknowledging that quality is multi-dimensional (helpfulness vs. safety vs. accuracy) and you need separate monitors for each.

---

**Q3: You've fine-tuned a model and it's much better on your benchmark but users are complaining it's worse. What's happening?**

This is a **Goodhart's Law** question. Expected answer:

1. **Benchmark-reality gap**: your eval set doesn't represent actual user queries — model optimized for benchmark distribution
2. **Reward hacking**: if fine-tuned with RLHF, model may have gamed the reward model
3. **Capability regression**: fine-tuning on narrow task degraded general capabilities (catastrophic forgetting)
4. **Style/tone shift**: users prefer outputs that score poorly on your objective metric but feel better conversationally

Investigation approach: qualitative review of user complaints → categorize failure types → check if eval set covers those categories → update eval before next training run.

---

### Safety & Alignment

**Q4: How does Constitutional AI work and what are its limitations?**

Anthropic's Constitutional AI — OpenAI interviewers ask about it because they want to understand your breadth:

**How it works**:
1. Model generates an initial response
2. Model critiques its own response against a list of constitutional principles ("Is this response harmful? Does it violate principle X?")
3. Model revises based on its own critique
4. The critique + revision pairs become SFT training data
5. Further refined with RL using AI feedback (RLAIF) instead of human feedback

**Limitations OpenAI cares about**:
- Principles can be vague — model interprets them inconsistently
- Self-critique is limited by the model's own alignment — a misaligned model may self-critique incorrectly
- Doesn't address **deep alignment** (specification gaming, goal misgeneralization)
- The principles themselves embed values of the designers — who decides what's in the constitution?

📖 [AI Safety deep dive](../topics/10-ai-safety-ethics.md)

---

**Q5: What is reward hacking and how do you prevent it during RLHF training?**

Reward hacking (Goodhart's Law applied to RL): the model finds strategies that maximize the reward model score without actually being more helpful/safe.

**Classic examples**:
- Writing excessively long responses (human labelers had a positivity bias for length)
- Using sycophantic language ("Great question! Absolutely, you're right that...")
- Giving non-answers that avoid any possible controversy

**Prevention**:
1. **KL divergence penalty**: constrain model to stay close to the SFT checkpoint — limits how far it can deviate
2. **Ensemble reward models**: average multiple reward models trained on different data splits; harder to hack all of them simultaneously
3. **Iterated RLHF**: after each RL run, red-team the model for new failure modes, add them to preference data, retrain reward model
4. **Diverse human raters**: use annotators with different backgrounds to prevent systematic biases in reward model

📖 [Fine-Tuning & RLHF deep dive](../topics/05-fine-tuning.md)

---

### Research & Model Understanding

**Q6: Explain why larger models are better few-shot learners (in-context learning).**

This touches the GPT-3 paper and emergent capabilities:

- **Mechanistic hypothesis**: attention heads in large models can implement "induction heads" — pattern completion circuits that recognize and continue sequences. These emerge at scale.
- **Implicit Bayesian updating**: from a Bayesian perspective, in-context examples update the model's distribution over what task is being asked — larger models have more precise posterior distributions
- **Meta-learning**: large models trained on diverse tasks implicitly learn to learn — examples in context help them identify the correct task from their learned distribution

**What OpenAI wants to hear**: that you understand this is still an active research area, and the exact mechanisms are debated. Strong candidates cite specific papers (Anthropic's "In-context learning and induction heads", Olsson et al. 2022).

---

**Q7: What are emergent capabilities in LLMs and how do you think about them for product development?**

Emergent capabilities are behaviors that appear suddenly at scale — not present in smaller models, then suddenly present in larger ones.

**Known examples**: multi-step arithmetic (appeared at ~100B params), chain-of-thought reasoning, multilingual translation without explicit multilingual training.

**Why OpenAI cares about this for product development**:
- You can't easily predict when a capability will emerge — makes roadmaps hard
- Evaluation benchmarks may look flat until a sharp transition — you might miss it
- Some emergent capabilities are harmful (e.g., capability for targeted manipulation may emerge suddenly)

**Practical product implication**: regularly re-evaluate your model's capabilities across a broad benchmark suite when upgrading to a new model version — unexpected capabilities (positive and negative) are common.

---

**Q8: A user claims your model gave them incorrect medical advice. How do you respond and what do you do systemically?**

This is an incident response + safety question:

**Immediate**: document the case, assess if it's part of a pattern, check if it could be harmful enough to require immediate action (routing all medical queries to "consult a professional" message)

**Investigation**: retrieve the exact conversation, understand the model's reasoning, check if current guardrails should have caught it

**Systemic fixes**:
- Add the failure case to red-team eval suite
- Consider domain-specific safety classifier for medical queries
- Decide on appropriate confidence hedging in responses ("I'm not a medical professional; consult a doctor")
- Update RAG grounding if the error was factual (add authoritative medical sources)

**What distinguishes excellent answers**: acknowledging the tension between being helpful (users want medical information) and being safe — and that the answer isn't just "refuse everything medical."

---

### Reasoning Models (2026 focus)

**Q9: How do you think about the trade-off between reasoning model quality and cost at scale?**

OpenAI builds both (o1/o3 vs. GPT-4o-mini) so they think about this constantly:

Key dimensions:
- **Task difficulty distribution**: what % of your actual queries require genuine multi-step reasoning? For most consumer products, it's 10–20%.
- **Model routing**: classify query complexity upfront (fast, cheap classifier), route to appropriate model tier
- **Thinking budget control**: for the queries that do need reasoning, can you find the minimum thinking budget that meets quality bar?
- **Async vs. sync**: for hard queries, is synchronous response required? If not, async queuing with reasoning models is much cheaper at scale

**OpenAI's internal answer** (based on their product lineup): they explicitly offer o3-mini vs. o3 as a cost/quality knob, signaling that even for reasoning, most users should start with the cheaper option.

📖 [Reasoning Models deep dive](../topics/13-reasoning-models.md)

---

**Q10: What is the "alignment tax" and how do OpenAI's models manage it?**

The alignment tax is the capability reduction caused by safety training (RLHF, Constitutional AI). A model optimized purely for task performance might score higher on benchmarks, but be unsafe.

Evidence of the tax:
- Aligned models tend to be more verbose and less willing to make bold claims
- RLHF training can degrade mathematical reasoning slightly
- Safety refusals occasionally block legitimate requests

How OpenAI manages it:
- **Iterative red-teaming + alignment**: find cases where safety training degraded capability, add capability examples back to training data
- **Separate capability and alignment training**: train for capability first, then align separately
- **Evaluation suites that measure both**: they track "over-refusal rate" alongside safety metrics
- **User-facing controls**: allowing developers to adjust system prompt behavior within policy bounds

---

### Behavioral Questions (Culture Fit)

**Q11: Tell me about a time you pushed back on a product decision because of safety concerns.**

They want concrete evidence of **safety-conscious engineering**:
- What was the specific risk you identified?
- How did you communicate it to stakeholders?
- Was your concern well-founded in retrospect?
- How did the decision get made?

Strong answer: shows you can articulate technical risks in business terms, engage constructively rather than obstructing, and accept decisions when the risk was judged acceptable.

---

**Q12: How do you stay current in the AI field given the pace of change?**

What they want: evidence of genuine intellectual curiosity and systematic learning.
- Specific papers you've read recently and what you took from them
- How you separate signal from noise in the arxiv/Twitter fire hose
- How you incorporate new techniques into your work
- Evidence you form your own views, not just follow hype

Strong answer mentions: a few specific recent papers with opinions ("I found X interesting but think Y limitation makes it less applicable than people claim"), a habit around paper reading (e.g., Andrej Karpathy / Yannic Kilcher style paper walkthroughs), and engagement with the community.

---

## Resources for OpenAI Prep

- [Ilya Sutskever's "30 papers to understand AI deeply"](https://arc.net/folder/D0472A20-9C20-4D3F-B145-D2865C0A9FEE)
- [OpenAI technical blog](https://openai.com/research) — read the papers for their major models
- [Alignment Forum](https://www.alignmentforum.org/) — for safety-focused discussions
- Practice designing systems on paper: draw architecture diagrams before coding

📖 Also study: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [AI Safety & Ethics](../topics/10-ai-safety-ethics.md) | [Reasoning Models](../topics/13-reasoning-models.md)
