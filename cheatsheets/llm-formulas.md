# LLM Formulas & Numbers Cheatsheet

[← Back to README](../README.md) | [See All Cheatsheets](../README.md#-cheatsheets)

> The numbers every AI engineer must know cold. Memorize these before your interview.

---

## Core Attention Formula

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

- $Q, K, V \in \mathbb{R}^{n \times d_k}$ — Query, Key, Value matrices
- $\sqrt{d_k}$ — scaling factor to prevent softmax saturation
- Complexity: **O(n² · d)** in time and space

**Multi-head attention:**

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, ..., \text{head}_h)W^O$$

$$\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

---

## Memory Calculations

### Model Weight Memory

$$\text{Memory (GB)} = \frac{\text{Parameters} \times \text{Bytes per param}}{10^9}$$

| Precision | Bytes | 7B Model | 13B Model | 70B Model |
|-----------|-------|----------|-----------|-----------|
| FP32 | 4 | 28 GB | 52 GB | 280 GB |
| BF16/FP16 | 2 | 14 GB | 26 GB | 140 GB |
| INT8 | 1 | 7 GB | 13 GB | 70 GB |
| INT4 | 0.5 | 3.5 GB | 6.5 GB | 35 GB |

### Training Memory (rule of thumb)

$$\text{Training Memory} \approx 16 \times \text{Parameters (in billions)} \text{ GB}$$

(Accounts for weights + gradients + optimizer states in mixed precision)

### KV Cache Memory

$$\text{KV Cache (GB)} = \frac{2 \times L \times H \times d_k \times S \times B \times \text{bytes}}{10^9}$$

- $L$ = number of layers
- $H$ = number of KV heads
- $d_k$ = head dimension
- $S$ = sequence length
- $B$ = batch size

**Quick estimate for LLaMA-3-8B (32 layers, 8 KV heads, d_k=128), BF16:**
- Per token, per sequence: 2 × 32 × 8 × 128 × 2 bytes = **131 KB per token**
- 4K context, batch=1: ~512 MB
- 128K context, batch=1: ~16 GB (nearly as much as the model itself)

---

## Transformer Architecture Numbers (Key Models)

| Model | Params | Layers | d_model | Heads | KV Heads | Context | Vocab |
|-------|--------|--------|---------|-------|----------|---------|-------|
| LLaMA 3 8B | 8B | 32 | 4096 | 32 | 8 | 8K | 128K |
| LLaMA 3 70B | 70B | 80 | 8192 | 64 | 8 | 8K | 128K |
| Mistral 7B | 7B | 32 | 4096 | 32 | 8 | 32K | 32K |
| GPT-2 Small | 117M | 12 | 768 | 12 | 12 | 1024 | 50K |
| GPT-3 175B | 175B | 96 | 12288 | 96 | 96 | 2048 | 50K |

---

## Scaling Laws

### Chinchilla Optimal (Hoffmann et al., 2022)

For compute budget $C$ (FLOPs):

$$N_{\text{opt}} \approx \sqrt{\frac{C}{6}}, \quad D_{\text{opt}} \approx 20N_{\text{opt}}$$

**Rule of thumb**: train on **~20 tokens per parameter** for compute-optimal training.

| Model Size | Compute-optimal tokens |
|-----------|----------------------|
| 1B params | 20B tokens |
| 7B params | 140B tokens |
| 70B params | 1.4T tokens |

**LLaMA 3 insight**: trained on 15T tokens (7B model) — much more than Chinchilla optimal. Trading training compute for better inference quality (fixed model size, more tokens = better model at the same serving cost).

### Loss Scaling (Neural Scaling Laws, Kaplan et al.)

$$L \propto N^{-0.076} \quad \text{(parameters)}$$
$$L \propto D^{-0.095} \quad \text{(data tokens)}$$
$$L \propto C^{-0.050} \quad \text{(compute FLOPs)}$$

---

## LoRA Parameter Counts

$$\text{LoRA params} = 2 \times r \times d \times \text{(number of target weight matrices)}$$

