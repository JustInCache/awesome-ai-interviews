# Meta AI Interview Preparation

[← Back to README](../README.md) | [See All Companies](../README.md#-company-interview-styles)

**What Meta AI looks for**: engineering excellence, open-source culture, efficiency obsession, and the ability to ship AI at Facebook/Instagram/WhatsApp scale. Meta AI (FAIR + applied AI) spans fundamental research to billions of users. They value people who can do both.

---

## Interview Format

| Round | Format | Duration |
|-------|--------|----------|
| Technical Phone Screen | ML + coding | 45–60 min |
| Coding (×2) | Algorithms + system design | 45 min each |
| ML Design | ML system design or model design | 60 min |
| ML Depth | Deep technical ML/AI knowledge | 60 min |
| Behavioral (×2) | Meta values (move fast, build things, be open) | 30–45 min each |

Meta's coding interviews are known to be rigorous — graph algorithms, DP, trees/heaps. Don't skip the data structures prep.

---

## What They Emphasize

- **Open-source mindset** — LLaMA is open; they want engineers who believe in open AI
- **Efficiency at scale** — serving billions of users means every model must be lean
- **Research-to-product pipeline** — FAIR research ships in Meta products (Instagram Reels ranking, integrity, AR/VR)
- **Move fast** — Meta culture is iterative; speed matters
- **Data-driven decisions** — A/B testing culture is deeply embedded

---

## Interview Questions

### LLaMA & Open-Source AI

**Q1: Why did Meta open-source LLaMA and what has been the impact?**

Meta's strategic rationale for open-sourcing their frontier models:

**Why they open-sourced**:
- **Ecosystem building**: open-source creates tooling, fine-tunes, and research that Meta benefits from
- **Safety through transparency**: more eyes on the model → more red-teaming and safety research
- **Talent attraction**: researchers want to work on models people can actually use and build on
- **Commoditizing the complement**: if LLMs become open infrastructure, Meta's moat is their data (social graph, engagement) and distribution, not the model weights

**Impact**:
- LLaMA created a massive fine-tuning ecosystem (Alpaca, Vicuna, Mistral, countless others)
- Enabled AI research in institutions that can't afford API costs
- Accelerated the understanding of how to make small models perform like large ones (Chinchilla fine-tuning, distillation)
- Sparked the "open vs. closed" AI debate

**Trade-offs** (Meta acknowledges):
- Some fine-tunes bypass safety training — open weights can be "uncensored"
- Accelerates capabilities for all actors (including adversarial ones)
- Meta doesn't control deployment once weights are released

---

**Q2: What architectural improvements distinguish LLaMA 3 from LLaMA 1?**

Meta's model architecture has evolved significantly:

| Feature | LLaMA 1 | LLaMA 2 | LLaMA 3 |
|---------|---------|---------|---------|
| Attention | MHA | GQA (70B only) | GQA (all sizes) |
| Context length | 2K | 4K | 8K (base), 128K (fine-tuned) |
| Vocab size | 32K | 32K | 128K (better tokenization) |
| Training tokens | 1T | 2T | 15T+ |
| RoPE theta | 10K | 10K | 500K (extended context) |
| SwiGLU FFN | Yes | Yes | Yes |
| Pre-norm (RMSNorm) | Yes | Yes | Yes |
| Safety training | Minimal | RLHF chat variant | Advanced RLHF + safety datasets |

**Key LLaMA 3 improvements**:
- 128K vocabulary (vs 32K) — better tokenization efficiency especially for code and non-English languages
- Larger RoPE theta (500K) — enables context extension more easily
- 15T training tokens — following Chinchilla++, much more data per parameter

📖 [LLM Fundamentals](../topics/01-llm-fundamentals.md)

---

**Q3: How would you fine-tune a LLaMA model for a specific enterprise task?**

End-to-end practical answer they want:

**Step 1: Define the task and success criteria**
- What's the exact input-output format?
- How do you measure success (BLEU, human eval, task-specific metric)?
- What's the baseline (zero-shot LLaMA, GPT-4, current system)?

**Step 2: Data preparation**
- Collect 500–5,000 high-quality (input, output) pairs minimum
- Data quality > data quantity
- Include edge cases and negatives (examples of what NOT to do)
- Format as instruction-following: `{"instruction": "...", "input": "...", "output": "..."}`

**Step 3: Choose fine-tuning method**
- QLoRA (4-bit base + LoRA adapter) for most cases — fits on single A100, fast
- Full fine-tune only if you have 100K+ examples and the task is very different from pre-training

**Step 4: Training**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import get_peft_model, LoraConfig
import torch

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B", 
                                               load_in_4bit=True,
                                               device_map="auto")
