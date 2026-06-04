# AI Engineer Interview Question

## Topic: Inference Latency Optimization

---

### Question

> Your AI feature is too slow—users are waiting 3-4 seconds for responses. The product team wants it under 500ms. How do you approach this?

---

### Answer

First, I'd **profile the entire request path** to find out where time is actually being spent. Slow AI features usually aren't slow because of one thing—it's death by a thousand cuts.

#### Step 1: Understand the Breakdown

A typical 3-4 second response might look like:

| Component | Time |
|-----------|------|
| Network round-trip | 100ms |
| Preprocessing & embedding | 200ms |
| Vector search / retrieval | 300ms |
| LLM inference | 2500ms |
| Post-processing | 100ms |
| **Total** | **3200ms** |

Once you know the breakdown, you know where to focus.

---

#### Step 2: Attack the Biggest Bottleneck

**If LLM inference is the bottleneck (most common):**

1. **Use a smaller model** – Can you use a 7B model instead of 70B? For many tasks, smaller fine-tuned models outperform larger general ones.

2. **Streaming responses** – Start showing tokens immediately. Perceived latency drops dramatically even if total time is the same.

3. **Speculative decoding** – Use a small draft model to propose tokens, larger model to verify. Can give 2-3x speedups.

4. **Quantization** – INT8 or INT4 quantization can halve inference time with minimal quality loss.

5. **Batching** – If you have multiple concurrent requests, batch them together for GPU efficiency.

6. **Caching** – Cache responses for identical or semantically similar queries.

---

**If retrieval is the bottleneck:**

1. **Optimize your vector index** – Use approximate nearest neighbor (ANN) algorithms like HNSW instead of exact search.

2. **Reduce embedding dimensions** – 384-dim embeddings search faster than 1536-dim.

3. **Pre-filter before vector search** – Use metadata filters to narrow the search space.

4. **Cache hot queries** – Popular questions can be cached at the retrieval layer.

---

#### Step 3: Architectural Changes

If incremental optimizations aren't enough:

- **Two-stage approach**: Fast, cheap model for simple queries; route complex ones to the full pipeline
- **Precomputation**: If queries are predictable, precompute answers during off-peak hours
- **Edge deployment**: Run smaller models closer to users for latency-sensitive features

---

### The Conversation I'd Have with Product

"We can hit 500ms, but there are trade-offs. Here are three options:

1. **500ms, slight quality drop** – Smaller model, aggressive caching
2. **800ms, same quality** – Optimized current pipeline
3. **200ms perceived, 2s actual** – Streaming with progressive display

Which matters more for this feature—absolute speed or perceived responsiveness?"

---

### What This Question Tests

- Systematic debugging approach (profile first, optimize second)
- Knowledge of the inference stack (models, retrieval, infrastructure)
- Ability to propose trade-offs rather than just technical solutions
- Communication with non-technical stakeholders
