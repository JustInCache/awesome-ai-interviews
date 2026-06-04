# ML Engineer Interview Question

## Topic: Model Compression & Optimization

---

### Question

> Your model is too large and slow for production deployment. Walk me through the techniques for compressing and optimizing ML models while maintaining performance.

---

### Answer

Model compression is about finding the sweet spot between model size, inference speed, and accuracy. The goal is often "good enough" performance at a fraction of the compute cost.

---

#### Understanding the Trade-offs

Before compressing, understand what you're optimizing for:

| Target | Primary Technique | Accuracy Impact |
|--------|-------------------|-----------------|
| Size reduction | Quantization, pruning | Low (often <1% drop) |
| Speed improvement | Architecture optimization | Medium (depends on changes) |
| Memory efficiency | Distillation, quantization | Low-Medium |
| Energy efficiency | Quantization, pruning | Low |

Different techniques have different cost-benefit profiles.

---

#### Technique 1: Quantization

**What it does**: Reduces numerical precision of weights and activations.

**Types:**

- **Post-training quantization**: Quantize trained model
  - INT8: 4x smaller, minimal accuracy loss
  - INT4: 8x smaller, ~1% accuracy drop
  - Dynamic range quantization: Per-tensor scaling

- **Quantization-aware training (QAT)**: Train with quantization in mind
  - Better accuracy than post-training
  - Requires retraining

**When to use**: Almost always first step. Easy wins with little accuracy cost.

**Implementation**:

```python
# Post-training INT8 quantization
import tensorflow as tf
converter = tf.lite.TFLiteConverter.from_saved_model(saved_model_dir)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
tflite_model = converter.convert()
```

---

#### Technique 2: Pruning

**What it does**: Removes unnecessary weights/connections.

**Types:**

- **Magnitude pruning**: Remove small weights (|w| < threshold)
- **Structured pruning**: Remove entire neurons/channels
- **Dynamic pruning**: Prune during training

**When to use**: When you can afford some retraining. Often combined with fine-tuning.

**Process**:

1. Train full model
2. Prune weights (sparsity 50-90%)
3. Fine-tune pruned model
4. Repeat for iterative pruning

**Benefits**: Can achieve 2-5x speedup with <1% accuracy loss.

---

#### Technique 3: Knowledge Distillation

**What it does**: Train smaller "student" model to mimic larger "teacher" model.

**How it works**:

- Teacher: Large, accurate model
- Student: Small, fast model
- Training objective: Match teacher's outputs (soft targets) + true labels

**When to use**: When you need significant size reduction but can retrain.

**Variants**:

- **Self-distillation**: Student learns from ensemble of its own checkpoints
- **Cross-modal distillation**: Teacher in one modality, student in another
- **Progressive distillation**: Multi-stage size reduction

---

#### Technique 4: Architecture Optimization

**What it does**: Change model structure for efficiency.

**Techniques:**

- **Depthwise separable convolutions**: MobileNet-style
- **Attention optimization**: Efficient attention (Flash Attention, sparse attention)
- **Dynamic computation**: Conditional computation based on input complexity
- **Neural architecture search (NAS)**: Automated architecture optimization

**When to use**: For custom architectures or when standard compression isn't enough.

---

#### Technique 5: Hardware-Specific Optimization

**What it does**: Leverage target hardware capabilities.

**Examples:**

- **TensorRT**: NVIDIA GPU optimization
- **Core ML**: Apple device optimization
- **Edge TPU**: Google Coral optimization
- **OpenVINO**: Intel hardware optimization

**When to use**: When deploying to specific hardware platforms.

---

#### Compression Pipeline

**Phase 1: Baseline Assessment**

- Measure current model: size, latency, accuracy
- Profile bottlenecks (which layers are slowest?)
- Set targets: "80% size reduction, <2% accuracy drop"

**Phase 2: Apply Techniques**

Start with lowest-risk, highest-reward:

1. **Quantization** (easy, big wins)
2. **Pruning** (moderate effort, good results)
3. **Distillation** (more work, best results)
4. **Architecture changes** (high effort, custom results)

**Phase 3: Validation & Iteration**

- Test compressed model on target hardware
- Measure accuracy on diverse data
- A/B test in production if possible
- Iterate based on results

---

#### Practical Example: Compressing BERT

**Original**: 340M parameters, 1.2GB, 200ms inference

**Compression pipeline**:

1. **Quantization**: INT8 → 300M params, 300MB, 150ms, -0.5% accuracy
2. **Pruning**: 50% sparsity → 150M params, 150MB, 120ms, -1.0% accuracy
3. **Distillation**: DistilBERT → 66M params, 250MB, 80ms, -2.0% accuracy

**Result**: 5x smaller, 2.5x faster, 2% accuracy drop

---

#### Common Challenges & Solutions

**Challenge: Accuracy drops too much**

- **Solution**: Use quantization-aware training, gradual pruning, better distillation

**Challenge: Hardware compatibility**

- **Solution**: Test on target hardware early, use hardware-specific tools

**Challenge: Maintenance overhead**

- **Solution**: Automate compression in CI/CD pipeline, version compressed models

**Challenge: Different requirements per use case**

- **Solution**: Multiple model variants (fast/accurate trade-off)

---

#### When Not to Compress

- **Research/prototyping**: Accuracy > efficiency
- **Offline batch processing**: Speed less critical
- **High-stakes decisions**: Prefer accuracy over efficiency
- **When compression cost > benefit**: Small models don't need compression

---

### What This Question Tests

- Knowledge of model optimization techniques
- Understanding of accuracy-efficiency trade-offs
- Practical experience with compression pipelines
- Hardware-aware development
