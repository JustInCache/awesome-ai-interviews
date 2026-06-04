# Google DeepMind Interview Preparation

[← Back to README](../README.md) | [See All Companies](../README.md#-company-interview-styles)

**What Google DeepMind looks for**: research depth, mathematical rigor, systems engineering excellence at Google scale, and the ability to work at the intersection of research and product. Post-merger (Google Brain + DeepMind), the org values both principled research and engineering pragmatism.

---

## Interview Format

| Round | Format | Duration |
|-------|--------|----------|
| Technical Screen | ML concepts + coding | 45–60 min |
| Research Discussion | Paper review or your prior work | 60 min |
| Coding (×2) | Algorithms + ML implementation | 45–60 min each |
| ML System Design | Large-scale ML system | 60 min |
| General System Design | Distributed systems (for SWE roles) | 60 min |
| Googleyness | Behavioral, collaboration, ambiguity | 45 min |

Google roles have standardized coding bars — expect LeetCode Medium/Hard. ML roles also expect strong mathematical foundations.

---

## What They Emphasize

- **Mathematical depth** — linear algebra, probability, optimization; can you derive things from scratch?
- **Research awareness** — deep knowledge of seminal and recent papers
- **Efficiency obsession** — Google operates at massive scale; every inefficiency costs millions
- **Multimodal AI** — Gemini is their flagship; expect questions on vision+language
- **Responsible AI** — Google has a formal Responsible AI team; it's part of the culture

---

## Interview Questions

### Technical Foundations

**Q1: Derive backpropagation through a self-attention layer.**

Google expects you to be able to derive ML fundamentals, not just use them.

Self-attention output: $O = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$

Gradient through softmax is the tricky part. For a softmax output $s_i$ w.r.t. logit $z_j$:

$$\frac{\partial s_i}{\partial z_j} = s_i(\delta_{ij} - s_j)$$

The gradient of the loss through the attention output:
- $\frac{\partial L}{\partial V} = A^T \frac{\partial L}{\partial O}$ where $A$ is the attention weight matrix
- $\frac{\partial L}{\partial A} = \frac{\partial L}{\partial O} V^T$
- Then backprop through softmax and scaling to get gradients w.r.t. Q and K

What they're looking for: comfort with matrix calculus, ability to reason about gradient flow, and knowing *why* this matters (gradient flow through softmax can vanish for very peaked distributions — relevant to training stability).

---

**Q2: What is the Chinchilla scaling law and what does it imply for training large models?**

Chinchilla (Hoffmann et al., 2022) showed that prior large models were significantly under-trained — too many parameters, too few tokens.

**The finding**: for a given compute budget C (in FLOPs), the optimal model has:
- Parameters N ≈ $\sqrt{C / 6}$  
- Training tokens D ≈ $20 \times N$ (roughly)

Implication: **tokens should scale ~linearly with parameters**. GPT-3 (175B params, 300B tokens) was trained on far too few tokens. A 70B model trained on 1.4T tokens (LLaMA 2) outperforms a 175B model trained on 300B tokens.

**Practical implication**: for a fixed inference budget (you want a small model that's cheap to serve), train a *smaller model on more tokens*. This is why LLaMA 3 (8B and 70B) beats older 175B models on many benchmarks.

**Caveats**: Chinchilla scaling was computed at a specific training compute range. At much larger scales, the laws may shift. Also, inference cost (not just training cost) may change the optimal point.

---

**Q3: How does Google's TPU architecture differ from NVIDIA GPUs for ML training?**

Google designs TPUs (Tensor Processing Units) specifically for ML workloads:

| Aspect | GPU (A100/H100) | TPU v4/v5 |
|--------|----------------|-----------|
| Design | General-purpose SIMD | Purpose-built for matrix ops |
| Memory | HBM3 (~80GB per card) | HBM2e (~32GB per chip) but larger pods |
| Interconnect | NVLink within node; InfiniBand between | TorusNet — designed for large-scale gradient sync |
| Programming | CUDA / PyTorch | XLA / JAX / TensorFlow (lower level) |
| Ecosystem | Mature, flexible | Requires XLA compilation; can be restrictive |
| Cost | Available via cloud | Google-only |

**Key Google insight**: TPU pods (thousands of chips with tight interconnect) enable **data and model parallelism at scales** not easily achievable with GPU clusters. This is why Gemini training could use hundreds of thousands of chips.

**What they want candidates to know**: that hardware architecture shapes what training algorithms are practical — e.g., the TPU's systolic array design rewards certain matrix operation patterns, which influenced how Google structures their model architectures.

---

**Q4: Explain how Gemini 1.5's 1 million token context works.**

Gemini 1.5 Pro uses **Mixture of Experts (MoE)** architecture and **efficient long-context attention** mechanisms:

**Sparse Attention**: rather than full O(n²) attention across 1M tokens, Gemini uses a combination of:
- Local attention windows (attend densely to recent tokens)
- Cross-chunk attention (sparse connections across chunks)
- Strided global attention (attend to every Kth token globally)

**Positional embeddings**: extended RoPE with careful frequency scaling to generalize to much longer sequences than seen in training.

**Training**: trained on progressively longer sequences with curriculum learning — start short, gradually increase context length. The "needle in a haystack" benchmark tests information recall across the full context.

**Practical limitations** (be honest):
- Performance degrades on information in the middle of very long contexts
- 1M token context is slow and expensive — practical use cases need 10–50K tokens
- "Recall" across 1M tokens ≠ "reasoning" across 1M tokens — the model can find a fact but may not integrate multiple distant facts well

📖 [LLM Fundamentals](../topics/01-llm-fundamentals.md)

---

### Multimodal AI

**Q5: How does vision-language alignment work in models like Gemini?**

Vision-language models need to align representations from two fundamentally different modalities.

**Core approach (Gemini architecture)**:
1. **Visual encoder**: a ViT (Vision Transformer) processes image patches → visual token embeddings
2. **Projection layer**: linear or cross-attention layer maps visual embeddings into the LLM's text embedding space
3. **Interleaved training**: train on massive amounts of image-text pairs so the LLM learns to interpret visual tokens alongside text tokens

**Key challenge**: images are dense information (a 224×224 image has 196 patches at 16×16 patch size), while text is sparse. Handling variable-resolution images efficiently without blowing up the context length required innovations:
- **Tile-based encoding**: resize image to multiple tiles, encode each separately, stitch
- **Dynamic resolution**: use fewer patches for simple images, more for complex ones

**What makes Gemini different from GPT-4V**: native multimodal training (not a separate vision adapter bolted on) — image and text tokens are trained together from scratch, leading to tighter cross-modal integration.

📖 [Multimodal AI](../topics/11-multimodal-ai.md)

---

**Q6: Design a multimodal RAG system that handles both images and text documents.**

A system that can answer questions over a mixed corpus of PDFs, images, and text documents:

**Indexing pipeline**:
```
Documents/Images
    ↓
Multimodal chunking:
  - Text chunks: standard recursive splitting
  - Images: extract as visual chunks; generate text captions using VLM
  - PDF tables: extract as structured data; generate natural language summaries
    ↓
Embedding:
  - Text: text embedding model (text-embedding-3-large, Cohere)
  - Images: CLIP or multimodal embedding (same vector space as text)
  - Combined: store in single vector DB with modality metadata
```

**Query pipeline**:
```
User query (text or image+text)
    ↓
Encode query (text embedding or VLM embedding)
    ↓
Hybrid retrieval across text + image chunks
    ↓
Re-rank: cross-encoder that handles text-text and image-text pairs
    ↓
Assemble context: text chunks + image URLs + captions
    ↓
VLM generation (Gemini/GPT-4V) with full multimodal context
```

**Key challenges**:
- Unifying embedding spaces: CLIP text-image alignment isn't perfect, especially for technical content
- Image retrieval precision: images of code, diagrams, or tables are hard to retrieve by embedding alone
- Context assembly: how do you efficiently pass 5 images + 5 text chunks in a single prompt?

---

### Systems & Scale

**Q7: How does Google's data center infrastructure enable Gemini-scale training?**

Gemini Ultra training required thousands of TPU v4 chips. Key infrastructure:

**Distributed training**:
- **Data parallelism**: copies of the model on different chips, each processes different data batches; gradients synced via all-reduce
- **Model parallelism (tensor parallelism)**: split large weight matrices across chips
- **Pipeline parallelism**: different layers on different chips; micro-batch pipelining to reduce idle time
- **Expert parallelism** (for MoE): different experts on different chips

**Network topology**: TPU pods connected via TorusNet (bidirectional ring topology in multiple dimensions) — enables efficient all-reduce gradient synchronization at scale

**Fault tolerance**: with 1,000+ chips, failure rates mean you'll have hardware failures during training runs. Checkpointing every N steps + automatic recovery from latest checkpoint is essential.

**What Google expects candidates to know**: you should understand these concepts and be able to discuss which bottleneck (compute, memory, communication) dominates at different scales.

---

**Q8: Design Google Search's query understanding system for AI-enhanced search.**

Google's AI-enhanced search (Search Generative Experience / AI Overviews) needs to:
- Understand query intent (informational, navigational, transactional)
- Decide when to show AI summary vs. traditional results
- Generate accurate, cited summaries for informational queries
- Ensure freshness (recent events must use fresh retrieval, not stale parametric knowledge)

**Architecture**:
```
Query
  ↓
Query Classification:
  - Intent: informational / navigational / transactional
  - Complexity: simple fact / multi-faceted / opinion / safety-sensitive
  - Freshness need: static knowledge / recent events / real-time
  ↓
Routing:
  - Simple factual (capital of France) → direct LLM answer
  - Complex informational → RAG with top web results
  - Recent events → RAG with freshness-ranked results
  - Transactional / navigational → traditional search results
  ↓
Generation (when RAG):
  - Retrieve top-K web documents
  - Generate grounded summary with citations
  - Quality filter: hallucination check, factual consistency
  ↓
User
```

**Key design considerations**:
- Citation integrity: every claim must trace to a verifiable source
- Advertiser conflict: AI summaries reduce clicks to advertiser pages — business tension
- Freshness: news stories can be 30 minutes old; parametric knowledge is months stale

---

### Research Discussion

**Q9: What is AlphaFold and why does it matter for AI?**

AlphaFold 2 (DeepMind, 2020) solved protein structure prediction — a 50-year-old biology grand challenge — with deep learning. Why AI people should care:

**Technically**:
- Used transformer attention to model amino acid sequence → 3D structure
- Trained on the Protein Data Bank (~170K structures) but generalized to millions of novel proteins
- Key innovation: **Evoformer** — a transformer that processes both sequence and pairwise distance representations simultaneously

**For AI practitioners**:
- Demonstrated AI solving scientific problems previously considered computationally intractable
- The AlphaFold paradigm (sequence + structure + evolutionary information) is now applied to DNA, RNA, antibodies
- Model architecture innovations (EvoFormer) influenced ideas in multi-modal AI

**DeepMind's broader mission**: using AI to accelerate scientific discovery. AlphaFold is exhibit A. Understanding why they made certain decisions (e.g., using evolutionary sequence data as signal) shows you understand the research culture.

---

**Q10: What are the key differences between pre-training and post-training for a model like Gemini?**

| Phase | Purpose | Data | Method |
|-------|---------|------|--------|
| Pre-training | Learn language, knowledge, capabilities | Massive web corpus + code + multimodal | Next-token prediction |
| SFT (supervised fine-tuning) | Follow instructions, adopt desired format | Curated demonstrations | MLE on demonstrations |
| RLHF / RLAIF | Align with human preferences | Preference comparisons | RL (PPO) or DPO |
| Constitutional filtering | Safety | AI self-critique | Rejection sampling + SFT |
| Task-specific fine-tuning | Domain expertise | Task data | LoRA or full fine-tune |

**What makes Google's post-training different**:
- Scale of human preference data (Google has massive labeling capacity)
- RLAIF at scale (using existing Gemini models to generate preference labels for training next Gemini)
- Instruction tuning data generated synthetically + human-curated
- Red teaming via internal Responsible AI teams + external partners

---

## Resources for Google DeepMind Prep

- [LeetCode](https://leetcode.com/) — practice 50–100 medium/hard problems; Google coding bar is high
- [Gemini technical report](https://arxiv.org/abs/2312.11805)
- [AlphaFold papers](https://www.nature.com/articles/s41586-021-03819-2)
- [Google Research blog](https://research.google/) — understand the breadth of their work
- [Papers with Code - Google papers](https://paperswithcode.com/institution/google)
- Practice deriving ML fundamentals from scratch: attention, backprop, EM algorithm

📖 Also study: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [Multimodal AI](../topics/11-multimodal-ai.md) | [AI Infrastructure](../topics/12-ai-infrastructure.md)
