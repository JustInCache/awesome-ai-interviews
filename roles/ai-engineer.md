# AI Engineer Interview Questions

[← Back to Home](../README.md) | [See All Roles](../README.md#by-role)

Scenario-based interview questions for AI Engineer roles. Each question reflects a real production challenge.

**Related**: [LLM Fundamentals](../topics/01-llm-fundamentals.md) | [RAG](../topics/03-rag.md) | [AI Agents](../topics/04-ai-agents.md) | [LLMOps](../topics/08-llmops-production.md)

---

## What AI Engineers Do

AI Engineers build and ship AI-powered products. Unlike ML Engineers (who focus on training pipelines) or AI Researchers (who advance the field), AI Engineers specialize in integrating foundation models into reliable, scalable applications — using prompting, RAG, fine-tuning, and agentic architectures.

**Key skills**: LLM API integration, RAG pipelines, agent systems, inference optimization, evaluation, prompt engineering, LLMOps.

---

## Interview Questions

### Q1: Context & Memory Management

> **A user says your AI assistant forgets important details mentioned earlier in a long conversation. What's going on, and how do you fix it?**

This is a **context window limitation**, not a model failure. LLMs are stateless and can only attend to a fixed number of tokens. Once the conversation exceeds that limit, older information is dropped or compressed in ways that lose detail.

**The fix is architectural:**

**1. Summarization memory** — Periodically compress conversation history into a running summary. Model sees summary + recent messages.
- Pros: simple, low latency
- Cons: lossy — specific details may be lost

**2. Vector memory (RAG-style)** — Store conversation turns as embeddings. When user asks something, retrieve relevant past context on demand.
- Pros: precise recall of specific facts
- Cons: retrieval latency, embedding infrastructure needed

**3. Hybrid** (recommended for production) — Summaries for general continuity + vector retrieval for important facts (user preferences, key decisions).

**Diagnosis first**: instrument the system to understand *what* users are forgetting — explicit facts, implicit preferences, or task context from many turns ago. Then choose the memory strategy based on use case:
- Customer support bot → summarization is enough
- Personal assistant → vector memory with persistence
- Complex multi-turn workflows → hybrid with structured state management

---

### Q2: Inference Latency Optimization

> **Your AI feature takes 3–4 seconds. The product team wants it under 500ms. How do you approach this?**

**Step 1: Profile the full request path.** A typical slow response:

| Component | Time |
|-----------|------|
| Network round-trip | 100ms |
| Preprocessing & embedding | 200ms |
| Vector search / retrieval | 300ms |
| LLM inference | 2500ms |
| Post-processing | 100ms |
| **Total** | **3200ms** |

**Step 2: Attack the bottleneck.**

*If LLM inference is slow*:
- Use a smaller model (7B often matches 70B for narrow tasks)
- Enable streaming — perceived latency drops dramatically
- Apply speculative decoding (2–3× speedup)
- Quantize to INT8/INT4 (half the inference time, minimal quality loss)
- Semantic caching for repeated queries

*If retrieval is slow*:
- Use ANN indexing (HNSW) over exact search
- Reduce embedding dimensions
- Apply metadata pre-filtering to narrow search space

**Step 3: Have the trade-off conversation with product**:
- 500ms with slight quality drop (smaller model, aggressive caching)
- 800ms with same quality (optimized pipeline)
- 200ms *perceived* with 2s actual (streaming + progressive display)

Streaming is often the right answer — users tolerate generation time much better than wait time.

---

### Q3: Prompt Injection Defense

> **Users discover they can manipulate your AI assistant by typing "Ignore all previous instructions and do X." How do you defend against this?**

Prompt injection is hard because LLMs can't fundamentally distinguish trusted instructions (system prompt) from untrusted input (user messages) — they're all tokens.

**Defense layers** (no single solution is sufficient):

**1. Input validation layer** — Secondary classifier (fine-tuned or rule-based) detects injection patterns before the prompt reaches the main LLM. Flag and reject/modify suspicious inputs.

**2. Privilege separation** — Never co-locate system instructions and user input in ways that allow user text to override system intent. Structure:
```
System: [trusted instructions — never user-modifiable]
User: [untrusted content — treat as data, not instructions]
```

**3. Output validation** — After generation, verify the response stays within expected task scope. A summarizer should output summaries; if it outputs system instructions, something went wrong.

**4. Indirection** — Don't embed sensitive configuration in the system prompt that would be harmful to reveal. Reference external config; treat system prompt contents as potentially exposed.

**5. Sandboxed execution** — For agents, limit what tools can be invoked based on user role, not just LLM decision.

**6. Adversarial testing** — Continuously red-team your system; share attacks with the team.

The goal is defense in depth, not a single magic fix.

---

### Q4: RAG Quality Issues

> **Your RAG system retrieves relevant documents but still gives wrong answers. How do you debug this?**

When retrieval looks good but answers are wrong, the failure is in the **generation step** or in a mismatch between what's retrieved and what's needed.

**Diagnostic checklist**:

1. **Verify retrieval quality** — Look at the actual retrieved chunks. Are they relevant? Is the key information *actually present*? High recall@5 doesn't mean the critical sentence was retrieved.

2. **Context window placement** — LLMs suffer from "lost in the middle" — they attend more to the start and end of context. Put the most relevant chunk first or last.

3. **Chunk granularity** — Is the chunk too large (relevant info buried in noise) or too small (missing surrounding context)? Try parent-child chunking.

4. **Hallucination source** — Is the model generating from its own knowledge instead of the provided context? Add explicit instruction: "Answer ONLY based on the provided documents. If the answer is not in the documents, say 'I don't have that information.'"

5. **Re-ranking** — After vector retrieval, apply a cross-encoder re-ranker to reorder chunks by relevance to the *full query*. Often fixes cases where BM25/ANN retrieval brings in the right chunk but ranks it #4 instead of #1.

6. **RAGAS evaluation** — Measure faithfulness (answer grounded in context?) and context precision/recall to pinpoint the failure step.

---

### Q5: Model Deployment Strategy

> **You need to deploy a 70B LLM for production API serving. Walk me through your infrastructure decisions.**

**GPU selection**: 70B model in BF16 = 140 GB VRAM. Options:
- 2× A100 80GB with tensor parallelism (TP=2)
- 1× H100 80GB with INT8 quantization (~70 GB)
- 4× A100 40GB with TP=4 (more GPUs, lower cost per GPU)

**Serving framework**: vLLM with continuous batching. PagedAttention handles KV cache efficiently; continuous batching maximizes GPU utilization.

**Quantization decision**: INT8 with AWQ or GPTQ — ~2% quality loss, 2× memory reduction, 1.5–2× inference speedup. Worth it for 70B.

**Latency optimization**:
- KV cache prefix caching for constant system prompt (SGLang RadixAttention)
- TTFT optimization: KV cache warming, tensor parallel for prefill
- Streaming responses

**Scaling**: horizontal pod autoscaler on GPU cluster keyed on request queue depth (not CPU/memory). Keep minimum 2 replicas to avoid cold-start latency. Use spot/preemptible instances for batch workloads.

**Monitoring**: Langfuse tracing for request traces; DCGM for GPU metrics; alert on P95 TTFT > 2s.

---

### Q6: Data Pipeline Design

> **Design a data pipeline that continuously updates your RAG knowledge base from multiple sources (Slack, Confluence, GitHub).**

**Architecture**:

```
Sources                  Ingestion              Storage
──────────               ─────────              ───────
Slack Events API    →    Event consumer         Raw docs (S3)
Confluence webhooks →    Document processor →   Vector DB (Qdrant)
GitHub webhooks    →    Chunker + Embedder      Elasticsearch (keyword)
                         ↑
                    Change detection
                    (content hash comparison)
```

**Processing pipeline**:
1. Source connector (poll or webhook) → raw document
2. Content hash check: skip if unchanged since last ingestion
3. Parse format (Markdown, PDF, HTML, code)
4. Chunk: semantic chunking with overlap
5. Enrich metadata: source, author, timestamp, access permissions
6. Embed: `text-embedding-3-large`
7. Upsert: vector DB + search index (with document ID for updates)
8. Invalidate semantic cache for affected topics

**Staleness handling**: each chunk stores `source_version` and `ingested_at`. Stale chunks surface in citations with "Last updated: X days ago."

**Access control**: store ACL per chunk; filter at query time.

---

### Q7: Long-Conversation Memory Architecture

> **Design memory for an AI assistant that needs to remember facts about a user across multiple sessions over months.**

**Three-tier memory**:

| Tier | Storage | What's Stored | Retrieval |
|------|---------|--------------|-----------|
| Working | Context window | Current session messages | Always included |
| Episodic | Vector DB | Session summaries, key events | Semantic search |
| Semantic | Structured DB | User facts, preferences | Direct lookup |

**Session end processing**:
```python
def end_session(session):
    # 1. Extract user facts (structured)
    facts = llm.extract("Extract stated preferences and facts about this user", session.history)
    user_facts_db.upsert(user_id, facts)
    
    # 2. Create episodic summary
    summary = llm.summarize(session.history)
    vector_db.upsert(session_id, embed(summary), metadata={"user_id": user_id, "date": today})
```

**Session start retrieval**:
- Always load semantic facts (user name, preferences, key context)
- Retrieve top-3 relevant past session summaries by semantic similarity to current query
- Recent session summary (last conversation) always included

**Privacy**: retention policy — auto-delete episodic memories older than N months unless marked important. User-accessible memory management UI.

---

### Q8: Observability for AI Inference

> **How do you instrument an AI system to debug a production issue where users report getting wrong answers for specific queries?**

**Step 1: Structured logging at every pipeline step**:
```python
with tracer.start_span("rag_pipeline") as span:
    span.set_attribute("query", query)
    
    chunks = retrieve(query)
    span.set_attribute("retrieved_chunks", [c.id for c in chunks])
    span.set_attribute("retrieval_scores", [c.score for c in chunks])
    
    response = generate(query, chunks)
    span.set_attribute("response", response)
    span.set_attribute("prompt_tokens", tokens_in)
    span.set_attribute("completion_tokens", tokens_out)
```

**Step 2: Reproduce the failing query** — query the tracing system for that specific query; inspect the full execution trace.

**Common failure patterns**:
- Wrong chunks retrieved (retrieval failure) → tune chunking or embedding model
- Right chunks retrieved but answer ignores them (generation failure) → prompt issue
- No relevant chunks retrieved (coverage gap) → missing content in knowledge base
- Guardrail falsely blocking the query → tune guardrail thresholds

**Step 3: Root cause with LLM-as-judge** — send the trace (query, retrieved chunks, response) to an evaluator: "Given these retrieved documents, is this response faithful and accurate? If not, what went wrong?"

**Step 4: Add to regression suite** — once fixed, add the query to the golden evaluation dataset so it can't regress.

---

## Preparation Checklist

For AI Engineer interviews, be ready to discuss:

- [ ] How RAG works end-to-end, including chunking and retrieval strategies
- [ ] LLM latency optimization techniques
- [ ] Prompt injection and input/output guardrails
- [ ] Agent architectures (ReAct, Plan-and-Execute)
- [ ] LLM evaluation (LLM-as-judge, RAGAS)
- [ ] Production monitoring and tracing (Langfuse, Arize)
- [ ] Cost optimization (model routing, semantic caching)
- [ ] Vector databases and embedding models

---

[← Back to Home](../README.md) | [ML Engineer →](ml-engineer.md)
