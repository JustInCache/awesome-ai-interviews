# Vector Databases & Embeddings

[← Fine-Tuning](05-fine-tuning.md) | [AI System Design →](07-ai-system-design.md)

25+ questions on embeddings, vector search, ANN algorithms, and production vector database engineering.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Embeddings Fundamentals](#embeddings-fundamentals)
- [Similarity Metrics](#similarity-metrics)
- [Vector Databases](#vector-databases)
- [Indexing & Search Algorithms](#indexing--search-algorithms)
- [Production Engineering](#production-engineering)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Embeddings Fundamentals

**Q: What are embeddings in the context of AI engineering?** `[B]`

Embeddings are dense, fixed-size vector representations of data (text, images, audio) where **semantic similarity corresponds to geometric proximity**. Similar inputs produce nearby vectors; dissimilar inputs produce distant vectors.

```
"dog"     → [0.21, -0.84, 0.52, ...]  ↑ similar vectors
"puppy"   → [0.19, -0.79, 0.58, ...]  ↑
"airplane" → [-0.45, 0.32, -0.91, ...]  ← far away
```

Embeddings enable semantic search: find all documents *similar in meaning* to a query, not just documents containing the same keywords.

---

**Q: How do embedding models convert text to vectors?** `[B]`

Modern embedding models are Transformer encoders trained via contrastive learning:

1. Encode input text through Transformer layers → pooled [CLS] token or mean-pooled representation
2. **Contrastive training**: positive pairs (semantically similar texts) pulled together; negative pairs pushed apart in the embedding space
3. Normalized to unit length (L2 normalization) → enables cosine similarity via dot product

Popular models:
| Model | Dimensions | Best For |
|-------|-----------|---------|
| `text-embedding-3-large` | 3072 | General purpose, OpenAI |
| `text-embedding-3-small` | 1536 | Cost-efficient, OpenAI |
| `BAAI/bge-m3` | 1024 | Multilingual, open source |
| `intfloat/e5-large-v2` | 1024 | Open source, strong quality |
| `Cohere embed-v3` | 1024 | Strong retrieval, API |

---

**Q: What is the difference between sparse and dense embeddings?** `[I]`

| | Dense Embeddings | Sparse Embeddings |
|---|---|---|
| **Representation** | Fixed-size dense vector (all dimensions non-zero) | Vocabulary-size vector (mostly zeros) |
| **Examples** | OpenAI ada-002, BGE, E5 | BM25, TF-IDF, SPLADE |
| **Strengths** | Semantic similarity, paraphrase matching | Exact matches, keyword search |
| **Weaknesses** | Misses exact keyword matches | Misses semantic meaning |
| **Dimensionality** | 384–3072 | 30K–100K (vocab size) |

**SPLADE** (Formal et al., 2021) is a learned sparse model that creates interpretable sparse vectors — it combines exact-match efficiency with some semantic capability. Used in hybrid search systems.

---

**Q: What is embedding dimensionality and how does it affect performance?** `[I]`

Higher dimensionality = more expressive, better quality — at the cost of storage and compute:

| Dimensions | Storage per vector | ANN search speed | Quality |
|-----------|-------------------|-----------------|---------|
| 384 | 1.5 KB | Fast | Good for many tasks |
| 768 | 3 KB | Moderate | Strong general quality |
| 1536 | 6 KB | Slower | High quality (text-embedding-3-small) |
| 3072 | 12 KB | Slow | Best quality (text-embedding-3-large) |

For 10M vectors at 1536 dims: 60 GB storage. Many systems use **dimension reduction** (PCA, Matryoshka Representation Learning) to truncate to smaller dimensions without proportional quality loss.

**Matryoshka embeddings** (OpenAI text-embedding-3): trained so that the first N dimensions form a valid embedding of quality proportional to N. You can truncate from 3072 to 256 and still get usable quality.

---

## Similarity Metrics

**Q: Explain cosine similarity, dot product, and Euclidean distance for vector search.** `[I]`

**Cosine similarity**: measures the angle between vectors, independent of magnitude.
$$\text{cosine}(A, B) = \frac{A \cdot B}{|A||B|}$$
Range: [-1, 1]. 1 = identical direction. Best for text where magnitude shouldn't matter.

**Dot product**: `A · B = Σ AᵢBᵢ`. Equivalent to cosine if vectors are L2-normalized (which most embedding models do). Efficient for GPU computation.

**Euclidean distance**: `||A - B||₂ = √Σ(Aᵢ - Bᵢ)²`. Sensitive to magnitude. Used for spatial data; generally not preferred for text embeddings.

**In practice**: use cosine similarity (or dot product on normalized vectors). Vector databases typically default to this.

---

**Q: What is embedding drift and how do you handle it?** `[I]`

Embedding drift occurs when you update the embedding model — the same text now produces a different vector. Vectors in your index computed with the old model are now incompatible with query vectors from the new model.

**Handling embedding model updates**:
1. **Shadow index**: build a new index with the new model in the background; switch over when complete
2. **Dual query**: query both old and new indexes during transition; merge results
3. **Lazy re-embedding**: re-embed documents on next access; maintain model version metadata per document
4. **Incremental re-indexing**: re-embed in batches during low-traffic periods

**Prevention**: pin embedding model version explicitly; treat embedding model changes like schema migrations — planned, tested, with rollback plan.

---

## Vector Databases

**Q: What is a vector database and how does it differ from a traditional database?** `[B]`

A traditional database stores structured data and queries by exact or range matching (SQL). A vector database stores high-dimensional vectors and queries by **approximate nearest neighbor (ANN) similarity**.

| Feature | Traditional DB | Vector DB |
|---------|---------------|----------|
| Primary query | Exact/range match | Semantic similarity |
| Indexing | B-tree, hash index | HNSW, IVF, PQ |
| Query type | `WHERE column = value` | `KNN(query_vector, k=10)` |
| Data type | Structured | High-dimensional float arrays |
| Scale | Billions of rows easily | Millions–billions of vectors |

**Popular choices**:
- **Pinecone**: managed, production-ready, simple API
- **Weaviate**: open source + cloud, multi-modal, GraphQL
- **Qdrant**: open source, Rust-based, high performance
- **Chroma**: simple, local-first, great for development
- **pgvector**: PostgreSQL extension — good for teams already on Postgres
- **Milvus**: open source, cloud-native, enterprise scale

---

**Q: How do you index and query multi-tenant data in a vector database?** `[A]`

Multi-tenant RAG (e.g., SaaS with multiple customers) requires strict isolation:

**Option 1 — Namespace per tenant**: dedicated namespace/collection per tenant. True isolation, easy access control. Trade-off: operational complexity at scale (10K customers = 10K namespaces).

**Option 2 — Shared index with metadata filtering**: one index, filter by `tenant_id` at query time. Simpler operations; requires vector DB with efficient metadata filtering (Pinecone namespaces, Qdrant payload filtering).

**Option 3 — Hybrid**: top-tier customers get dedicated namespaces for isolation and performance guarantees; bulk customers share an index with metadata filtering.

**Access control pattern**:
```python
results = vector_db.query(
    vector=query_embedding,
    filter={"tenant_id": user.tenant_id, "user_id": user.id},
    top_k=10
)
```

---

## Indexing & Search Algorithms

**Q: What is HNSW and why is it the dominant ANN algorithm?** `[A]`

**Hierarchical Navigable Small World** (Malkov et al., 2016) builds a multi-layer graph:

- **Bottom layer**: full graph where each node connects to its M nearest neighbors
- **Upper layers**: progressively sparser subgraphs for fast coarse navigation

Query: enter at top layer → greedy graph traversal to approximate nearest neighbor → descend to lower layers → refine.

**Why HNSW dominates**:
- O(log n) query time — scales to billions of vectors
- High recall (>99%) at relatively low ef_search values
- Incremental insertions (no full rebuild needed)
- Available in Faiss, Qdrant, Weaviate, pgvector

**Trade-offs**: high memory (each node stores neighbor lists); index construction is slow; deletion is expensive.

---

**Q: What is the difference between IVF and HNSW indexing?** `[I]`

| | HNSW | IVF (Inverted File Index) |
|---|---|---|
| **Structure** | Navigable graph | Voronoi cell partitioning |
| **Query speed** | Very fast | Fast (with IVF-PQ, very fast) |
| **Memory** | High | Lower (especially with PQ compression) |
| **Recall** | Very high | Good with enough probes |
| **Insertions** | Incremental | Batch (requires rebuild) |
| **Best for** | General use, real-time insertions | Billions of vectors, memory-constrained |

**IVF-PQ** (Product Quantization): compresses vectors by encoding each sub-vector as an index into a learned codebook — reduces 1536×4 bytes → 96 bytes per vector. Enables billions of vectors in RAM.

---

**Q: How do you handle large-scale vector search with billions of vectors?** `[A]`

At billions of vectors, standard HNSW becomes impractical (TB of RAM). Scale strategies:

1. **IVF-PQ**: Faiss IVF with product quantization — compress vectors 16–32× with ~5% recall loss
2. **Sharding**: partition the index across machines by document ID or category; route queries to relevant shards
3. **Quantized HNSW (ScaNN)**: Google's ScaNN uses asymmetric quantization with HNSW-like traversal for extreme scale
4. **Disk-based indexes**: DiskANN (Microsoft) stores vectors on SSD; uses beam search to minimize disk I/O; 1B vectors on ~$100/month storage
5. **Metadata pre-filtering**: narrow the search space before ANN using metadata filters — effective filter reduces search space by 10–100×, dramatically improving performance

---

## Production Engineering

**Q: How do you choose the right embedding model for your use case?** `[I]`

Evaluation framework:

1. **Benchmark on your data**: run MTEB (Massive Text Embedding Benchmark) scores for your domain, or better — benchmark on your actual query-document pairs
2. **Asymmetric vs symmetric**: query ("what is X?") vs document ("X is a concept that...") may need different encoders (use models with `query_instruction` / `passage_instruction` support)
3. **Multilingual**: if serving non-English content, use multilingual models (BGE-M3, E5-multilingual)
4. **Domain specificity**: general models may underperform for specialized domains (medical, legal, code). Consider domain-specific models or fine-tuned embeddings.
5. **Cost/latency vs quality**: OpenAI ada-002 is convenient and good; BGE-large is competitive and free if self-hosted
6. **Dimensionality vs cost**: smaller dimensions = cheaper storage and faster search at some quality cost

---

**Q: What is quantization of embeddings and how does it reduce costs?** `[I]`

**Scalar quantization**: convert float32 (4 bytes/dim) → int8 (1 byte/dim). 4× compression, ~5% recall loss. Widely supported.

**Binary quantization**: convert to 1 bit/dim by sign(x). 32× compression, ~10–15% recall loss. Can recover quality by re-ranking binary-retrieved candidates with full float precision.

**Product quantization (PQ)**: cluster sub-vectors into codebooks; represent each vector as indices into codebooks. 16–32× compression. Used in Faiss IVF-PQ.

**Matryoshka dimension truncation**: truncate OpenAI text-embedding-3-large from 3072 → 512 dimensions with modest quality loss. Same model, no quantization error.

---

**Q: How do you benchmark and evaluate embedding model quality?** `[I]`

**Offline evaluation** (before deployment):
- **MTEB leaderboard**: standardized benchmarks across retrieval, classification, clustering — check your task category
- **Custom evaluation**: collect 100–500 query-document pairs from your domain; compute recall@1, recall@5, MRR

**Online evaluation** (after deployment):
- **Click-through rate**: do users click on the top retrieved results?
- **Zero-result rate**: what fraction of queries return no relevant results above threshold?
- **Feedback signals**: thumbs up/down, session abandonment after seeing results

A model that ranks #1 on MTEB may underperform a smaller model on your specific domain. Always evaluate on your actual data.

---

## Troubleshooting Scenarios

**Q: Your vector database is consuming too much memory. How do you reduce it?** `[I]`

1. **Scalar quantization (int8)**: 4× memory reduction, ~5% quality loss. Most vector DBs support this natively.
2. **Binary quantization**: 32× reduction; add re-ranking step to recover quality.
3. **Dimension reduction**: truncate from 1536→512 with Matryoshka models; or apply PCA post-hoc.
4. **IVF-PQ indexing**: replace HNSW with IVF-PQ for 16–32× compression at scale.
5. **Tiered storage**: keep hot data in RAM (HNSW), cold data on SSD (DiskANN).
6. **Deduplicate**: embed document fingerprints; don't re-index identical or near-duplicate content.

---

**Q: Your new embedding model has different dimensions from existing vectors. How do you handle migration?** `[I]`

You cannot mix vectors of different dimensions in the same HNSW index.

**Migration plan**:
1. **Build shadow index** with new model — embed all documents in background; validate quality
2. **Test quality** on eval dataset — confirm new model improves or matches old
3. **Dual-query transition** — during cutover period, query both indexes; merge results by score normalization
4. **Atomic switchover** — when shadow index is complete and validated, swap the primary index pointer
5. **Decommission old index** after confidence period

**Avoid**: gradual migration where some documents are in new index and some in old — split indexes cause inconsistent retrieval quality.

---

**Q: Your semantic search fails for short queries (1–3 words). How do you improve it?** `[I]`

Short queries produce poor embeddings — one token embeddings are ambiguous:
- "Python" → programming language? snake? Monty Python?

**Fixes**:
1. **HyDE (Hypothetical Document Embeddings)**: generate a hypothetical full answer to the short query, embed that
2. **Query expansion**: use an LLM to expand the short query into a full sentence: "Python" → "Python programming language tutorial and documentation"
3. **Keyword augmentation**: combine vector search with BM25 — keyword search works well for short, specific queries
4. **Contextual queries**: use conversation context to expand the query (if user asked about programming earlier, "Python" is the language)

---

## Advanced Indexing

**Q: How does HNSW work internally? Walk through the construction and query algorithms.** `[A]`

HNSW (Hierarchical Navigable Small World) is a graph-based ANN algorithm with O(log n) query time.

**Layer structure**: Each inserted node appears in multiple layers with exponentially decreasing probability. Layer L probability: $P(\text{layer} \geq L) = e^{-mL \cdot \ln(m)}$ where m is the average degree.

```
Layer 2: sparse, few nodes  [entry point]
           A -------- F
Layer 1: denser
           A --- B --- F --- G
           |         |
Layer 0: full density (all nodes)
           A-B-C-D-E-F-G-H-I
```

**Construction** (inserting node q):
1. Find entry point at top layer
2. For each layer (top to bottom): greedy search for `ef_construction` nearest neighbors → connect q to M nearest
3. Bottom layer: connect to M × 2 neighbors (more connections for better recall)

**Query** (find k nearest to q):
1. Enter at top layer, greedy descent to bottom layer using current best node
2. At bottom layer: expand search using priority queue (ef_search nearest candidates)
3. Return top k from candidates

**Key parameters**:
- `M` (16–64): max connections per node. Higher = better recall, more memory. M=16 is a common default.
- `ef_construction` (100–200): beam width during construction. Higher = better index quality, slower build.
- `ef_search` (50–200): beam width during query. Higher = better recall, slower query. Tune to hit recall target.

**Memory cost**: each node stores M neighbor IDs per layer. For N=1M vectors, d=128 dims, M=16: ~N×M×4 bytes overhead ≈ 64MB for neighbor lists + 1M×128×4=512MB for vectors = ~576MB total.

---

**Q: What is Product Quantization (PQ) and how does it compress vectors?** `[A]`

Product Quantization reduces storage by 16–64× with modest recall loss.

**Core idea**: divide each vector into M subvectors; quantize each subvector to the nearest centroid in a learned codebook.

```
Original vector: [v₁, v₂, ..., v₁₂₈]
Split into 8 subvectors of 16 dims each:
  [v₁...v₁₆] → nearest of 256 centroids → code (1 byte)
  [v₁₇...v₃₂] → nearest of 256 centroids → code (1 byte)
  ...
  [v₁₁₃...v₁₂₈] → nearest of 256 centroids → code (1 byte)

Result: 8 bytes per vector (vs 128×4 = 512 bytes) → 64× compression
```

**Training**: run K-means separately on each subspace → M codebooks of 256 centroids each.

**ADC (Asymmetric Distance Computation)**: during query, the query vector is NOT quantized (kept at full precision). Distance computation uses a lookup table: precompute distance from query subvector to each centroid. This asymmetric approach reduces quantization error significantly.

**IVF-PQ**: Combine PQ with IVF (Inverted File Index) for large-scale search:
1. Cluster all vectors into K cells (IVF)
2. At query time, search only `nprobe` nearest cells
3. Within each cell, use PQ + ADC for fast approximate distance computation

Enables **billions of vectors in memory** (IVF-PQ with 1B vectors: ~4GB storage vs 4TB for float32).

---

**Q: What is DiskANN and when should you use it?** `[A]`

DiskANN (Microsoft, 2019) is a disk-based ANN algorithm for datasets too large to fit in RAM.

**Key insight**: HNSW requires the full graph in RAM for fast traversal. DiskANN stores the graph on SSD and uses:
- **Fresh DRAM cache**: cache the top 1–2 layers of the graph in RAM (often only ~1GB for billion-scale indexes)
- **Beam search with SSD I/O**: for each candidate expansion, batch read neighbors from SSD
- **Compressed in-RAM vectors**: store PQ-compressed versions of all vectors in RAM for fast candidate pruning; only load full-precision vectors from disk for final re-ranking

**Performance on 1B vectors (1536 dims)**:
- Storage: ~6GB on SSD (with 8-byte PQ compression per vector)
- RAM: ~5GB (PQ vectors + graph top layers + OS cache)
- QPS: ~1K queries/sec on single server
- Recall@10: >95%

**When to use DiskANN**:
- >100M vectors
- RAM cost is prohibitive
- SSD latency is acceptable (10–30ms per query vs 1–5ms for HNSW in RAM)

Available in Azure Cognitive Search (Microsoft's own product) and open-source (github.com/microsoft/DiskANN).

---

**Q: What are FAISS index types and when do you use each?** `[I]`

FAISS (Facebook AI Similarity Search) is the most widely used ANN library. Index type selection:

| Index | Description | When to Use | Notes |
|-------|-------------|-------------|-------|
| `IndexFlatL2` / `IndexFlatIP` | Exact brute-force search | <100K vectors, need perfect recall | Baseline; no approximation |
| `IndexIVFFlat` | IVF with exact cell search | 1M–10M vectors, moderate recall | Set `nprobe` to tune recall/speed |
| `IndexIVFPQ` | IVF + Product Quantization | 10M–1B+ vectors, memory-limited | 16–64× compression; use for scale |
| `IndexHNSWFlat` | HNSW with full-precision vectors | 1M–100M, low-latency requirement | Best recall/speed tradeoff |
| `IndexHNSWSQ` | HNSW + scalar quantization | 1M–100M, moderate memory | 4× compression, ~2% recall loss |
| `IndexScalarQuantizer` | SQ8 vector storage | Any, add to above | Reduce memory 4× |

**Practical recipe for production**:
- < 500K vectors: `IndexFlatL2` (exact is fast enough)
- 500K–50M: `IndexIVFFlat` with `nlist=sqrt(N)`, `nprobe=20–50`
- 50M+: `IndexIVFPQ` with `M=8` (subspaces), `nbits=8` (256 centroids per subspace)

---

**Q: How do you run a vector database in production with high availability?** `[I]`

Production vector DB requirements: uptime, consistency, disaster recovery.

**Replication**:
- Most production vector DBs support primary-replica replication (Qdrant, Weaviate, Milvus)
- Writes go to primary; reads distributed across replicas
- Configure replica count ≥ 3 for HA

**Sharding** (when index exceeds single node):
- Hash-based sharding: assign vectors to shards by ID modulo shard count
- Range-based sharding: by time or document category
- Most production DBs handle sharding automatically (Weaviate, Milvus)

**Backup and restore**:
- Snapshot entire index to object storage (S3) periodically
- Restore time depends on index size — pre-warm by running a batch of queries after restore
- Test restore procedures quarterly

**Monitoring**:
- Query latency (p50, p99)
- QPS (queries per second)
- Index size and growth rate
- Recall drift (run golden query set periodically; alert if recall drops)

---

[← Fine-Tuning](05-fine-tuning.md) | [AI System Design →](07-ai-system-design.md)
