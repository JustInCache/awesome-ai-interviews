# Fine-Tuning & Model Adaptation

[← AI Agents](04-ai-agents.md) | [Vector Databases →](06-vector-databases.md)

25+ questions on fine-tuning strategies, PEFT methods, alignment training, and dataset preparation.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Fundamentals](#fundamentals)
- [PEFT Methods](#peft-methods)
- [Alignment Training](#alignment-training)
- [Dataset Preparation](#dataset-preparation)
- [Evaluation & Production](#evaluation--production)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Fundamentals

**Q: What is fine-tuning and when should you do it?** `[B]`

Fine-tuning updates a pre-trained model's weights on a task-specific dataset to adapt its behavior for that task.

**When fine-tuning beats prompting/RAG**:
- Desired output **format or style** that prompting can't reliably produce
- **Consistent behavior** across thousands of diverse inputs (prompting is brittle at scale)
- **Domain-specific tone** (legal, medical, customer service voice)
- **Latency constraints** — a fine-tuned 7B model can match prompted GPT-4 for narrow tasks at 10–100× lower cost
- **Privacy** — can't send proprietary data to an external API

**When NOT to fine-tune**:
- To inject new factual knowledge (use RAG — fine-tuning doesn't reliably memorize facts)
- If you have <1,000 high-quality examples (likely to overfit)
- If the task changes frequently (fine-tuned weights are static)

Decision framework: **Prompt engineering first** → **RAG if knowledge needed** → **Fine-tuning if behavior/format can't be reliably prompted**

---

**Q: What is the difference between full fine-tuning and parameter-efficient fine-tuning (PEFT)?** `[B]`

| Method | Parameters Updated | GPU Required | Cost | Quality |
|--------|-------------------|-------------|------|---------|
| **Full fine-tuning** | All ~7B+ | Multiple A100s | Very high | Best possible |
| **LoRA** | ~0.1–1% (low-rank adapters) | Single A100 / RTX 4090 | Low | Near-full for most tasks |
| **QLoRA** | Same as LoRA, base is 4-bit | Single consumer GPU | Very low | Slightly below LoRA |
| **Prefix tuning** | Soft prompt tokens only | Small | Very low | Task-specific |
| **Adapter layers** | Small bottleneck layers | Small | Low | Good for NLP tasks |

PEFT is the practical choice for 99% of production fine-tuning needs.

---

**Q: What is continual pre-training vs fine-tuning?** `[I]`

| | Continual Pre-Training | Supervised Fine-Tuning (SFT) |
|---|---|---|
| **Objective** | Next-token prediction on domain text | Supervised on instruction-response pairs |
| **Data** | Raw domain text (papers, code, docs) | Curated Q&A or instruction-following pairs |
| **Goal** | Inject domain knowledge into weights | Teach behavior / format / task patterns |
| **When to use** | Model lacks domain vocabulary or concepts | Model has knowledge, needs to follow instructions |

Typical workflow: pre-train on domain corpus → SFT → alignment (RLHF/DPO).

---

## PEFT Methods

**Q: What is LoRA and how does it work?** `[B]`

LoRA (Hu et al., 2022) adds small trainable low-rank matrices alongside frozen original weights:

```
Original: W ∈ R^(d×k) — frozen
LoRA: W' = W + BA  where B ∈ R^(d×r), A ∈ R^(r×k), rank r << d

r=16, d=4096: 16×4096 + 16×4096 = 131K params (vs 16.7M original)
```

During inference: merge W' = W + BA (no latency overhead). During training: only optimize A, B.

**Key hyperparameters**:
- **Rank (r)**: 4–64 typical. Higher = more expressive, more parameters. Start with 16.
- **Alpha (α)**: scaling factor, often set to 2×rank. Controls magnitude of LoRA updates.
- **Target modules**: which weight matrices to apply LoRA to. Standard: Q, K, V, output projections.
- **Dropout**: 0.05–0.1 on LoRA layers to prevent overfitting.

---

**Q: What is QLoRA?** `[I]`

QLoRA (Dettmers et al., 2023) extends LoRA by **quantizing the frozen base model to 4-bit NF4 (NormalFloat4)**, while keeping LoRA adapters in 16-bit:

- 7B model in 4-bit: ~4GB VRAM
- QLoRA adapters in 16-bit: ~1GB additional
- Result: fine-tune 7B on 8GB GPU (consumer RTX 3080); fine-tune 70B on single A100 80GB

**Three key innovations**:
1. **4-bit NF4 quantization** — quantization format optimal for normally distributed weights
2. **Double quantization** — quantize the quantization constants themselves
3. **Paged optimizers** — use CPU RAM to handle VRAM overflow during optimizer steps

Quality: within 1–2% of full LoRA fine-tuning for most tasks.

---

**Q: What is Prefix Tuning and Prompt Tuning?** `[I]`

**Prompt Tuning**: prepend trainable "soft" token embeddings to the input. Only these embeddings are updated, not any model weights. Extremely parameter-efficient (~10K–100K params). Quality scales with model size — works well for models >10B.

**Prefix Tuning**: extends prompt tuning by prepending trainable embeddings to each Transformer layer's keys and values, not just the input. More expressive than prompt tuning. Used for encoder-decoder models.

**Compared to LoRA**: LoRA generally achieves better quality at comparable parameter counts and has become the dominant PEFT method. Prefix/prompt tuning is interesting for academic research but rarely used in production.

---

**Q: How do you merge multiple LoRA adapters?** `[I]`

When you have multiple task-specific LoRA adapters, merging options:

1. **Sequential merge**: `W = W_base + α₁BA₁ + α₂BA₂` — sum of all adapter contributions. Works well when tasks are not conflicting.

2. **Task arithmetic**: perform vector arithmetic in weight space — `W_merged = W_base + λ(W_task1 - W_base) + μ(W_task2 - W_base)`. Can combine capabilities or negate unwanted behaviors.

3. **TIES-Merging**: resolves conflicts between adapter weights through magnitude pruning and sign agreement before summing.

4. **DARE**: randomly drops (with high probability) and rescales adapter delta weights before merging — reduces interference.

---

## Alignment Training

**Q: What is RLHF?** `[I]`

**Reinforcement Learning from Human Feedback** — the process that transforms a capable base LLM into a helpful, harmless assistant:

1. **SFT phase**: fine-tune on demonstration data (human-written responses to instructions)
2. **Reward model training**: collect human preference comparisons (A vs B responses); train a reward model to predict human preference
3. **PPO training**: use the reward model as a signal to optimize the policy (LLM) via Proximal Policy Optimization; add KL divergence penalty to prevent excessive drift from SFT model

**Reward hacking**: models can game the reward model (optimize for what the reward model scores well, not what humans actually want). Mitigated by KL penalty, diverse reward models, and regular reward model updates.

---

**Q: What is DPO (Direct Preference Optimization)?** `[I]`

DPO (Rafailov et al., 2023) achieves RLHF-equivalent alignment without the RL loop. It directly trains on preference pairs using a closed-form loss derived from the reward model:

$$\mathcal{L}_{DPO} = -\mathbb{E}[\log \sigma(\beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)})]$$

Where y_w = preferred response, y_l = rejected response, π_ref = reference model.

**Why DPO won**: simpler (no reward model, no PPO), more stable training, competitive quality. Now the dominant alignment method for open-source models. ORPO, SimPO, and other variants further simplify by removing the reference model.

---

**Q: What is GRPO (Group Relative Policy Optimization)?** `[A]`

GRPO (used in DeepSeek R1) extends PPO for reasoning model training:

- Sample a **group** of N responses for the same prompt
- Compute relative rewards within the group (which response was better?)
- Use group-relative advantage estimates instead of a separate value model (removes critic network)
- Apply policy gradient update toward high-reward responses

GRPO is particularly effective for math/code reasoning tasks where there's a verifiable correct answer (outcomes-based reward), enabling the model to improve through self-play without human annotation.

---

**Q: What is catastrophic forgetting and how do you prevent it?** `[I]`

Catastrophic forgetting: fine-tuning on a narrow domain degrades the model's general capabilities — a legal fine-tune becomes worse at math, coding, and general language understanding.

**Prevention strategies**:
- **LoRA** — most effective in practice; frozen base weights retain all general capabilities; only the adapter is task-specific
- **Data mixing** — include 5–20% general-domain data (e.g., from OpenHermes, ShareGPT) in your fine-tuning dataset
- **Elastic Weight Consolidation (EWC)** — regularize updates to parameters important for prior tasks; works but computationally expensive
- **Low learning rate + fewer epochs** — conservative training stays closer to the pre-trained distribution
- **Evaluation** — track general benchmarks (MMLU, HellaSwag) alongside task-specific metrics during training; stop before general performance degrades

---

## Dataset Preparation

**Q: How do you prepare a dataset for fine-tuning?** `[I]`

Dataset quality matters more than quantity for fine-tuning. Key steps:

1. **Define the task precisely** — what is a perfect response? Document it as an evaluation rubric before collecting data.

2. **Data format** — instruction-response pairs (Alpaca format), chat format (ShareGPT), or completion format depending on the base model's expected format.

3. **Quality filtering**:
   - Remove examples where response doesn't follow instructions
   - Filter by response length (too short = low-quality; outliers = formatting issues)
   - Deduplication (exact + near-duplicate removal)
   - Language filter if needed

4. **Size guidance**: 1K examples for style/format tasks; 10K+ for complex behavior; 100K+ for domain-specific language models. Quality > quantity.

5. **Train/validation split** — hold out 5–10% for validation; ensure no leakage between splits.

---

**Q: What is synthetic data generation for fine-tuning?** `[I]`

When real labeled data is scarce, use a strong LLM to generate training data for a weaker model:

1. **Seed examples** — start with 50–200 human-curated examples
2. **Prompt a strong model** (GPT-4) — "Generate 100 more examples similar to these, with diversity in [X]"
3. **Quality filter** — apply heuristics or use another LLM call to score/filter generated examples
4. **Human review** — sample 5–10% of synthetic data for human quality review

**Legal consideration**: check the terms of service of the model generating synthetic data. Some providers (OpenAI) prohibit using API outputs to train competing models.

**Risks**: distribution collapse (synthetic data narrows the distribution); error amplification (if seed examples have biases, synthetic data amplifies them). Always evaluate on real-world held-out data.

---

## Evaluation & Production

**Q: How do you evaluate a fine-tuned model?** `[I]`

**Task-specific metrics** (primary):
- Exact match, BLEU/ROUGE for generation tasks
- Accuracy for classification
- Human evaluation / LLM-as-judge for open-ended quality

**Capability preservation** (secondary — check for catastrophic forgetting):
- MMLU (general knowledge)
- HellaSwag / ARC (reasoning)
- HumanEval (coding) — if coding capability matters

**Comparison baseline**: always compare against the best prompted base model (without fine-tuning). Fine-tuning should show measurable improvement over prompting on the target task.

---

**Q: How do you choose between LoRA and full fine-tuning?** `[I]`

**Use LoRA when** (almost always):
- One GPU available
- Task is specialized but base model has relevant pre-training
- Want to maintain general capabilities (no catastrophic forgetting)
- Need to serve multiple task-specific adapters from one base model

**Use full fine-tuning when**:
- You have many A100s and massive training budget
- Task requires fundamental behavioral change (training a new base model)
- You have 100K+ high-quality training examples
- Maximum possible task performance is required with no constraints

In practice: LoRA or QLoRA is the right choice for 99% of production fine-tuning projects.

---

## Troubleshooting Scenarios

**Q: Your fine-tuned model produces factually wrong outputs due to training data quality issues.** `[I]`

1. **Audit training data** — sample 100 examples; rate factual accuracy manually; compute error rate
2. **Identify systematic errors** — are errors in specific topics, formats, or data sources?
3. **Data cleaning** — remove or correct factually wrong examples; add factual verification step to data pipeline
4. **RAG hybrid** — don't rely on fine-tuned weights for factual recall; add RAG to ground outputs in verified sources
5. **Confidence calibration** — add training examples where the correct answer is "I don't know" for uncertain domains

---

**Q: Your fine-tuned LLM forgot its general capabilities (catastrophic forgetting). How do you fix it?** `[I]`

1. **Switch to LoRA** — if you used full fine-tuning, the primary fix is using LoRA; frozen base weights cannot forget
2. **Add data mixing** — re-run fine-tuning with 10–20% general-domain data mixed in
3. **Reduce training intensity** — fewer epochs, lower learning rate, reduce LoRA rank
4. **Merge carefully** — if merging LoRA into base, use a lower scaling factor (α/r ratio)
5. **Evaluate incrementally** — track both task metric and general benchmarks during training; apply early stopping when general capability starts degrading

---

**Q: Your RLHF preference data has low annotator agreement. How do you ensure quality?** `[A]`

1. **Calibration sessions** — have annotators rate the same examples and discuss disagreements to build shared understanding of the rubric
2. **Clear annotation guidelines** — define criteria specifically (what makes response A better than B for this task type)
3. **Filter high-agreement pairs** — only include pairs where ≥3 of 4 annotators agree
4. **Track inter-annotator agreement (Cohen's κ)** — monitor and alert when it drops below threshold
5. **Stratify by annotator** — train separate reward models per annotator; ensemble for diversity; or detect annotation style and route to calibrated annotator subsets

---

## Advanced Topics

**Q: What are ORPO and SimPO and how do they differ from DPO?** `[A]`

Both are improvements over DPO that simplify the training setup:

**DPO** requires a separate reference model (the SFT checkpoint) for KL divergence computation — doubles GPU memory.

**ORPO (Odds Ratio Policy Optimization)**: eliminates the reference model by combining the SFT loss and preference optimization in a single loss term. The odds ratio naturally penalizes rejected responses relative to chosen ones using the current model as an implicit reference:

$$\mathcal{L}_{ORPO} = -\log P(y_w|x) - \lambda \log \sigma(\log \frac{\text{odds}(y_w|x)}{\text{odds}(y_l|x)})$$

**SimPO (Simple Preference Optimization)**: also reference-free; uses length-normalized reward instead of the KL-constrained reward used in DPO. Adds a target reward margin to ensure winning responses clearly outperform losing ones. Empirically competitive with DPO at lower memory cost.

**When to use**:
- Memory-constrained: ORPO or SimPO (no reference model needed)
- Stability priority: DPO (well-tested, predictable)
- Latest research: SimPO shows strong benchmark performance

---

**Q: How do you serve multiple LoRA adapters efficiently from one base model?** `[I]`

Running separate inference servers for each LoRA adapter is wasteful — 50 task-specific adapters would require 50× the GPU memory. Efficient multi-adapter serving:

**Option 1 — Dynamic adapter loading**: load base model once; swap LoRA adapter weights per request. Simple but adds latency for swapping (~10–50ms). Works when request rate per adapter is low.

**Option 2 — S-LoRA / LoRA-as-a-Service**: batch requests from multiple adapters together. During a forward pass, apply different LoRA weights to different sequences in the same batch:
- Unified paging: store all LoRA adapter weights in a pool (like KV cache paging)
- Scheduled merging: fuse applicable adapters into base weights for the current batch
- Implementations: S-LoRA (vLLM community fork), Punica

**Option 3 — Merged inference for top adapters**: for your highest-traffic adapters, pre-merge the adapter into base weights (`W_merged = W_base + BA`) and serve as a standalone model. No runtime overhead.

**Decision rule**:
- < 10 adapters, moderate traffic: dynamic loading
- 10–100 adapters: S-LoRA or request batching by adapter
- >100 adapters: multi-tier (merged for top 5, dynamic for rest)

---

**Q: What is knowledge distillation and how is it used in LLM fine-tuning?** `[I]`

Distillation trains a smaller **student** model to mimic the outputs of a larger **teacher** model:

**Standard distillation (response distillation)**:
- Run teacher model on training inputs → collect outputs (or full probability distributions)
- Train student model to reproduce teacher outputs (KL divergence or cross-entropy on soft labels)
- Student benefits from teacher's "soft" probability distribution which contains more information than hard labels

**Sequence-level KD for LLMs**:
```
Teacher (GPT-4): given "Explain RAG" → generates high-quality explanation
Student (7B): trained to reproduce teacher's output on thousands of such examples
```

Used to create models like: Alpaca (LLaMA fine-tuned on GPT-3.5 outputs), WizardLM, Orca (reasoning steps from GPT-4).

**Legal caveat**: distillation from commercial models (OpenAI, Anthropic) may violate their terms of service. Check before using.

**Reasoning model distillation**: DeepSeek-R1 released distilled variants (8B, 14B, 32B) trained on R1's reasoning traces. The small models inherit long chain-of-thought reasoning capability from the 671B teacher — highly effective.

---

**Q: What is the fine-tuning data flywheel and how do you implement it?** `[A]`

The data flywheel: use your deployed model's outputs to generate training data for the next version, creating a self-improvement loop.

**Implementation**:
1. Deploy fine-tuned v1 model
2. Collect production outputs + user feedback (thumbs up/down, edit corrections)
3. **Positive examples**: outputs with high ratings → add to SFT dataset
4. **Preference pairs**: collect (model output, user-corrected output) → add to DPO dataset
5. **Active learning**: identify queries where the model is uncertain or frequently corrected → prioritize these for human labeling
6. Fine-tune v2 on augmented dataset

**Quality controls**:
- Don't blindly trust user feedback — filter for signal (corrections are signal; binary ratings are noise)
- Audit samples from each feedback category before adding to training data
- Maintain a stable held-out eval set; don't let it get contaminated by production data

**Result**: each model generation improves quality on the specific use cases users care about, without expensive human annotation at scale.

---

**Q: What are the computational requirements for fine-tuning common model sizes?** `[B]`

Quick reference for planning fine-tuning infrastructure:

| Model | Method | GPU | VRAM Needed | Training Time (1K examples) |
|-------|--------|-----|------------|---------------------------|
| 7B | QLoRA (4-bit) | RTX 4090 (24GB) | ~12 GB | ~1 hour |
| 7B | LoRA (16-bit) | A100 40GB | ~25 GB | ~45 min |
| 13B | QLoRA | A100 40GB | ~20 GB | ~2 hours |
| 70B | QLoRA | A100 80GB | ~50 GB | ~10 hours |
| 70B | LoRA | 2× A100 80GB | ~140 GB | ~8 hours |

**Cloud cost reference** (2026 approx):
- A100 40GB on cloud: ~$2–3/hr
- A100 80GB: ~$3–4/hr
- Finetuning 7B with QLoRA for 1K examples: ~$3–5 total

**Tips for reducing cost**:
- Start with QLoRA before full LoRA — quality difference is small for most tasks
- Use gradient checkpointing (2× memory reduction, 30% speed reduction)
- Flash Attention 2 reduces memory significantly for long contexts

---

[← AI Agents](04-ai-agents.md) | [Vector Databases →](06-vector-databases.md)
