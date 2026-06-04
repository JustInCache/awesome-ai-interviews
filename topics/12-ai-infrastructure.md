# AI Infrastructure

[← Multimodal AI](11-multimodal-ai.md) | [← Back to Home](../README.md)

25+ questions on GPU infrastructure, model serving, parallelism, and production optimization for LLMs.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [GPU Fundamentals](#gpu-fundamentals)
- [Model Serving Frameworks](#model-serving-frameworks)
- [Parallelism Strategies](#parallelism-strategies)
- [Memory Management](#memory-management)
- [Inference Optimization](#inference-optimization)
- [Auto-Scaling & Cloud](#auto-scaling--cloud)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## GPU Fundamentals

**Q: What GPU should you use for different LLM workloads?** `[B]`

| GPU | VRAM | Best For | Price/hr (cloud) |
|-----|------|---------|-----------------|
| **A100 40GB** | 40 GB | 7B–30B inference, fine-tuning | ~$3 |
| **A100 80GB** | 80 GB | 70B inference, large fine-tuning | ~$4 |
| **H100 80GB** | 80 GB | Highest throughput inference, training | ~$8 |
| **H200 141GB** | 141 GB | Very large model inference, 405B models | ~$12 |
| **RTX 4090** | 24 GB | Local 7B inference, QLoRA fine-tuning | Own/~$2 |
| **L40S** | 48 GB | Cost-effective inference | ~$3.5 |

**VRAM rule of thumb**: 1B parameters ≈ 2 GB VRAM (FP16). 7B model: ~14 GB; 70B model: ~140 GB; 405B model: multi-node required.

**H100 vs A100**: H100 has ~3× higher throughput for inference (900 GB/s memory bandwidth vs 2 TB/s for H100 SXM5). For production inference at scale, H100 has significantly better economics per token.

---

**Q: What is memory bandwidth and why does it matter for LLM inference?** `[I]`

LLM inference is **memory-bandwidth bound** (not compute-bound) for small batch sizes. Each forward pass requires loading all model weights from VRAM into compute units.

**Why bandwidth matters**:
- 70B model in FP16: 140 GB of weights
- A100 memory bandwidth: 2 TB/s
- Minimum inference time per token: 140 GB / 2 TB/s = **70ms** just from weight loading
- H100 SXM5: 3.35 TB/s → 42ms minimum per token

**Implication**: faster memory bandwidth = lower latency for LLM inference. Compute (FLOPS) doesn't matter much for small batch autoregressive generation — you're limited by how fast you can read the weights.

**At large batch sizes**: compute-bound; FLOPS matter more. KV cache grows, memory becomes constrained differently.

---

## Model Serving Frameworks

**Q: What is vLLM and how does it work?** `[I]`

vLLM (Kwon et al., UC Berkeley, 2023) is a high-throughput LLM serving framework built around PagedAttention.

**Key innovations**:
1. **PagedAttention**: manages KV cache in non-contiguous memory pages (like OS virtual memory). Eliminates memory fragmentation — allows serving more concurrent requests on the same GPU.
2. **Continuous batching** (aka iteration-level scheduling): instead of padding all sequences to the same length, dynamically add new requests to a batch as sequences in the batch complete.
3. **Tensor parallelism**: distribute model weights across multiple GPUs.

**Performance**: 2–24× higher throughput vs HuggingFace Transformers for the same latency SLA.

**Quick start**:
```bash
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Llama-3.1-70B-Instruct \
  --tensor-parallel-size 4 \
  --max-model-len 8192
```

---

**Q: Compare vLLM, TGI, and SGLang.** `[I]`

| Framework | Strengths | Best For |
|-----------|-----------|---------|
| **vLLM** | Widest model support, PagedAttention, mature community | General production serving |
| **TGI (HuggingFace)** | HF-native, easy model selection, good docs | HuggingFace model ecosystem |
| **SGLang** | Fastest for multi-step generation, RadixAttention | Agentic workloads, multi-turn chat |
| **LMDeploy** | Strong quantization, turbomind backend | Memory-constrained inference |
| **llama.cpp** | CPU inference, GGUF format, consumer hardware | Local inference, edge deployment |
| **Ollama** | Developer-friendly local serving | Development, testing |

**SGLang's key innovation**: RadixAttention — cache KV state for shared prompt prefixes across requests. Ideal when many requests share the same system prompt.

---

## Parallelism Strategies

**Q: What is tensor parallelism vs pipeline parallelism?** `[I]`

When a model doesn't fit on one GPU, distribute it across multiple GPUs:

**Tensor Parallelism** (TP): split individual weight matrices across GPUs.
```
Weight matrix W (d×d) split across 4 GPUs:
GPU0: W[:, :d/4]   GPU1: W[:, d/4:d/2]
GPU2: W[:, d/2:3d/4]   GPU3: W[:, 3d/4:]
```
- All GPUs active during each forward pass layer
- Requires high-bandwidth interconnect (NVLink) between GPUs
- Best within a single node (NVLink bandwidth >> PCIe or InfiniBand)

**Pipeline Parallelism** (PP): split model layers across GPUs.
```
GPU0: Layers 1-20   GPU1: Layers 21-40
GPU2: Layers 41-60   GPU3: Layers 61-80
```
- Micro-batch pipelining to keep all GPUs busy
- Works across nodes (lower bandwidth interconnect OK)
- Pipeline bubbles reduce efficiency at small batch sizes

**In practice**: use TP within a node, PP across nodes. 70B model on 2 nodes × 8 A100s = TP=8 + PP=2.

---

**Q: What is data parallelism for training?** `[I]`

Data parallelism (DP): replicate the full model on each GPU; each GPU processes a different batch of training data; gradients are averaged across all replicas.

**Standard DP**: each GPU has a complete copy of model + optimizer states. Requires entire model to fit on one GPU.

**FSDP (Fully Sharded Data Parallel, PyTorch)**: shard model parameters, gradients, and optimizer states across GPUs. Only gather parameters when needed for computation. Much lower memory per GPU.

**DeepSpeed ZeRO**:
- Stage 1: shard optimizer states (4× memory reduction vs DP)
- Stage 2: shard gradients (8× memory reduction)
- Stage 3: shard parameters too (N×GPU memory reduction, N = # GPUs)

**FSDP vs DeepSpeed**: FSDP is PyTorch-native (simpler); DeepSpeed is more mature with more features. Both support 70B+ training on 8 A100 nodes.

---

## Memory Management

**Q: What is KV cache and how do you manage it?** `[I]`

During autoregressive generation, each new token must attend to all previous tokens. The KV (Key-Value) matrices for all previous tokens are cached to avoid recomputation:

```
Without KV cache: compute K, V for all 1000 previous tokens at each step
With KV cache: compute K, V only for the new token; load cached K, V for previous tokens
```

**KV cache memory cost**: `2 × n_layers × n_heads × head_dim × seq_len × batch_size × bytes_per_element`

For LLaMA-3 70B (FP16): ~6 GB for a single 4096-token sequence. At batch_size=32: ~200 GB — exceeds GPU VRAM.

**Management strategies**:
- **PagedAttention** (vLLM): non-contiguous memory pages reduce fragmentation
- **KV cache quantization**: INT8 KV cache reduces memory 2×; minimal quality loss
- **Streaming**: for very long sequences, offload KV cache to CPU RAM (latency penalty)
- **Prefix caching**: reuse KV cache for identical prompt prefixes across requests (SGLang RadixAttention)

---

**Q: What is quantization and which format should you use?** `[I]`

Quantization reduces model size by using fewer bits per parameter:

| Format | Bits | Memory | Quality | Use Case |
|--------|------|--------|---------|---------|
| FP32 | 32 | 4 bytes/param | Perfect | Training (gradients need precision) |
| BF16 | 16 | 2 bytes/param | Near-lossless | Inference/fine-tuning standard |
| FP8 | 8 | 1 byte/param | Minimal loss | H100 native, training |
| INT8 | 8 | 1 byte/param | ~1% quality loss | Inference, widely supported |
| INT4 (GPTQ) | 4 | 0.5 bytes/param | ~2-3% quality loss | Memory-constrained inference |
| 4-bit NF4 (QLoRA) | 4 | 0.5 bytes/param | ~1% loss for fine-tuning | QLoRA fine-tuning |
| GGUF (llama.cpp) | 2-8 | Variable | Configurable | Local/CPU inference |

**Recommended**: BF16 for production inference on modern GPUs; INT8 with AWQ or GPTQ when memory is the constraint; GGUF for local deployment.

---

## Inference Optimization

**Q: What is continuous batching and why does it matter?** `[I]`

**Static batching** (naive): pad all sequences in a batch to the same length; wait for all to finish before starting new requests. GPU utilization drops to near 0% while waiting.

**Continuous batching** (iteration-level scheduling):
- Process one iteration (one token generation step) at a time
- After each iteration, completed sequences are removed from the batch and new requests are inserted
- GPU is always maximally utilized

**Throughput impact**: continuous batching provides 2–10× higher throughput than static batching for typical LLM serving workloads. All modern serving frameworks (vLLM, TGI, SGLang) implement this.

---

**Q: What is speculative decoding?** `[A]`

Speculative decoding uses a small "draft model" to accelerate generation by the large "target model":

1. Small model (3B) generates K draft tokens quickly
2. Large model (70B) verifies all K tokens in a single forward pass (parallelizable)
3. If draft token[i] matches what the large model would have generated: accept
4. If not: reject token[i] and all subsequent; regenerate from token[i] using the large model

**Why it's faster**: the large model does one forward pass to verify K tokens instead of K sequential forward passes. Large models are memory-bandwidth bound; parallel verification is much more efficient than sequential generation.

**Typical speedup**: 2–3× for generation tasks where the small model achieves 70%+ token acceptance rate.

**Use cases**: tasks where output is predictable (coding, constrained generation) benefit most. Creative tasks have lower acceptance rates.

---

**Q: What is Flash Attention?** `[I]`

Flash Attention (Dao et al., 2022) is an I/O-aware exact attention algorithm that reduces the memory bottleneck in standard attention:

**Standard attention memory**: O(n²) in sequence length (must materialize full n×n attention matrix)
**Flash Attention memory**: O(n) — never materializes the full attention matrix in HBM

**Technique**: tile the attention computation into blocks that fit in SRAM (fast on-chip memory). Compute softmax incrementally with online rescaling. Only write the final output to HBM.

**Speedup**: 2–4× faster than PyTorch standard attention; 5–20× memory reduction for long sequences. Now the standard implementation in PyTorch (`torch.nn.functional.scaled_dot_product_attention`).

**Flash Attention 3**: extends Flash Attention 2 with FP8 support and H100-optimized kernels; 1.5–2× over FA2.

---

## Auto-Scaling & Cloud

**Q: How do you auto-scale LLM inference on Kubernetes?** `[I]`

LLM scaling is different from standard web services because:
- GPU pods are expensive and take 2–10 minutes to start (model loading)
- Requests are bursty; cold-start latency is unacceptable for real-time chat

**Architecture**:
```
Ingress / API Gateway
    ↓ Request queue (Redis/Kafka)
Request Router
    ↓ Route to available GPU pod
GPU Pod Pool (min-replicas > 0 to avoid cold starts)
    ↓ Horizontal Pod Autoscaler
HPA triggers on: queue depth, GPU utilization, active request count
```

**Scale-to-zero**: not suitable for real-time; OK for batch/async workloads. Use spot/preemptible instances for batch workloads to reduce cost.

**Knative / KEDA**: custom autoscaler metrics (queue length, token throughput) work better than CPU/memory for LLM scaling signals.

**Cloud options**: AWS Inferentia 2 + SageMaker, Google Cloud TPU v5e + Vertex AI, Azure ND-series (A100 clusters). For OpenAI-compatible API, use vLLM on managed k8s.

---

## Troubleshooting Scenarios

**Q: Your LLM serving pod OOMkills on large batches. How do you fix it?** `[I]`

Out-of-memory during inference typically caused by KV cache growth:

1. **Reduce max_model_len**: set `--max-model-len 4096` (vLLM) to limit max sequence length; shorter sequences = smaller KV cache
2. **Quantize KV cache**: INT8 KV cache (`--kv-cache-dtype int8`) reduces KV cache memory 2×
3. **Reduce max_num_seqs**: limit maximum concurrent sequences in a batch
4. **Model quantization**: INT8 or INT4 quantization reduces model weight memory, leaving more for KV cache
5. **GPU upgrade**: if workload genuinely requires more VRAM, move from A100 40GB to A100 80GB or H100 80GB
6. **Monitor with DCGM**: NVIDIA Data Center GPU Manager provides per-GPU memory usage, temperature, and utilization metrics for capacity planning

---

**Q: GPU utilization is consistently at 30%. How do you identify and fix the bottleneck?** `[I]`

Low GPU utilization usually means the GPU is waiting for:

1. **CPU preprocessing bottleneck**: tokenization, request routing, or pre/post-processing is slow. Profile with `py-spy`; optimize tokenization throughput.
2. **Batch size too small**: not enough concurrent requests to saturate the GPU. Increase dynamic batching window (wait up to 50ms to assemble a larger batch).
3. **I/O bottleneck**: model loading from disk on every request? Ensure model is pre-loaded into VRAM on startup.
4. **Network latency**: API requests are slow to arrive, leaving GPU idle. Reduce network hops; use co-located services.
5. **Static batching**: if not using continuous batching, GPU idles while waiting for all sequences to complete. Switch to vLLM or TGI for continuous batching.

**Tools**: NVML profiling, `nvidia-smi dmon` for real-time metrics, NVIDIA Nsight for kernel-level profiling.

---

[← Multimodal AI](11-multimodal-ai.md) | [← Back to Home](../README.md)
