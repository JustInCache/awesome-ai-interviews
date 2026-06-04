# ML Engineer Interview Question

## Topic: Distributed Training

---

### Question

> Your model is too large to fit on a single GPU. Walk me through the options for distributed training and how you'd choose between them.

---

### Answer

When a model exceeds single-GPU memory, you have three fundamental strategies—each with different trade-offs. The right choice depends on *what* is too large: the model, the batch, or both.

---

#### Understanding the Memory Breakdown

First, understand what's consuming GPU memory:

| Component | Scales With |
|-----------|-------------|
| Model parameters | Model size |
| Gradients | Model size |
| Optimizer states | Model size (2-8x parameters for Adam) |
| Activations | Batch size × model depth |

For a 7B parameter model in float32:
- Parameters: ~28GB
- Gradients: ~28GB
- Adam optimizer states: ~56GB
- **Total before activations: ~112GB**

That's why large models don't fit on even 80GB GPUs.

---

#### Strategy 1: Data Parallelism

**What it does**: Same model replicated on each GPU, different data batches.

**How it works**:
1. Each GPU has a full copy of the model
2. Split the batch across GPUs
3. Each GPU computes gradients on its portion
4. All-reduce to synchronize gradients
5. Each GPU updates its local copy

**When to use**: Model fits on one GPU, but you want faster training or larger effective batch sizes.

**Trade-offs**:
- ✅ Simple to implement
- ✅ Near-linear scaling for compute
- ❌ Memory usage doesn't decrease per GPU
- ❌ Communication overhead grows with model size

---

#### Strategy 2: Model Parallelism (Tensor Parallelism)

**What it does**: Split individual layers across GPUs.

**How it works**:
- Large matrix multiplications are partitioned
- Each GPU computes part of the result
- Results are combined via all-reduce

Example: A 4096×4096 weight matrix split across 4 GPUs as 4096×1024 each.

**When to use**: Individual layers are too large, especially attention and FFN layers.

**Trade-offs**:
- ✅ Reduces memory per GPU proportionally
- ❌ High communication cost (every layer needs synchronization)
- ❌ Complex to implement correctly
- ❌ Best within a single node (fast interconnect required)

---

#### Strategy 3: Pipeline Parallelism

**What it does**: Split model vertically—different layers on different GPUs.

**How it works**:
1. GPU 0 has layers 1-10, GPU 1 has layers 11-20, etc.
2. Forward pass: activations flow GPU 0 → GPU 1 → ...
3. Backward pass: gradients flow in reverse
4. Micro-batching to keep all GPUs busy

**When to use**: Model depth is the issue, and you can tolerate some pipeline bubbles.

**Trade-offs**:
- ✅ Lower communication than tensor parallelism
- ✅ Can span multiple nodes efficiently
- ❌ Pipeline bubbles reduce efficiency
- ❌ Complex scheduling to minimize idle time

---

#### Strategy 4: Fully Sharded Data Parallelism (FSDP/ZeRO)

**What it does**: Shards optimizer states, gradients, and optionally parameters across GPUs.

**How it works**:
- ZeRO Stage 1: Shard optimizer states only
- ZeRO Stage 2: + shard gradients  
- ZeRO Stage 3: + shard parameters

Parameters are gathered just-in-time for computation, then discarded.

**When to use**: When you want data parallelism benefits but model doesn't fit.

**Trade-offs**:
- ✅ Combines memory efficiency with data parallel simplicity
- ✅ Near-linear memory scaling with GPU count
- ❌ More communication than vanilla data parallel
- ❌ Stage 3 has significant communication overhead

---

#### Decision Framework

```
Is the model too large for one GPU?
├── No → Use standard Data Parallelism
└── Yes → What's the constraint?
    ├── Optimizer states → ZeRO Stage 1-2
    ├── Model parameters → ZeRO Stage 3 or Pipeline Parallelism
    └── Single layer too large → Tensor Parallelism
    
How many nodes?
├── Single node (fast NVLink) → Tensor parallelism works well
└── Multiple nodes → Pipeline + FSDP hybrid
```

---

#### Practical Example

Training a 70B model on 8× A100 80GB GPUs:

**Option A: FSDP (ZeRO-3)**
- Shard everything across all 8 GPUs
- Each GPU holds 1/8 of parameters
- Effective memory: ~14GB per GPU for model state
- Good for: Maximum memory efficiency, simpler code

**Option B: Tensor Parallel (8-way)**
- Each layer split across all 8 GPUs
- Very high communication volume
- Good for: Single node with NVLink, maximum throughput

**Option C: Pipeline Parallel (4 stages) + Data Parallel (2 replicas)**
- Layers 1-20 on GPUs 0,4; Layers 21-40 on GPUs 1,5; etc.
- Two pipeline replicas for larger effective batch
- Good for: Multi-node setups, balancing memory and communication

**My recommendation for most cases**: Start with FSDP. It's the best balance of simplicity and efficiency. Only add tensor/pipeline parallelism if profiling shows communication bottlenecks.

---

### What This Question Tests

- Deep understanding of GPU memory and distributed systems
- Knowledge of trade-offs between parallelism strategies
- Practical decision-making under constraints
- Experience with large-scale training