peft_config = LoraConfig(r=16, lora_alpha=32, target_modules=["q_proj", "v_proj"],
                          lora_dropout=0.1, task_type="CAUSAL_LM")
model = get_peft_model(model, peft_config)
# Train with standard cross-entropy on output tokens only
```

**Step 5: Evaluate properly**
- Hold-out test set with human evaluation
- Check for catastrophic forgetting on general tasks
- Red-team for new failure modes introduced by fine-tuning

📖 [Fine-Tuning deep dive](../topics/05-fine-tuning.md)

---

### AI at Scale (Meta's Core Challenge)

**Q4: How does Meta serve AI models to billions of users on mobile?**

Meta's scale forces extreme efficiency — Instagram has 2B+ users, many on low-end Android phones:

**On-device inference**:
- **Model compression pipeline**: quantize to INT4 or even INT2 → prune → distill to smaller architecture
- **Hardware-aware optimization**: generate kernels optimized for specific mobile chipsets (Qualcomm, MediaTek)
- **ExecuTorch**: Meta's framework for on-device PyTorch inference — enables running LLaMA variants on iPhone
- **Segment Anything Model (SAM)** ships on-device for photo editing

**Cloud inference for heavy workloads**:
- Recommendation models run on Meta's custom accelerators (MTIA chips)
- LLaMA serving uses vLLM-style continuous batching
- Model parallelism across their GPU/MTIA clusters

**What they want to hear**:
- Understanding of the on-device vs. cloud trade-off (latency, privacy, cost)
- Knowledge that Meta builds hardware (MTIA) because NVIDIA margins are unsustainable at their scale
- Appreciation for how compression techniques (quantization, pruning, distillation) enable on-device AI

---

**Q5: Design Meta's feed ranking system incorporating LLM signals.**

Meta's Instagram/Facebook feed ranking is one of the most consequential ML systems in the world:

**Current architecture (simplified)**:
```
Candidate generation (retrieve ~10K posts from your social graph + interest signals)
    ↓
Coarse ranking (lightweight model scores all 10K, keeps top 500)
    ↓
Fine ranking (heavy model with 100s of features, scores top 500, keeps top 50)
    ↓
Policy re-ranking (safety, diversity, integrity filters, ad injection)
    ↓
User feed
```

**Where LLMs add value**:
1. **Content understanding**: LLMs extract semantic meaning from captions, comments — better than TF-IDF features. Embed caption, encode with LLM, use embedding as ranking feature.
2. **User intent modeling**: encode a user's last 20 interactions with an LLM to get a rich intent representation
3. **Quality scoring**: LLM-as-judge for content quality / interestingness, applied to training data labeling
4. **Safety**: LLM-based toxicity and misinformation detection (more accurate than keyword/pattern matching)

**Key consideration**: LLMs are too slow for the fine ranking step (serving millions of posts/sec). They're used in **batch inference** (pre-compute embeddings and quality scores for all content periodically) + **async online** (update scores when new content is posted).

📖 [AI System Design](../topics/07-ai-system-design.md)

---

**Q6: Explain Meta's approach to model safety with open-source LLaMA models.**

Meta faces a unique challenge: they release weights, so they can't control deployment. Their approach:

**Before release**:
- **Internal red-teaming**: systematic adversarial testing for harmful capabilities
- **Llama Guard**: a safety classifier model (also open-sourced) that can be used to filter inputs/outputs
- **Responsible Use Guide**: documentation on responsible deployment
- **System card**: documented known limitations, risks, intended use cases

**Technical safety measures**:
- SFT training on safety examples (harmlessness)
- RLHF with safety-conscious reward models
- Refusals for high-risk categories (bioweapons, CSAM, specific criminal instructions)

**Post-release**:
- Monitor public fine-tunes and derivatives for safety regressions
- Update safety training based on observed failure modes
- Community reports of misuse are incorporated into next version

**Honest acknowledgment**: once weights are released, Meta has limited control. "Safety by architecture" (baking safety into weights) is their main lever. This is imperfect — fine-tuning can remove safety training.

---

### Research & Systems Engineering

**Q7: What is PyTorch 2.0 and why does torch.compile matter?**

Meta maintains PyTorch — the dominant deep learning framework. `torch.compile` (PT 2.0) is a major new capability:

**The problem**: PyTorch's eager execution mode (default) runs operations one at a time, making optimization opportunities visible only at the individual op level. This leaves significant performance on the table.

**torch.compile**:
- Traces your model and compiles it using TorchInductor (generates optimized CUDA/Triton kernels)
- Applies graph-level optimizations: fusing operations, eliminating redundant memory reads, optimizing for specific hardware
- Typically **1.5–3× training speedup** with zero code changes beyond `model = torch.compile(model)`

```python
import torch

