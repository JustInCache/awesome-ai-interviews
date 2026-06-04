# LLM Fundamentals

[← Back to README](../README.md) | [Prompt Engineering →](02-prompt-engineering.md)

60+ questions covering LLM architecture, training, inference, and advanced topics.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Architecture & Core Concepts](#architecture--core-concepts)
- [Tokenization & Embeddings](#tokenization--embeddings)
- [Attention Mechanisms](#attention-mechanisms)
- [Training & Alignment](#training--alignment)
- [Inference & Decoding](#inference--decoding)
- [Advanced Architectures](#advanced-architectures)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Architecture & Core Concepts

**Q: What is a Large Language Model (LLM) and how does it work?** `[B]`

An LLM is a neural network — primarily a Transformer decoder — trained on massive text corpora to predict the next token. Key mechanisms:

1. **Tokenization** — text split into subword tokens via BPE/WordPiece
2. **Embeddings** — tokens mapped to dense vectors via a lookup table
3. **Self-Attention** — each token attends to all previous tokens to build contextual representations
4. **Feed-Forward layers** — non-linear transformations applied token-wise
5. **Autoregressive generation** — outputs one token at a time, conditioning on all previous tokens

Training uses **next-token prediction** (cross-entropy loss) over trillions of tokens, followed by SFT (instruction tuning) and RLHF/DPO for alignment.

---

**Q: Explain the Transformer architecture.** `[B]`

The Transformer (Vaswani et al., 2017) processes sequences in parallel using attention rather than recurrence. Modern LLMs use a **decoder-only** variant:

- **Token + Positional Embeddings** — encode meaning and position (modern: RoPE instead of learned/sinusoidal)
- **Multi-Head Self-Attention** — tokens attend to each other; multiple heads capture different relationships
- **Feed-Forward Network** — two linear layers with activation (modern standard: SwiGLU)
- **Residual connections + Normalization** — stabilize training (modern: Pre-RMSNorm)
- **Causal masking** — prevents attending to future tokens during generation

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

**Encoder-only** (BERT): bidirectional attention, used for classification/NER.  
**Encoder-decoder** (T5): encoder processes input, decoder generates output — for translation/summarization.  
**Decoder-only** (GPT, LLaMA, Claude): dominant for generative AI.

---

**Q: What are the key components of the Transformer architecture?** `[B]`

| Component | Purpose |
|-----------|---------|
| Token embeddings | Convert token IDs to dense vectors |
| Positional encoding (RoPE) | Inject sequence order information |
| Multi-head self-attention | Capture relationships between all token pairs |
| Feed-forward network (SwiGLU) | Position-wise non-linear transformation |
| Residual connections | Enable gradient flow through deep networks |
| Layer normalization (RMSNorm) | Stabilize activations, prevent gradient issues |
| Output projection + softmax | Map final hidden states to vocabulary probabilities |

---

**Q: What is the difference between encoder-only, decoder-only, and encoder-decoder architectures?** `[B]`

| Architecture | Attention | Training Objective | Examples | Best For |
|---|---|---|---|---|
| Encoder-only | Bidirectional | Masked LM | BERT, RoBERTa | Classification, NER, embeddings |
| Decoder-only | Causal (unidirectional) | Next-token prediction | GPT, LLaMA, Claude | Text generation, chat |
| Encoder-decoder | Cross-attention | Seq2seq | T5, BART | Translation, summarization |

---

**Q: What are Feed-Forward Networks in LLMs?** `[I]`

Each Transformer layer has two sublayers: attention and FFN. The FFN applies the same transformation to each token position independently (hence "position-wise"):

```
FFN(x) = activation(x·W₁ + b₁)·W₂ + b₂
```

Modern LLMs use **SwiGLU** activation instead of ReLU/GELU:
$$\text{SwiGLU}(x) = \text{Swish}(xW_1) \odot (xW_2)$$

The FFN has ~4× more parameters than the attention layer and is thought to act as a "key-value memory" — storing factual associations learned during pre-training.

---

**Q: What are residual (skip) connections and why are they critical?** `[B]`

Residual connections add the input of a sublayer to its output: `output = x + sublayer(x)`. This allows gradients to flow directly through the network during backpropagation without passing through the sublayer, solving the vanishing gradient problem in deep networks.

Without residual connections, 96-layer models would be untrainable. With them, the gradient can always flow along the "highway" of direct connections.

---

**Q: What is RMSNorm and how does it differ from LayerNorm?** `[A]`

**LayerNorm** normalizes by subtracting mean and dividing by std dev, then applies learned scale (γ) and shift (β) parameters.

**RMSNorm** (Root Mean Square Normalization) drops the mean subtraction:
$$\text{RMSNorm}(x) = \frac{x}{\text{RMS}(x)} \cdot \gamma \quad \text{where} \quad \text{RMS}(x) = \sqrt{\frac{1}{n}\sum x_i^2}$$

This makes RMSNorm ~15% faster while achieving comparable training stability. Used in LLaMA, Mistral, Qwen, and most modern open LLMs.

---

**Q: What are autoregressive models? Compare autoregressive and masked language modeling.** `[I]`

**Autoregressive (Causal) LM**: predict the next token given all previous tokens. Loss computed over the entire sequence at once (efficient). Used in GPT, LLaMA.

**Masked Language Modeling (MLM)**: randomly mask 15% of tokens; predict the masked tokens using bidirectional context. Used in BERT, RoBERTa.

| | Autoregressive | Masked LM |
|---|---|---|
| Context | Left-to-right only | Bidirectional |
| Good for | Generation | Understanding/encoding |
| Training signal | All tokens | Only masked tokens |

---

**Q: How do Transformers address the limitations of CNNs and RNNs?** `[B]`

**RNNs**: process sequentially — gradient vanishes over long sequences; can't parallelize.  
**CNNs**: capture local patterns only; need many layers for long-range dependencies.

**Transformers** solve both: self-attention gives every token direct access to every other token in O(1) steps (vs O(n) for RNN), and the entire sequence is processed in parallel (vs sequential RNN).

Trade-off: Transformers have O(n²) attention complexity — RNNs are O(n). But modern hardware and Flash Attention make this manageable in practice.

---

## Tokenization & Embeddings

**Q: What is tokenization? Explain BPE.** `[B]`

Tokenization converts raw text into token IDs. **Byte Pair Encoding (BPE)**:

1. Initialize vocabulary with individual characters (or bytes)
2. Count all adjacent symbol pairs in the corpus
3. Merge the most frequent pair into a new symbol
4. Repeat until vocabulary reaches target size (e.g. 50K)

Result: common words become single tokens; rare/novel words split into subword pieces (`"tokenization"` → `["token", "ization"]`).

**WordPiece** (BERT): similar to BPE but uses likelihood maximization for merges.  
**SentencePiece** (T5, LLaMA): works directly on raw bytes, language-agnostic.

---

**Q: Why is subword tokenization preferred over word-level tokenization?** `[B]`

**Word-level problems**: huge vocabulary (millions of words); unknown words become `[UNK]`; can't handle morphological variants.

**Subword benefits**:
- Handles out-of-vocabulary words by decomposing them into known subwords
- Manageable vocabulary size (32K–128K typically)
- Learns morphology naturally (prefixes, suffixes)
- Works across all languages with the same tokenizer

---

**Q: What are the trade-offs in using a large vocabulary size?** `[I]`

| Aspect | Larger Vocabulary | Smaller Vocabulary |
|--------|-------------------|-------------------|
| Token fragmentation | Less (rare words stay whole) | More (words split into pieces) |
| Sequence length | Shorter | Longer |
| Memory (embedding layer) | Higher | Lower |
| Softmax compute | Slower | Faster |
| Rare word coverage | Better | Worse |

Typical sweet spot: 32K–128K. LLaMA 3 uses ~128K; GPT-4 uses ~100K.

---

**Q: What are embeddings and what do they capture?** `[B]`

An embedding is a dense, fixed-size vector representation of a discrete token. The **embedding matrix** (shape: vocab_size × d_model) maps each token ID to a vector through a simple lookup.

What embeddings capture through training:
- **Semantic similarity** — synonyms have nearby vectors
- **Analogy relationships** — king - man + woman ≈ queen
- **Syntactic roles** — verbs cluster together
- **Contextual meaning** — in Transformers, the embedding is further refined by each attention layer, so the final hidden state is a **contextualized** embedding

---

**Q: How are tokens converted to embeddings and through the model?** `[B]`

1. Token ID → embedding matrix lookup → token embedding vector (shape: d_model)
2. Add positional encoding (RoPE applied inside attention)
3. Pass through N Transformer blocks: attention + FFN + residual + norm
4. Final hidden state → linear projection to vocab size → softmax → probability over next token

---

## Attention Mechanisms

**Q: What is self-attention and how is it computed step by step?** `[B]`

1. Project each token embedding into Q, K, V vectors using learned weight matrices (Wq, Wk, Wv)
2. Compute attention scores: `scores = Q·Kᵀ / √d_k` (dot product of queries with all keys, scaled)
3. Apply causal mask (decoder): set future positions to -∞ before softmax
4. Softmax over scores → attention weights (non-negative, sum to 1 per row)
5. Output = weighted sum of values: `output = attention_weights · V`

The query asks "what am I looking for?" Keys say "what do I contain?" Values say "what do I give you?"

---

**Q: Why is attention scaled by √dₖ?** `[I]`

With large d_k, dot products grow large in magnitude, pushing softmax into saturation regions where gradients become vanishingly small. Dividing by √d_k keeps the variance of the dot product at ~1 regardless of d_k, maintaining healthy gradient flow.

---

**Q: What is multi-head attention? Why use multiple heads?** `[B]`

Multi-head attention runs h parallel attention operations (heads) with different learned projections, then concatenates and projects the results:

```
MultiHead(Q,K,V) = Concat(head₁,...,headₕ)·Wᵒ
where headᵢ = Attention(Q·Wᵢᵩ, K·Wᵢᴷ, V·Wᵢᵛ)
```

**Why multiple heads?** Each head can specialize in different relationship types — one head tracks syntactic dependencies, another tracks coreferences, another tracks positional proximity. Single-head attention averages over all relationship types simultaneously.

---

**Q: What is causal masking?** `[B]`

Causal masking ensures the model cannot "see" future tokens during generation — enforcing the autoregressive property. Implemented by adding a mask matrix before softmax: future positions get -∞ (which becomes 0 after softmax), so token i only attends to tokens 0..i.

---

**Q: What is Grouped-Query Attention (GQA)?** `[I]`

GQA reduces KV cache memory by sharing K and V heads across groups of Q heads:

```
MHA: 32 Q heads, 32 K heads, 32 V heads  → large KV cache
MQA: 32 Q heads, 1 K head, 1 V head      → tiny KV cache, quality drop
GQA: 32 Q heads, 8 K heads, 8 V heads   → moderate cache, near-MHA quality
```

Most modern open models (LLaMA 2/3, Mistral, Qwen) use GQA. The tradeoff is a moderate reduction in model quality for a large reduction in inference memory.

---

**Q: What is Rotary Position Embedding (RoPE)?** `[I]`

RoPE encodes positional information by rotating the Q and K vectors by an angle proportional to position:

- Position i: rotate Q by θᵢ; position j: rotate K by θⱼ
- The dot product QᵢKⱼ naturally encodes the relative position (i-j), not absolute positions
- Generalizes to longer sequences than trained on (with YaRN/RoPE scaling)

Used in LLaMA, Mistral, Falcon, Qwen — essentially all modern open-source LLMs. Replaced learned positional embeddings and sinusoidal encodings because of superior length generalization.

---

**Q: What is cross-attention and where is it used?** `[I]`

Cross-attention is attention where Q comes from one sequence and K,V come from another. In encoder-decoder Transformers (T5, BART), decoder layers use cross-attention to attend to encoder hidden states — letting the decoder "look at" the full input sequence at each generation step.

In decoder-only models, there is no cross-attention — all attention is self-attention with causal masking.

---

**Q: What is Flash Attention?** `[I]`

Standard attention materializes the full N×N attention matrix in GPU HBM, creating a memory and compute bottleneck. Flash Attention (Dao et al., 2022) is an IO-aware exact attention implementation:

1. Tiles computation into blocks fitting in fast SRAM
2. Uses online softmax to compute results in one pass
3. Never materializes the full attention matrix

Result: **2–4× speedup**, **~10× memory reduction**, **identical mathematical output** (no approximation). Now standard in all production LLMs. Flash Attention 3 adds additional optimizations for H100 GPUs.

---

## Training & Alignment

**Q: What is the LLM training pipeline?** `[I]`

1. **Pre-training** — next-token prediction on trillions of web/book/code tokens. Builds general knowledge and language understanding.
2. **SFT (Supervised Fine-Tuning)** — fine-tune on curated instruction-following examples. Teaches the model to follow instructions.
3. **Reward Modeling** — train a reward model on human preference pairs (A vs B, which is better?)
4. **RLHF/DPO** — optimize model outputs toward higher reward. Creates the helpful, harmless, honest behavior of deployed models.

---

**Q: What is RLHF?** `[I]`

**Reinforcement Learning from Human Feedback** — the core alignment technique:

1. Collect human preference data: show annotators two model responses, they pick the better one
2. Train a reward model on these comparisons
3. Use PPO (Proximal Policy Optimization) to update the LLM toward responses with higher reward
4. Add KL penalty to prevent the model from drifting too far from the original SFT model

Used by OpenAI for ChatGPT, Anthropic for Claude. DPO (Direct Preference Optimization) achieves similar results without the RL loop.

---

**Q: What is cross-entropy loss and how is it used in LLM training?** `[B]`

Cross-entropy loss measures the difference between the model's predicted probability distribution over tokens and the true next token:

$$\mathcal{L} = -\sum_{t} \log P(\text{token}_t | \text{token}_{1..t-1})$$

For a correct next token, the model should assign probability close to 1. Cross-entropy penalizes low probability for the correct token and rewards high probability. Averaged over all positions in all training sequences, minimizing this loss makes the model a better predictor of natural language.

---

**Q: What are Large Reasoning Models (LRMs)?** `[A]`

LRMs (e.g., OpenAI o1, DeepSeek R1) are models trained to produce extended reasoning traces before final answers, using techniques like:

- **Process reward models** — reward is given for correct reasoning steps, not just final answer
- **GRPO / RLHF on reasoning traces** — train on verified step-by-step solutions
- **Monte Carlo Tree Search** — sample multiple reasoning paths, select the best

Result: dramatically better performance on math, coding, and logic — at the cost of much higher latency (thinking before answering). The "thinking" tokens are visible as `<think>` blocks in some models.

---

## Inference & Decoding

**Q: What are the steps in LLM inference?** `[B]`

1. **Tokenization** — input text → token IDs
2. **Prefill** — process all input tokens in parallel, compute K,V for all; builds KV cache
3. **Decode** — generate tokens one at a time, each using cached K,V from previous tokens
4. **Sampling** — at each step, apply temperature/top-p to logits → sample next token
5. **Detokenization** — token IDs → output text
6. **Termination** — stop when `[EOS]` token generated or max length reached

---

**Q: What is KV Cache and how does it speed up inference?** `[B]`

At each attention layer, the model computes K and V matrices from all previous tokens. Without caching, these are recomputed from scratch at every generation step — O(N²) total computation for N tokens.

KV Cache stores these K and V tensors. Each new step only computes the new token's Q, K, V, appends to cache, and reuses all previous K,V. Result: O(N) total computation instead of O(N²).

**Memory growth**: KV cache size = `2 × layers × heads × d_k × sequence_length × batch_size × bytes_per_element`. For large batches or long sequences this dominates GPU memory — addressed by GQA, PagedAttention, KV quantization.

---

**Q: What is temperature, top-p, and top-k?** `[B]`

All three control token sampling randomness:

**Temperature (τ)**: `logits_scaled = logits / τ`
- τ → 0: deterministic (greedy)
- τ = 1: standard sampling
- τ > 1: more random/creative

**Top-k**: retain only top k tokens, renormalize, sample. Fixed cutoff.

**Top-p (nucleus)**: retain smallest set of tokens whose cumulative probability ≥ p. Adaptive — naturally narrows when one token dominates.

**Best practice**: use temperature (0.7–1.0) + top-p (0.9–0.95) together. Top-p adapts better than top-k to distribution shape.

---

**Q: Explain greedy decoding vs beam search vs sampling.** `[I]`

| Strategy | How | Best For |
|----------|-----|----------|
| **Greedy** | Always pick highest-probability token | Fastest, deterministic, often repetitive |
| **Beam search** | Keep top-k partial sequences, pick best complete sequence | Machine translation, summarization (exact outputs) |
| **Temperature sampling** | Sample from softmax distribution | Creative generation, chat |
| **Nucleus (top-p)** | Sample from top-p cumulative probability | General purpose — best quality/diversity balance |

Beam search is deterministic and optimizes sequence-level probability but can be generic/repetitive. Sampling is stochastic but produces more diverse outputs.

---

**Q: What is speculative decoding?** `[I]`

A small **draft model** generates K candidate tokens quickly. The large **verifier model** evaluates all K tokens in **one parallel forward pass** (much cheaper than K sequential passes) and accepts or rejects each token.

- If accepted: use that token, continue from there
- If rejected: resample from the verifier's distribution at that position

Mathematically identical output distribution to unaccelerated generation. **2–3× speedup** with no quality loss. Requires the draft model to have similar vocabulary as the verifier.

---

**Q: What is autoregressive generation and what are its limitations?** `[B]`

Autoregressive generation produces text one token at a time, each conditioned on all previous tokens. This captures coherent sequential structure but has key limitations:

- **Slow**: inherently sequential — can't parallelize generation (only prefill)
- **Error accumulation**: each token conditions on all previous, so early mistakes propagate
- **No revision**: unlike humans, the model cannot go back and edit

Alternatives like diffusion language models aim to generate all tokens in parallel, but current quality lags autoregressive models for text.

---

**Q: What is Paged Attention?** `[I]`

PagedAttention (vLLM, 2023) manages KV cache memory like an OS virtual memory system:

- Divides KV cache into fixed-size **pages** (blocks)
- Maintains a page table mapping logical sequence positions to physical memory locations
- Allows non-contiguous storage of KV cache
- Enables sharing KV cache across requests with common prefixes

Result: **near-zero memory waste** from fragmentation (traditional KV cache allocation wastes up to 60%), enabling 2–4× higher batch sizes and throughput.

---

## Advanced Architectures

**Q: What is model quantization?** `[I]`

Quantization reduces weight precision:

| Format | Bits | 7B model size | Notes |
|--------|------|---------------|-------|
| FP32 | 32 | ~28 GB | Training only |
| BF16 | 16 | ~14 GB | Standard training |
| FP16 | 16 | ~14 GB | Inference, older GPUs |
| INT8 | 8 | ~7 GB | Production inference |
| INT4 | 4 | ~3.5 GB | Consumer GPU inference |

**Post-training quantization (PTQ)**: quantize after training (GPTQ, AWQ, GGUF). Fast but small quality loss.  
**Quantization-aware training (QAT)**: simulate quantization during training. Better quality at INT4.

BF16 preferred over FP16 for training: larger exponent range prevents overflow on extreme gradient values.

---

**Q: What is Mixture of Experts (MoE)?** `[I]`

MoE replaces the dense FFN with multiple "expert" FFNs plus a learned router:

1. Router computes scores for each expert
2. Select top-K experts (typically K=2 of 8–64)
3. Expert outputs are weighted-summed

**Key benefit**: a 56B MoE model (Mixtral 8×7B) activates only ~13B parameters per token — similar inference cost to a 13B dense model but with much higher effective capacity.

**Challenges**: load balancing (routing too many tokens to same expert wastes others), communication overhead in distributed settings.

---

**Q: What is model distillation?** `[I]`

Distillation trains a small **student** model to mimic a large **teacher** model's behavior:

- **Soft targets**: train student to match teacher's output probability distribution (not just the hard labels)
- **Intermediate layers**: can also match intermediate hidden states
- **Reasoning traces**: distill teacher's chain-of-thought (used for DeepSeek R1 distilled models)

Student models often achieve 80–90% of teacher performance at 5–10× lower inference cost.

---

**Q: What is the difference between Small Language Models (SLMs) and LLMs?** `[B]`

SLMs are typically <10B parameters, designed to run on edge devices, laptops, or CPUs. They sacrifice breadth of capability for efficiency, speed, and deployability.

Key trade-offs:
- **SLMs**: faster, cheaper, on-device, privacy-preserving — but weaker on complex reasoning
- **LLMs**: stronger on open-ended tasks — but require expensive GPU infrastructure

With fine-tuning and distillation, SLMs can match LLMs on narrow, well-defined tasks.

---

## Troubleshooting Scenarios

**Q: Your LLM keeps ignoring your instructions. How do you make it follow structured output formats?** `[I]`

1. **Explicit format instruction** — describe the exact schema in the system prompt with an example
2. **Few-shot examples** — include 2–3 example input/output pairs showing the format
3. **JSON mode / structured outputs** — use provider features (OpenAI's `response_format`, function calling) that constrain output to valid JSON
4. **Output parsing with retry** — if output doesn't match schema, retry with error feedback
5. **Fine-tuning** — last resort: fine-tune on examples of the desired format

---

**Q: Your LLM doesn't admit when it doesn't know the answer. How do you fix it?** `[I]`

1. **System prompt calibration**: explicitly instruct "If you are not certain, say 'I don't know' instead of guessing"
2. **Include confidence instruction**: "Rate your confidence 1–5. If below 3, decline to answer"
3. **RAG with retrieval threshold**: if no context was retrieved above a similarity threshold, route to a fallback response
4. **Fine-tuning with abstention examples**: train on examples where the correct output is "I don't know"
5. **Evaluator chain**: a second LLM call checks whether the answer is grounded in available context

---

**Q: Your Transformer runs out of memory on long documents. How do you scale it?** `[A]`

1. **Flash Attention** — eliminates O(n²) memory materialization; enables 4–8× longer sequences
2. **Sliding window attention** — each token only attends to a local window (Mistral's approach); reduce to O(n·w)
3. **Chunked processing with summarization** — process document in chunks; maintain rolling summary
4. **Retrieval over embedding** — don't put everything in context; retrieve the relevant parts
5. **RoPE scaling** — extend context window of an existing model beyond training length using YaRN or linear scaling

---

**Q: After RLHF alignment, your LLM became safer but lost capability on hard tasks. How do you manage the alignment tax?** `[A]`

This is the classic **capability-alignment trade-off**:

1. **Constitutional AI approach** — use self-critique + revision instead of heavy RLHF; lower alignment tax
2. **DPO with capability-preserving pairs** — ensure preference data includes difficult tasks, not just safety
3. **Task-specific fine-tuning post-alignment** — recover capability for specific domains without disturbing safety
4. **Evaluate on capability benchmarks** — track alignment tax quantitatively during training; stop or reduce RLHF intensity if capability drops too much
5. **Separate system prompts** — use stricter instructions for consumer-facing deployment, less restricted for developer API

---

**Q: Your chatbot loses context after 10 turns. How do you maintain long conversation context?** `[I]`

Three main strategies, often combined:

1. **Sliding window** — keep last N turns in context. Simple; loses old information.
2. **Summarization memory** — periodically compress old turns into a summary. Lossy but compact.
3. **Vector memory (RAG-style)** — embed and store all turns; retrieve relevant ones on demand. Precise recall, adds latency.

**Hybrid approach**: keep recent turns verbatim (short-term), summary of earlier session (medium-term), vector store of important facts mentioned (long-term). Choose strategy based on what users actually need to remember.

---

[← Back to README](../README.md) | [Prompt Engineering →](02-prompt-engineering.md)
