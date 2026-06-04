# RAG Pipeline Checklist

[← Back to README](../README.md) | [See All Cheatsheets](../README.md#-cheatsheets)

> A complete checklist for building, evaluating, and debugging RAG systems. Use this as a reference when designing a system or preparing for interviews.

---

## RAG Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  INGESTION (offline)                                            │
│                                                                 │
│  Documents → Chunking → Embedding → Vector DB + Metadata Store │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  RETRIEVAL (online)                                             │
│                                                                 │
│  Query → Query Rewriting → Hybrid Search → Re-ranking → Top-K  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  GENERATION (online)                                            │
│                                                                 │
│  Prompt Assembly → LLM → Guardrails → Response + Citations     │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ Ingestion Pipeline Checklist

### Document Processing
- [ ] Handle all input formats (PDF, DOCX, HTML, Markdown, CSV, images)
- [ ] Extract clean text (strip headers/footers, remove boilerplate)
- [ ] Preserve metadata: source URL, title, author, date, section headings
- [ ] Handle tables: extract as structured data AND as natural language summary
- [ ] Handle images: extract alt text or run OCR / image captioning

### Chunking Strategy
- [ ] Choose strategy based on document type:
  - Uniform prose → Recursive character splitting (512 tokens, 10–20% overlap)
  - Technical docs with clear sections → Section-aware splitting
  - Mixed topics → Semantic chunking
  - Q&A style content → Split on Q&A boundaries
- [ ] Include source metadata in each chunk (document ID, page number, section)
- [ ] Consider parent-child chunking: small chunk for retrieval, parent chunk for context
- [ ] Test chunk quality: retrieve a known query and manually inspect returned chunks

### Embedding
- [ ] Choose embedding model appropriate to domain:
  - General: `text-embedding-3-large`, `embed-english-v3.0` (Cohere)
  - Code: code-specialized embedding (CodeBERT, code-embed models)
  - Multilingual: `multilingual-e5-large`, `embed-multilingual-v3.0`
- [ ] Embed documents and queries with the same model
- [ ] Store embeddings with metadata in vector DB
- [ ] Optionally: maintain BM25 index for hybrid search

### Infrastructure
- [ ] Choose vector DB for your scale:
  - Prototype: ChromaDB, FAISS (in-memory)
  - Production (small-medium): Qdrant, Weaviate
  - Production (large): Pinecone, pgvector, Milvus
- [ ] Set up metadata filtering (user ACLs, date filters, source filters)
- [ ] Plan incremental updates (add new docs without full re-index)
- [ ] Set up deletion/update handling (when source docs change)

---

## ✅ Retrieval Pipeline Checklist

### Query Processing
- [ ] Query expansion / rewriting (optional but high-value):
  - HyDE (Hypothetical Document Embedding): generate a fake answer, embed it, use for retrieval
  - Query decomposition: break complex queries into sub-queries
  - Multi-query: generate 3–5 query variants, retrieve for each, deduplicate
- [ ] Extract filters from query (dates, entities, specific sources) and apply as metadata filters

### Search
- [ ] Implement hybrid search (BM25 + vector):
  - Run both separately
  - Merge with Reciprocal Rank Fusion (RRF):
    ```python
    def reciprocal_rank_fusion(rankings: list[list], k=60):
        scores = {}
        for ranking in rankings:
            for rank, doc_id in enumerate(ranking):
                scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank + 1)
        return sorted(scores.items(), key=lambda x: x[1], reverse=True)
    ```
- [ ] Retrieve top-50 candidates (not top-5 — leave re-ranking to do its job)
- [ ] Apply access control filtering BEFORE returning results

### Re-ranking
- [ ] Apply cross-encoder re-ranker on top-50 candidates
  - Options: `cross-encoder/ms-marco-MiniLM-L-6-v2`, Cohere Rerank, Jina Reranker
- [ ] Keep top-5 chunks for the final LLM prompt
- [ ] Log which documents were retrieved and their scores (for debugging)

---

## ✅ Generation Pipeline Checklist

### Prompt Assembly
- [ ] System prompt: instruct model to answer ONLY from provided context
- [ ] Include source attribution instructions: "cite the document title and page number"
- [ ] Format context clearly with delimiters:
  ```
  <context>
  [Source: doc_title.pdf, Page 3]
  ...chunk text...
  
  [Source: other_doc.pdf, Page 7]
  ...chunk text...
  </context>
  ```
- [ ] Handle "no relevant context found" gracefully (don't hallucinate)

### LLM Call
- [ ] Set temperature low (0.1–0.3) for factual Q&A
- [ ] Use structured output (JSON) when citations need to be machine-parseable
- [ ] Set max_tokens appropriately — don't let model pad answers
- [ ] Handle rate limits and implement retry with exponential backoff

### Output Processing
- [ ] Extract citations and validate they appear in the retrieved context
- [ ] Run faithfulness check: does the answer claim anything not in the context?
- [ ] Detect and handle empty/refused responses
- [ ] Format for display (render markdown, resolve citation links)

---

## ✅ Evaluation Checklist

### Build a Golden Dataset (highest priority)
- [ ] Collect 100–500 representative queries
- [ ] Label expected answers AND relevant source documents
- [ ] Include edge cases: empty retrieval, conflicting sources, time-sensitive queries
- [ ] Version control your golden dataset

### Retrieval Metrics
| Metric | Formula | Target |
|--------|---------|--------|
| Recall@K | % of queries where relevant doc in top K | >0.85 for K=5 |
| MRR | mean(1/rank of first relevant doc) | >0.7 |
| Context Precision | % of retrieved chunks that are actually relevant | >0.7 |

### Generation Metrics (use RAGAS or LLM-as-judge)
| Metric | What It Measures | Target |
|--------|-----------------|--------|
| Faithfulness | Answer only uses context facts | >0.85 |
| Answer Relevance | Answer actually addresses the query | >0.80 |
| Context Recall | Used all relevant context in context | >0.75 |

### RAGAS quick setup
```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_recall

results = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevancy, context_recall]
)
```

---

## 🐛 Common Failure Modes & Fixes

| Failure | Symptom | Likely Cause | Fix |
|---------|---------|-------------|-----|
| Wrong chunks retrieved | Answer misses key information | Embedding model poor fit, bad chunking | Switch embedding model; adjust chunk size; add query rewriting |
| Hallucination despite context | Answer adds facts not in context | LLM didn't stay grounded | Stronger system prompt; lower temperature; add faithfulness check |
| Exact terms not found | "What is RFC 2234?" returns nothing | Pure vector search misses exact term | Add BM25; enable hybrid search |
| Slow retrieval | >500ms for vector search | Index too large without filtering | Enable metadata pre-filtering; add ANN index tuning |
| Stale answers | Answer uses outdated info | Old chunks in index | Add date metadata; filter by recency; incremental re-index |
| Access control leak | User A sees User B's data | Missing ACL filter on retrieval | Apply user-based metadata filter BEFORE returning results |
| Lost in the middle | Key info in context but ignored | Long context attention issue | Put most important chunks first; reduce context length |
| Chunk splits key info | Answer misses split concept | Chunk boundary cuts a key sentence | Add overlap; use semantic chunking; increase chunk size |

---

## 🔧 Production Tuning Parameters

| Parameter | Start Value | Tune When |
|-----------|------------|-----------|
| Chunk size | 512 tokens | Recall@5 < 0.80 |
| Chunk overlap | 10–15% | Answers miss context at chunk boundaries |
| Retrieval K | 50 candidates | High false positives after re-ranking |
| Re-rank top-N to LLM | 5 | Faithfulness drops (too many chunks = noise) |
| Embedding model | text-embedding-3-large | Retrieval recall low for domain-specific queries |
| LLM temperature | 0.1 | Hallucination rate high |

---

## RAG vs Fine-Tuning Decision

```
Does it need factual, updateable knowledge?
    ├── YES → RAG (can update without retraining)
    └── NO  ↓
Does it need specific output style/format/behavior?
    ├── YES → Fine-tuning (or prompting + examples)
    └── NO  ↓
Is prompting enough for the task?
    ├── YES → Use prompting only
    └── NO  → Evaluate both; likely need RAG + fine-tuning
```

---

📖 Related: [RAG deep dive](../topics/03-rag.md) | [LLM Formulas](llm-formulas.md) | [System Design Patterns](system-design-patterns.md)