model = MyModel()
model = torch.compile(model)  # one line, 1.5-3x speedup
```

**Why Meta cares**: at the scale they train models, a 2× training speedup is worth hundreds of millions of dollars in GPU time.

**Limitations**: compilation adds overhead (first forward pass is slow), some dynamic control flow is hard to compile, debugging compiled models is harder.

---

**Q8: How would you debug a training run where loss has stagnated at epoch 5?**

Meta FAIR runs massive training jobs — debugging them is a core skill.

**Systematic diagnosis**:

1. **Learning rate**: is it too low (no progress) or schedule issue (decayed too early)? Plot LR vs. time.

2. **Gradient flow**: check gradient norms layer by layer. Vanishing gradients → loss plateau. Look for `grad_norm` in logs.

3. **Data issues**: are you hitting duplicate data? Did data pipeline introduce corruption? Sample and inspect training batches at epoch 5 vs epoch 1.

4. **Gradient accumulation bug**: if using gradient accumulation, ensure you're dividing loss by accumulation steps to prevent gradient explosion.

5. **Model capacity**: is the model large enough for the task? Compare training loss vs. validation loss — if both plateau, it's a capacity issue. If training decreases but validation plateaus, it's overfitting.

6. **Batch norm / layer norm issues**: check activation statistics. Dead neurons, exploding/vanishing norms indicate normalization problems.

7. **Optimizer state reset**: did you accidentally reset optimizer momentum (e.g., when loading checkpoint)?

```python
# Quick diagnostic hooks
for name, param in model.named_parameters():
    if param.grad is not None:
        print(f"{name}: grad_norm={param.grad.norm():.4f}, weight_norm={param.norm():.4f}")
```

---

**Q9: What is the difference between FSDP and DDP for distributed training?**

Meta developed FSDP (Fully Sharded Data Parallel) as the successor to DDP for very large models:

**DDP (Distributed Data Parallel)**:
- Each GPU holds a full copy of the model
- Each GPU processes a different data batch
- After backward pass, gradients are averaged across GPUs (all-reduce)
- Limitation: each GPU must fit the full model in memory — can't train models larger than single-GPU memory

**FSDP (Fully Sharded Data Parallel)**:
- Model parameters, gradients, and optimizer state are **sharded across GPUs**
- During forward/backward, each GPU gathers the full layer it needs, computes, then discards
- Memory per GPU = total model memory / num_GPUs
- Trade-off: more communication (all-gather before each layer) vs. DDP (all-reduce once per backward)

```
DDP: GPU0=[model], GPU1=[model], GPU2=[model] → all-reduce gradients
FSDP: GPU0=[shard0], GPU1=[shard1], GPU2=[shard2] → gather for each layer
```

**When to use**:
- Model fits on single GPU: DDP is simpler and slightly faster
- Model too large for single GPU: FSDP (or tensor parallelism)
- Very large models (70B+): combine FSDP with pipeline parallelism

Meta uses FSDP extensively for LLaMA training. PyTorch's FSDP implementation is theirs.

---

**Q10: Describe Meta's approach to evaluation for LLaMA models.**

Meta evaluates LLaMA across multiple dimensions:

**Capability benchmarks**:
- General knowledge: MMLU (57 academic subjects)
- Reasoning: ARC Challenge, HellaSwag, WinoGrande
- Math: GSM8K, MATH
- Code: HumanEval, MBPP
- Long context: "needle in haystack" tests, multi-document QA

**Safety evaluations**:
- TruthfulQA: does the model assert known falsehoods?
- BBQ (Bias Benchmark for QA): bias across demographic groups
- Red team adversarial prompts: internal red-team, public datasets
- Over-refusal rate: doesn't block legitimate requests

**Human evaluation**:
- Side-by-side comparisons (LLaMA vs. GPT-4, LLaMA vs. Claude) — win rate
- Task-specific human eval for instruction following quality

**Meta's honest acknowledgment** in model cards: no benchmark suite comprehensively captures real-world deployment quality. They explicitly encourage external evaluation and community testing.

---

## Resources for Meta AI Prep

- [LLaMA paper series](https://arxiv.org/abs/2302.13971) — read all three (1, 2, 3)
- [Meta AI Research blog](https://ai.meta.com/research/)
- [PyTorch documentation](https://pytorch.org/docs/) — especially distributed training
- [LeetCode](https://leetcode.com/) — Meta coding bar is high; 100+ medium/hard problems
- Explore Meta's open-source AI GitHub: [github.com/facebookresearch](https://github.com/facebookresearch)
- Practice ML system design for social-scale products (billions of users, real-time signals)

📖 Also study: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [Fine-Tuning](../topics/05-fine-tuning.md) | [AI Infrastructure](../topics/12-ai-infrastructure.md)
