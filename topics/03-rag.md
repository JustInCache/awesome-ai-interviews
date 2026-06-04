# Retrieval-Augmented Generation (RAG)

[← Prompt Engineering](02-prompt-engineering.md) | [AI Agents →](04-ai-agents.md)

40+ questions covering RAG architecture, chunking, retrieval, evaluation, and production failure modes.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Core Concepts](#core-concepts)
- [Chunking & Indexing](#chunking--indexing)
- [Retrieval Techniques](#retrieval-techniques)
- [Generation & Synthesis](#generation--synthesis)
- [Evaluation](#evaluation)
- [Advanced RAG Patterns](#advanced-rag-patterns)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Core Concepts

**Q: What is RAG and why is it important?** `[B]`

RAG (Retrieval-Augmented Generation) grounds LLM responses in retrieved external knowledge:

```
User Query → Retrieve relevant docs → Augment prompt with context → LLM generates answer
```

**Why not just use the LLM's parametric knowledge?**
- LLMs have a knowledge cutoff — they don't know recent events
- LLMs hallucinate facts when uncertain — retrieved facts are verifiable
- LLMs were never trained on private enterprise data
- Updating knowledge requires retraining (expensive) without RAG

RAG separates **knowledge** (vector store) from **reasoning** (LLM), letting you update either independently.

---

**Q: Explain the architecture of a basic RAG system.** `[B]`

```
INDEXING PIPELINE (offline):
Documents → Chunking → Embedding Model → Vector Database

QUERY PIPELINE (online):
User Query → Embed Query → Vector Search → Top-K Chunks
           → [Re-ranking] → Prompt Assembly → LLM → Answer
```

**Key components**:
- **Embedding model** — converts text to vectors for semantic similarity search
- **Vector database** — stores and indexes embeddings for fast approximate nearest neighbor search
- **Retriever** — finds the top-K most relevant chunks for a query
- **Generator** (LLM) — synthesizes a final answer from the retrieved context

---

**Q: What are the key components of a RAG pipeline?** `[B]`

| Component | Options | Key Decisions |
|-----------|---------|---------------|
| **Document loader** | LangChain loaders, PyMuPDF, Unstructured | File formats, OCR for scanned docs |
| **Chunker** | Fixed-size, recursive, semantic | Chunk size, overlap strategy |
| **Embedding model** | OpenAI ada-002, E5, BGE, Cohere | Domain fit, dimensionality, cost |
| **Vector store** | Pinecone, Weaviate, Chroma, pgvector | Scale, metadata filtering needs |
| **Retriever** | Dense, sparse, hybrid | Recall vs precision tradeoff |
| **Re-ranker** | Cohere rerank, BGE-reranker, cross-encoder | Quality vs latency |
| **LLM** | GPT-4, Claude, Llama | Quality, cost, latency |
| **Orchestration** | LangChain, LlamaIndex | Developer experience |

---

**Q: RAG vs fine-tuning — when do you choose each?** `[I]`

| Factor | Choose RAG | Choose Fine-Tuning |
|--------|-----------|-------------------|
| **Knowledge type** | Factual, frequently changing | Style, format, behavioral patterns |
| **Update frequency** | Daily/weekly knowledge updates | Behavior set once |
| **Auditability** | Need to cite sources | Output quality is the goal |
| **Training data** | Documents/text corpus available | High-quality Q&A pairs needed |
| **Cost** | Lower upfront | Higher (GPU training required) |
| **Hallucination risk** | Lower (grounded in docs) | Higher for knowledge injection |
| **Latency** | Adds retrieval step (~100-500ms) | No retrieval overhead |

**In practice**: start with RAG. Add fine-tuning if RAG can't achieve required output style or format. RAG + fine-tuning together achieves maximum performance (model adapts to domain, RAG provides fresh knowledge).

---

## Chunking & Indexing

**Q: What are chunking strategies and how do you choose chunk size?** `[I]`

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Fixed-size** | Split every N tokens with overlap | Baseline; homogeneous documents |
| **Sentence-based** | Split at sentence boundaries | QA, summaries |
| **Recursive** | Split on paragraph→sentence→word boundaries | General purpose (LangChain default) |
| **Semantic** | Cluster sentences by embedding similarity | Mixed-topic documents |
| **Parent-child** | Small chunks for retrieval, large for context | Dense technical docs |
| **Document-based** | Respect natural structure (sections, headings) | Structured documents (legal, medical) |

**Choosing chunk size**:
- Smaller chunks (128–256 tokens): higher retrieval precision, less context per chunk
- Larger chunks (512–1024 tokens): more context preserved, lower retrieval precision
- Sweet spot: 512 tokens, 10–20% overlap — start here, tune based on recall metrics

---

**Q: What is parent-child chunking?** `[I]`

Small chunks retrieve more precisely; large chunks provide better context to the LLM. Parent-child gets both:

1. **Index small chunks** (128 tokens) — high precision retrieval
2. **Return parent chunk** (512 tokens) to LLM when small chunk is retrieved

```
Document section (parent, 512 tokens) → stored in docstore
  ├── Small chunk 1 (child, 128 tokens) → indexed in vector store
  ├── Small chunk 2 (child, 128 tokens) → indexed in vector store
  └── Small chunk 3 (child, 128 tokens) → indexed in vector store

Query → retrieves child → look up parent → send parent to LLM
```

Effectively: you search with precision of small chunks but generate with context of large chunks.

---

**Q: How do you handle structured data (tables, PDFs with layouts) in RAG?** `[I]`

PDFs with tables, charts, and complex layouts are notoriously difficult:

1. **Table extraction**: use specialized parsers (Camelot, pdfplumber, Unstructured) to extract tables as structured data, then serialize to markdown or CSV before embedding
2. **Multi-modal**: use vision models (GPT-4V, LLaVA) to describe image/chart content; embed the descriptions
3. **Layout-aware chunking**: Unstructured's `partition_pdf()` preserves element types (title, table, narrative); chunk by element type, not just token count
4. **Separate indexes**: maintain separate vector indexes for tables vs. text; route queries to appropriate index based on query type classifier

---

## Retrieval Techniques

**Q: What is hybrid search and why is it better than pure vector search?** `[I]`

Pure **vector search** excels at semantic similarity but fails on:
- Exact strings, codes, product names
- Rare domain-specific terms not in the embedding model's training data
- Abbreviations and acronyms

Pure **keyword search (BM25)** excels at exact matches but misses:
- Synonyms, paraphrases
- Semantic meaning across vocabulary

**Hybrid search** combines both:
```
BM25 results + Vector similarity results → Reciprocal Rank Fusion (RRF) → Merged ranking
```

RRF: `score = Σ 1/(k + rank_i)` — penalizes low ranks, rewards consistently high ranks across both methods.

Virtually all production RAG systems use hybrid search for better recall across all query types.

---

**Q: What is re-ranking and how does it improve RAG quality?** `[I]`

Initial retrieval (BM25 + vector) is fast but imprecise — it optimizes for approximate similarity, not exact relevance. Re-ranking applies a **cross-encoder** that scores each candidate document against the query jointly:

```
Query + Doc A → CrossEncoder → 0.92
Query + Doc B → CrossEncoder → 0.61  ← would have been ranked #2
Query + Doc C → CrossEncoder → 0.88

After reranking: A → C → B
```

Cross-encoders are ~100× slower but far more accurate — they model query-document interaction directly instead of comparing independent embeddings. Apply only to top-50 initial candidates; pass top-5 to LLM.

Models: Cohere Rerank API, `BAAI/bge-reranker-v2-m3`, `cross-encoder/ms-marco-MiniLM-L-6-v2`.

---

**Q: What is query transformation in RAG?** `[I]`

User queries are often vague, ambiguous, or syntactically different from indexed content. Query transformation improves recall:

**HyDE (Hypothetical Document Embeddings)**: generate a hypothetical answer to the query, embed that answer, retrieve documents similar to the hypothetical answer. Works because the hypothetical answer is in the "document space" rather than the "question space."

**Query decomposition**: split a complex query into sub-questions, retrieve for each, merge results.
```
"Compare GPT-4 and Claude on reasoning and coding" →
  ["GPT-4 reasoning benchmarks", "Claude reasoning benchmarks", "GPT-4 coding", "Claude coding"]
```

**Step-back prompting**: generate a more abstract version of the query to retrieve foundational context.
```
"Why did LeCun say transformers can't scale?" → "LeCun's views on transformer limitations"
```

---

**Q: How do you implement metadata filtering in RAG?** `[I]`

Metadata filtering narrows the retrieval scope before semantic search — dramatically improving precision for structured data:

```python
# Only retrieve documents from the last 6 months, tagged as "product-docs"
results = vector_store.similarity_search(
    query="How do I configure authentication?",
    filter={
        "date": {"$gte": "2025-12-01"},
        "category": "product-docs",
        "department": user.department  # per-user access control
    },
    k=10
)
```

Use metadata for: date ranges, document type, author, team, access level, product version. Metadata filtering is also the primary mechanism for **per-user access control** — filter to only documents the requesting user has permission to see.

---

**Q: How do you handle multi-hop questions in RAG?** `[A]`

Multi-hop questions require combining information from multiple documents:
> "What technology does the company that acquired DeepMind in 2014 use for its search engine?"

Standard single-retrieval RAG fails because no single document contains the full answer.

**Strategies**:
1. **Query decomposition** — decompose into "Who acquired DeepMind in 2014?" + "What search technology does [company] use?" — retrieve and answer each, then combine
2. **Iterative retrieval** — answer hop 1, use that answer to form hop 2 query, continue
3. **GraphRAG** — if entities and relationships are indexed in a knowledge graph, traverse the graph rather than doing sequential retrieval
4. **Self-RAG** — model decides to retrieve again after each reasoning step

---

## Generation & Synthesis

**Q: Your RAG system retrieves relevant documents but answers are still wrong. What's happening?** `[I]`

This is the **retrieval-generation gap**. Possible causes:

1. **Retrieved but not used** — context too long, important info buried in middle (lost-in-the-middle); model uses parametric knowledge instead. Fix: shorter context, explicit grounding instruction.
2. **Chunk boundary problem** — answer spans multiple chunks; each chunk alone is incomplete. Fix: larger chunks, parent-child chunking, overlapping chunks.
3. **Synthesis failure** — needs combining information from multiple docs; model summarizes each separately. Fix: explicit synthesis instruction; multi-hop decomposition.
4. **Hallucination despite context** — model "confabulates" plausible extensions of the retrieved text. Fix: "Answer ONLY based on the provided context. If the answer is not there, say so."
5. **Wrong retrieval despite good metrics** — eval metrics measure the wrong thing. Inspect actual retrieved chunks manually.

---

**Q: How do you implement citation and source attribution in RAG?** `[I]`

Force the model to quote and reference source chunks:

```python
system_prompt = """
Answer the user's question based on the provided context.
For every claim you make, include a citation in the format [Source: doc_id, chunk_id].
If information is not in the provided context, say "I don't have information about this."
"""
```

**Post-processing**: extract citation references from output, look up original chunks, verify the cited chunk actually supports the claim (LLM-as-judge for faithfulness), append source URLs or document metadata to the response.

**UI pattern**: show inline citations; let users click to see the original source chunk.

---

## Evaluation

**Q: How do you evaluate a RAG system?** `[I]`

Evaluate retrieval and generation separately, then end-to-end:

**Retrieval metrics** (offline, against labeled dataset):
- **Recall@K** — is the relevant document in the top K results?
- **MRR (Mean Reciprocal Rank)** — how highly is the relevant document ranked?
- **Context Precision@K** — of the K retrieved chunks, what fraction are actually relevant?

**Generation metrics** (LLM-as-judge or model-based):
- **Faithfulness** — is every claim in the answer supported by the retrieved context? (anti-hallucination metric)
- **Answer Relevance** — does the answer actually address the query?
- **Context Recall** — does the answer use all the important information in the context?

**Framework**: RAGAS automates all of the above using an LLM as evaluator. Maintain a golden dataset of (query, expected answer, relevant documents) for regression testing after every change.

---

## Advanced RAG Patterns

**Q: What is GraphRAG?** `[A]`

Standard RAG retrieves locally similar chunks but misses global patterns across the entire corpus. GraphRAG (Microsoft, 2024) builds a knowledge graph:

1. LLM extracts entities and relationships from every document
2. Build graph: nodes = entities, edges = relationships with context
3. Use community detection (Leiden algorithm) to cluster related entities
4. Generate LLM summaries for each community

At query time:
- **Local search**: standard vector search + graph traversal for specific entity questions
- **Global search**: query community summaries for questions about the whole corpus ("What are the main themes across all documents?")

**Cost**: much higher indexing cost (LLM call per document chunk). Best for knowledge-dense corpora where relationship analysis matters.

---

**Q: What is Agentic RAG?** `[I]`

Standard RAG is a fixed pipeline: query → retrieve → generate. Agentic RAG makes retrieval a dynamic tool:

```
User: "Summarize the key risks from Q4 reports of our top 3 competitors"

Agent:
  Thought: I need to find Q4 reports for competitors A, B, C
  Action: retrieve("Competitor A Q4 report")
  Observation: [chunks]
  Thought: Need B and C as well
  Action: retrieve("Competitor B Q4 report")
  ...
  Thought: Now I can synthesize
  Answer: [synthesized summary with risks]
```

Agent decides **whether to retrieve**, **what to search for**, **when to search again** if results are insufficient. Key patterns: Self-RAG (model learns retrieval decision tokens), FLARE (retrieve when model is uncertain), ReAct with retrieval tool.

---

**Q: What is Self-RAG?** `[A]`

Self-RAG (Asai et al., 2023) fine-tunes a model to output special reflection tokens that govern when and how to retrieve:

- `[Retrieve]` — model decides to retrieve before continuing
- `[ISREL]` / `[ISIRREL]` — model evaluates if retrieved doc is relevant
- `[ISSUP]` / `[ISNOSUP]` — model evaluates if its statement is supported by retrieved doc
- `[ISUSE]` / `[ISNOUSE]` — model evaluates utility of retrieved doc for the task

This gives fine-grained, in-generation control over retrieval decisions. The model can skip retrieval for factual claims it's confident about and retrieve for others — reducing latency and improving faithfulness simultaneously.

---

**Q: How do you scale a RAG system to millions of documents?** `[A]`

**Indexing at scale**:
- Parallel embedding pipeline with batch processing (embed thousands of chunks/sec with GPU)
- Incremental index updates (append-only; avoid full re-indexing for every change)
- Distributed vector databases (Qdrant, Weaviate, Milvus) with sharding

**Query at scale**:
- HNSW indexing for sub-linear approximate nearest neighbor search
- Metadata pre-filtering reduces search space before ANN
- Horizontal scaling with read replicas
- Query result caching (identical queries, semantic caching for near-identical queries)

**Quality at scale**:
- At millions of docs, precision matters more — top-100 candidates out of 1M are all "similar"; re-ranking is essential
- Maintain per-domain sub-indexes for routing (reduce noise from irrelevant domains)

---

## Troubleshooting Scenarios

**Q: Your RAG system returns duplicate or redundant results. How do you fix it?** `[I]`

1. **MMR (Maximal Marginal Relevance)**: when selecting top-K, prefer results that are both relevant AND dissimilar to already-selected results
2. **Deduplication at indexing**: detect near-duplicate chunks during ingestion (cosine similarity > 0.98) and deduplicate before indexing
3. **Parent-child chunking**: since child chunks from the same parent would be retrieved together, deduplicate to one parent per result
4. **Post-retrieval deduplication**: cluster retrieved chunks by similarity; keep one representative per cluster

---

**Q: Your RAG system fails for domain-specific jargon the embedding model doesn't understand.** `[I]`

1. **Keyword search augmentation**: add BM25 alongside vector search — BM25 matches exact terms even if embeddings don't capture meaning
2. **Domain-specific embedding model**: fine-tune an embedding model on domain corpus; or use one pre-trained on similar domains (e.g., `BioLinkBERT` for biomedical)
3. **Synonym expansion**: build a domain glossary; expand queries with known synonyms before retrieval
4. **Chunk size reduction**: smaller chunks improve precision for specialized terminology retrieval

---

**Q: Your RAG knowledge base is updated frequently. How do you manage freshness?** `[I]`

1. **Incremental ingestion pipeline**: trigger re-embedding only for changed/new documents (change detection via MD5 hash or document version field)
2. **Soft delete + re-index**: mark outdated chunks as inactive; re-index updated version; run background cleanup
3. **Metadata timestamps**: store `last_updated` with each chunk; at query time, optionally filter to documents updated within N days
4. **Two-index pattern**: "current" index (recent, high quality) + "archive" index; route time-sensitive queries to current index

---

**Q: Your RAG system gives contradictory answers from different source documents.** `[A]`

1. **Explicit contradiction detection**: after retrieval, run a check: "Do any of these documents contradict each other? Identify conflicts." Then handle based on policy.
2. **Source authority ranking**: establish a hierarchy (official documentation > blog post > user comment); when conflicts exist, prefer higher-authority source
3. **Date-based resolution**: for evolving facts, prefer the most recent document; include `last_updated` in metadata and prompt
4. **Expose the conflict**: rather than hiding it, surface contradictions to users: "Source A says X, but Source B says Y. The most recent source (2025) indicates Y."
5. **Structured retrieval with source annotation**: require the LLM to attribute every claim to a specific source; contradictions become visible in the output

---

[← Prompt Engineering](02-prompt-engineering.md) | [AI Agents →](04-ai-agents.md)