For LLaMA-3-8B, targeting q_proj and v_proj with rank r=16:
- d_model = 4096, 32 layers
- LoRA params = 2 × 16 × 4096 × 2 matrices × 32 layers = **8.4M params**
- vs. full model 8,000M params = **0.1% of parameters**

| Rank | LLaMA 3 8B LoRA Params | % of Model |
|------|----------------------|------------|
| 4 | 2.1M | 0.026% |
| 16 | 8.4M | 0.105% |
| 64 | 33.6M | 0.42% |
| 128 | 67.1M | 0.84% |

---

## Token Cost Estimates (2026 pricing, approximate)

| Model | Input (per M tokens) | Output (per M tokens) |
|-------|---------------------|----------------------|
| GPT-4o | $5 | $15 |
| GPT-4o-mini | $0.15 | $0.60 |
| Claude 3.5 Sonnet | $3 | $15 |
| Claude Haiku | $0.25 | $1.25 |
| Gemini 1.5 Pro | $3.50 | $10.50 |
| Gemini 1.5 Flash | $0.075 | $0.30 |

**Quick cost calculation**: 1000 queries × avg 500 input + 200 output tokens = 700K tokens total.
- GPT-4o: 700K tokens → ~$6 per 1000 queries
- GPT-4o-mini: 700K tokens → ~$0.20 per 1000 queries

---

## Inference Throughput (approximate baselines)

| Setup | Throughput | Notes |
|-------|-----------|-------|
| A100 80GB, vLLM, LLaMA-3-8B BF16 | ~2,000 tokens/sec | Single GPU, continuous batching |
| A100 80GB, vLLM, LLaMA-3-70B BF16 | ~200 tokens/sec | Needs 2× A100 |
| H100 80GB, TRT-LLM, LLaMA-3-8B | ~4,000 tokens/sec | Optimized serving |
| OpenAI API, GPT-4o | ~40–80 tokens/sec | Per-request (not server throughput) |

**TTFT (Time to First Token)** is separate from throughput. TTFT is dominated by prefill compute.

---

## Positional Encoding: RoPE

For position $m$ and dimension pair $(2i, 2i+1)$:

$$\begin{bmatrix} q_{2i}' \\ q_{2i+1}' \end{bmatrix} = \begin{bmatrix} \cos m\theta_i & -\sin m\theta_i \\ \sin m\theta_i & \cos m\theta_i \end{bmatrix} \begin{bmatrix} q_{2i} \\ q_{2i+1} \end{bmatrix}$$

where $\theta_i = 10000^{-2i/d}$ (base theta; LLaMA 3 uses 500,000).

The dot product between position-encoded Q at position $m$ and K at position $n$ depends only on $m - n$ — relative position encoding falls out naturally.

---

## Sampling Parameters Quick Reference

| Parameter | Range | Effect |
|-----------|-------|--------|
| temperature | 0 → ∞ | 0 = deterministic greedy; 1 = standard; >1 = more random |
| top_p | 0 → 1 | Nucleus sampling; 0.9 = common default |
| top_k | 1 → vocab | Truncate to k tokens; 40–50 common |
| frequency_penalty | -2 → 2 | Positive reduces repetition |
| presence_penalty | -2 → 2 | Positive encourages topic diversity |

**Common production defaults**: temperature=0.7, top_p=0.9 for creative; temperature=0.1, top_p=0.95 for factual/structured.

---

## Evaluation Metrics Quick Reference

| Metric | Range | Higher = Better | Used For |
|--------|-------|----------------|---------|
| Perplexity | 1 → ∞ | No | Language modeling quality |
| BLEU | 0 → 1 | Yes | Translation, summarization |
| ROUGE-L | 0 → 1 | Yes | Summarization recall |
| BERTScore | -1 → 1 | Yes | Semantic similarity |
| Faithfulness (RAGAS) | 0 → 1 | Yes | RAG hallucination detection |
| MRR | 0 → 1 | Yes | Retrieval quality |
| NDCG@K | 0 → 1 | Yes | Ranked retrieval quality |
| Human win rate | 0% → 100% | Yes | vs. baseline model |

---

📖 Related: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [RAG Checklist](rag-checklist.md) | [System Design Patterns](system-design-patterns.md)
