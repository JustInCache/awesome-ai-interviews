# AI Researcher Interview Questions

[← AI Architect](ai-architect.md) | [Data Scientist →](data-scientist.md)

Scenario-based interview questions for AI Research roles — methodology, experimental rigor, and advancing the field.

**Related**: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [Evaluation & Testing](../topics/09-evaluation-testing.md) | [AI Safety & Ethics](../topics/10-ai-safety-ethics.md)

---

## What AI Researchers Do

AI Researchers advance the state of the art in machine learning and AI. They design and run experiments, evaluate novel methods rigorously, and publish findings. At companies, they also bridge the gap between academic advances and production systems.

---

## Interview Questions

### Q1: Evaluation Methodology

> **You've developed a new language model technique that seems to outperform the baseline on your test set. How do you validate that this is a genuine improvement?**

The history of ML is littered with improvements that didn't generalize. Rigorous validation:

**1. Multiple evaluation sets**:
- In-distribution test set (same distribution as training): expected to show improvement
- Out-of-distribution test sets: does it generalize beyond training distribution?
- Established public benchmarks (MMLU, HellaSwag, MATH): allows comparison to published work

**2. Statistical significance**:
- Run multiple seeds (at least 5); report mean ± std. A 1% improvement that's within variance isn't an improvement.
- Bootstrap confidence intervals for aggregate metrics
- Paired significance tests (McNemar's test for binary outcomes; Wilcoxon signed-rank for continuous)

**3. Ablation studies**:
- Remove each component of your method independently. Does removing it hurt? By how much?
- Ablations prevent the "kitchen sink" problem where an apparent improvement is actually just more compute/data.

**4. Failure case analysis**:
- Where does your method perform *worse* than baseline? Understanding failures is as important as successes.
- Is the improvement uniform or concentrated in specific examples?

**5. Compute-controlled comparison**:
- Does your method use the same compute as the baseline? Improvements that require 2× compute should be compared against baselines with the same 2× compute budget.

---

### Q2: Research Direction

> **How do you identify high-value research directions in AI? What makes a research problem worth pursuing?**

Research direction selection is one of the highest-leverage decisions a researcher makes. Framework:

**Value dimensions**:

1. **Scientific importance**: does this advance fundamental understanding? Does it resolve an open theoretical question?

2. **Practical impact**: will this change how systems are built in 2–3 years? The best research operates at the intersection of theoretical insight and practical applicability.

3. **Timing**: is the field ready for this? (1) necessary prerequisites exist, (2) compute is available, (3) enough researchers are adjacent to build on your work.

4. **Tractability**: can you make meaningful progress in a reasonable timeframe? Long-shot moonshots are valuable, but a portfolio of research should include tractable near-term wins.

**Signals of high-value problems**:
- Practitioner pain points that lack theoretical explanation (e.g., "why does RLHF cause sycophancy?")
- Surprising empirical observations that theory can't explain (e.g., grokking, emergent abilities)
- Techniques that work empirically but have no principled justification (a good target for rigorous explanation)
- Scaling behavior questions — how does X scale with compute, data, or model size?

**Signals to avoid**:
- Incremental improvements on well-established leaderboards with no conceptual novelty
- Problems where the evaluation is easier to game than to genuinely solve

---

### Q3: Handling Negative Results

> **Your experiment failed to show improvement. You've spent 3 months on this direction. What do you do?**

Negative results are undervalued in AI research but are essential scientific contributions.

**Don't abandon immediately — diagnose first**:

1. **Implementation bugs**: is the code correct? Reproduce the baseline first — if you can't reproduce the baseline, your comparison is invalid.

2. **Hyperparameter sensitivity**: did you tune both baseline and proposed method equally? An untuned baseline is an unfair comparison.

3. **Evaluation metric mismatch**: is your metric capturing what you care about? Maybe the method improves on dimensions your metric doesn't measure.

4. **Scale dependence**: does your method require more data, compute, or model size to show improvement? Test at different scales.

**If it's genuinely not working**:

1. **Understand *why*** — a well-characterized failure is valuable. "Technique X doesn't work because of Y" saves other researchers 3 months.

2. **Document and share**: negative results written up rigorously are publishable (EMNLP, NeurIPS workshops have tracks for this). The community needs to know what doesn't work.

3. **Pivot strategically**: what did you learn that opens new directions? Failed experiments often reveal surprising properties of the problem that suggest better approaches.

4. **Sunk cost fallacy**: 3 months is not a reason to continue if you have strong evidence the direction is dead. Reset with fresh eyes.

---

### Q4: Scaling Laws

> **Explain scaling laws and how you would use them to plan your next model training run.**

Scaling laws (Kaplan et al., 2020; Hoffmann et al., 2022 — Chinchilla) describe how model performance changes with scale:

**Kaplan scaling laws** (OpenAI):
$$L(N, D) \approx \left(\frac{N_c}{N}\right)^{\alpha_N} + \left(\frac{D_c}{D}\right)^{\alpha_D}$$

Loss decreases as a power law with number of parameters N and training data D.

**Chinchilla scaling laws** (DeepMind): for optimal performance per compute budget, the number of training tokens should be approximately 20× the number of parameters. GPT-3 (175B params, 300B tokens) was undertrained; a 70B model trained on 1.4T tokens (Chinchilla-optimal) matches or beats it.

**Using scaling laws to plan a training run**:

1. **Compute budget first**: what's your FLOPs budget? Compute ≈ 6 × N × D.
2. **Optimal allocation**: given budget C, optimal N* = C^0.5 / 6.4^0.5, optimal D* = C^0.5 × 6.4^0.5 (Chinchilla ratios)
3. **Run small scaling experiments first**: train 5 models at different N (100M, 300M, 1B, 3B, 10B) on small data; fit a power law; extrapolate to predict 100B performance. Validate before committing compute.
4. **Monitor actual vs. predicted loss**: if actual loss deviates significantly from predicted, investigate — likely a data quality or training stability issue.

**Scaling laws have limits**: they don't capture emergent capabilities, reasoning improvements, or instruction-following quality. They're tools for compute allocation, not oracles.

---

### Q5: Novel Architecture Design

> **How would you approach designing a new neural network architecture for a task that existing models struggle with?**

**Step 1: Understand the failure mode deeply**

Before designing, answer: *why* do existing architectures fail on this task? Options:
- Insufficient capacity for the task complexity
- Inductive bias mismatch (e.g., CNNs struggle with long-range dependencies)
- Training objective mismatch (the loss doesn't capture what we care about)
- Data efficiency: model requires too much data to learn this pattern

**Step 2: Identify what inductive bias would help**

Architecture = inductive biases. Transformers bake in: position-aware, all-to-all attention. What bias would help *your task*?
- Graph-structured data → message-passing networks
- Hierarchical structure → hierarchical attention
- Physical simulation → equivariant networks
- Long-range dependencies with O(n) complexity → SSMs (Mamba), sparse attention

**Step 3: Design minimally**

Start with the smallest possible modification to a working baseline. Resist the urge to add many components simultaneously. Each component must earn its place via ablation.

**Step 4: Validate at small scale first**

Train the new architecture at small scale (< 100M parameters) on a toy version of the task. Verify the training signal is healthy (gradients, loss curves). Only scale up once small-scale results are promising.

**Step 5: Compare fairly**

New architecture vs. scaled-up baseline with the same compute budget. Many "architectural improvements" in the literature disappear when the baseline is given equivalent compute.

---

### Q6: Interdisciplinary Applications

> **How would you apply AI/ML to a domain you're unfamiliar with — say, climate science?**

Interdisciplinary applications require domain expertise you don't have yet. Approach:

**1. Partner before building**: collaborate with domain experts from day one. They know what questions matter, what data exists, what current methods are, and what "good" looks like. Building in isolation produces tools domain experts won't use.

**2. Understand the data generating process**: what are the physical/biological/social mechanisms? What are the constraints (e.g., physics must be conserved)? These domain constraints can be incorporated as architectural priors or loss terms.

**3. Identify where ML adds unique value**: ML is not always the right tool. It's most valuable where:
   - Patterns are too complex for manual feature engineering
   - Approximation of expensive simulations (emulators)
   - Discovery of patterns in high-dimensional data (e.g., climate teleconnections)

**4. Evaluation requires domain validation**: standard ML metrics (MSE, AUC) are necessary but not sufficient. Domain experts must evaluate whether predictions are physically plausible and useful for their questions.

**5. The "Last Mile" problem**: even correct ML results are useless if they can't be used by domain practitioners. Build for interoperability with domain workflows and tools.

---

### Q7: Robustness to Domain Shift

> **You've trained a model on domain A, and it performs well. When deployed on domain B (different distribution), it fails. How do you address this systematically?**

Domain shift is one of the most common failure modes in deployed ML systems.

**Diagnosis first**:
1. Quantify the distribution shift — feature-level vs. label-level vs. both?
2. Is shift covariate (same labels, different features), label (different label distribution), or concept drift (feature-label relationship changed)?
3. Which features or patterns are most affected?

**Adaptation approaches**:

**1. Domain-invariant representations**: train the model to learn features that are predictive across both domains. Adversarial domain adaptation: add a domain classifier and train the encoder to fool it.

**2. Domain adaptation via fine-tuning**: if you have some labeled data from domain B, fine-tune on it with a low learning rate. Even 100 labeled examples can significantly improve transfer.

**3. Test-time adaptation (TTA)**: update batch normalization statistics or a small subset of parameters at test time using unlabeled target domain data.

**4. Data augmentation for robustness**: augment training data with transformations that simulate the domain B distribution (if you can characterize the shift).

**5. Ensemble with domain-specific models**: train a domain A model + domain B model; mix predictions based on domain confidence.

**Prevention**: before deployment to domain B, run a domain shift analysis. If shift exceeds threshold, require labeled target domain data before deployment.

---

### Q8: Designing Negative Result Studies

> **How do you design a study specifically to investigate a *negative* result you suspect — e.g., "scaling doesn't help for reasoning"?**

Negative result studies require extra rigor because reviewers and the community are skeptical.

**Design principles**:

**1. Precise hypothesis statement**: "Scaling model size from 7B to 70B does not improve symbolic reasoning as measured by GSM8K and MATH benchmarks when controlling for training data quality and quantity."
Vague hypotheses produce vague negative results that convince no one.

**2. Statistical power analysis**: before running experiments, calculate how many examples are needed to detect a meaningful effect size (e.g., 5% improvement) with 80% power. Running underpowered experiments wastes compute and produces inconclusive results.

**3. Eliminate confounders**: if you claim "scaling doesn't help," you must rule out that:
- You were compute-suboptimal (not enough training data for larger model)
- Your evaluation is the bottleneck (not the model)
- You're using a suboptimal training objective for the task
- Your hyperparameters were tuned for the baseline, not the new scale

**4. Multiple evaluation settings**: show the negative result holds across different datasets, prompting strategies, and evaluation methodologies. A negative result on one dataset can be dismissed; on five datasets it cannot.

**5. Publish the null**: even if scale doesn't help for reasoning in your setting, characterizing *when* it does and doesn't help is a genuine scientific contribution.

---

## Preparation Checklist

For AI Researcher interviews, be ready to discuss:

- [ ] Experimental design and statistical rigor
- [ ] Ablation studies and fair baselines
- [ ] Scaling laws (Kaplan, Chinchilla, compute-optimal training)
- [ ] Handling null results and negative results
- [ ] Modern training techniques (RLHF, DPO, GRPO)
- [ ] Attention mechanisms and architecture variants
- [ ] Evaluation methodology for generative models
- [ ] Research communication (paper writing, reproducibility)

---

[← AI Architect](ai-architect.md) | [Data Scientist →](data-scientist.md)
