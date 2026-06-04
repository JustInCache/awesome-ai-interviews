# AI System Design Patterns Cheatsheet

[← Back to README](../README.md) | [See All Cheatsheets](../README.md#-cheatsheets)

> Canonical patterns that appear in AI system design interviews. Study these architectures, know the trade-offs, and adapt them to the problem at hand.

---

## Pattern 1: RAG System

**Use when**: private/updateable knowledge, citation required, no fine-tuning data.

```
┌────────────┐    ┌─────────────┐    ┌──────────────┐    ┌─────┐
│  Documents │───▶│   Chunking  │───▶│  Embeddings  │───▶│VecDB│
└────────────┘    └─────────────┘    └──────────────┘    └──┬──┘
                                                             │
User Query ──▶ Query Rewrite ──▶ Hybrid Search ──────────────┘
                                       │
                                   Re-ranking
                                       │
                               Prompt Assembly
                                       │
                                  LLM Generate
                                       │
                              Guardrails + Citations
                                       │
                                  User Response
```

**Key decisions**: chunk size (512), embedding model, hybrid search (BM25 + vector), re-ranker.

📖 [RAG deep dive](../topics/03-rag.md) | [RAG Checklist](rag-checklist.md)

---

## Pattern 2: AI Agent Loop

**Use when**: multi-step tasks, tool use required, environment interaction.

```
┌─────────────────────────────────────────────────────────────────┐
│                       AGENT LOOP                                │
│                                                                 │
│  User Task                                                      │
│      │                                                          │
│      ▼                                                          │
│  ┌───────┐    ┌──────────────────────────────────────────────┐ │
│  │ LLM   │◀───│ Context:                                      │ │
│  │       │    │  - System prompt + tool definitions           │ │
│  │Reason │    │  - Task description                          │ │
│  │  +    │    │  - Working memory (past steps + observations) │ │
│  │ Plan  │    └──────────────────────────────────────────────┘ │
│  └───┬───┘                                                      │
│      │                                                          │
│      ├── Tool Call? ──▶ Execute Tool ──▶ Observation ──▶ Loop  │
│      │                                                          │
│      └── Final Answer? ──▶ Output to User                      │
└─────────────────────────────────────────────────────────────────┘
```

**Key decisions**: tool set (principle of least privilege), memory strategy, loop exit conditions, human-in-the-loop checkpoints for irreversible actions.

📖 [AI Agents deep dive](../topics/04-ai-agents.md)

---

## Pattern 3: Multi-Agent System (Orchestrator-Worker)

**Use when**: tasks too complex for a single agent, need specialization.

```
                    ┌──────────────────┐
                    │   ORCHESTRATOR   │
                    │   (Planner LLM)  │
                    └────────┬─────────┘
                             │ routes sub-tasks
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
       ┌──────────┐   ┌──────────┐   ┌──────────┐
       │ Retrieval│   │  Coding  │   │ Analysis │
       │  Agent   │   │  Agent   │   │  Agent   │
       └──────────┘   └──────────┘   └──────────┘
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌──────────────────┐
                    │  Shared Context  │
                    │     Store        │
                    └──────────────────┘
                             │
                    ┌──────────────────┐
                    │    VERIFIER      │
                    │ (Checks output)  │
                    └──────────────────┘
```

**Key decisions**: role decomposition (single responsibility), shared context format, conflict resolution when agents disagree, verifier triggers.

---

## Pattern 4: LLM Inference Stack

**Use when**: deploying self-hosted LLMs at scale.

```
                         Load Balancer
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       ┌────────────┐  ┌────────────┐  ┌────────────┐
       │  Inference │  │  Inference │  │  Inference │
       │  Node 1    │  │  Node 2    │  │  Node 3    │
       │ (vLLM)     │  │ (vLLM)     │  │ (vLLM)     │
       │            │  │            │  │            │
       │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │
       │ │GPU 0-1 │ │  │ │GPU 0-1 │ │  │ │GPU 0-1 │ │
       │ └────────┘ │  │ └────────┘ │  │ └────────┘ │
       └────────────┘  └────────────┘  └────────────┘
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                     Semantic Cache
                    (Redis + embeddings)
                              │
                     Request Queue
                    (async, priority)
```

**Key decisions**: continuous batching, KV cache management (PagedAttention), quantization level, GPU count for tensor parallelism, semantic cache hit rate optimization.

📖 [AI Infrastructure](../topics/12-ai-infrastructure.md)

---

## Pattern 5: Evaluation Pipeline

**Use when**: any production LLM system that needs continuous quality monitoring.

```
┌─────────────────────────────────────────────────────────────────┐
│                    EVAL PIPELINE                                │
│                                                                 │
│  Production Traffic                                             │
│       │                                                         │
│       ├─── 5% sample ──▶ Evaluation Queue                      │
│       │                        │                               │
│       ▼                        ▼                               │
│  Serve Response          LLM-as-Judge                          │
│                          (quality score)                        │
│                                │                               │
│                         ┌──────┴──────┐                        │
│                         ▼             ▼                        │
│                    Score DB      Alert if score                 │
│                    (metrics,     drops below                    │
│                    dashboard)    threshold                      │
│                                                                 │
│  Golden Test Suite ──▶ Automated Eval (CI) ──▶ Block deploy    │
│  (versioned Q&A pairs)                         if regression   │
└─────────────────────────────────────────────────────────────────┘
```

**Key decisions**: sampling strategy (random, stratified, anomaly-triggered), judge model choice, metric thresholds, golden dataset maintenance.

📖 [Evaluation & Testing](../topics/09-evaluation-testing.md)

---

## Pattern 6: Fine-Tuning Pipeline

**Use when**: task-specific behavior that prompting can't achieve; cost reduction.

```
Raw Data
   │
   ▼
Data Curation
  - Quality filtering (LLM scorer)
  - Deduplication
  - Format standardization
  - Train/val split
   │
   ▼
Fine-Tuning Job (QLoRA)
  - Base model (frozen 4-bit)
  - LoRA adapter (trainable)
  - Monitor: loss curves, grad norms
   │
   ▼
Evaluation
  - Task benchmark vs. baseline
  - General capability regression check
  - Red team for new failure modes
   │
   ▼
Model Registry
  - Version the adapter
  - Tag with eval scores
   │
   ▼
Serving
  - Merge adapter + base (optional)
  - Or serve base + adapter separately
```

**Key decisions**: LoRA rank (8–64), target modules (q_proj, v_proj, o_proj typically), learning rate (1e-4 to 5e-5), epochs (1–3 typically).

📖 [Fine-Tuning deep dive](../topics/05-fine-tuning.md)

---

## Pattern 7: Streaming Response

**Use when**: user-facing chat where latency matters.

```
Client ──── HTTP/SSE ────▶ API Gateway
                                │
                                ▼
                          Token Streaming:
                          │ LLM generates tok1
                          │ ──stream──▶ Client renders "The"
                          │ LLM generates tok2
                          │ ──stream──▶ Client renders "The answer"
                          │ ...
                          │ LLM generates [EOS]
                          │ ──stream──▶ Client done

Client-side:
  - Render tokens as they arrive (markdown streaming)
  - Show "thinking..." indicator before first token
  - Handle stream interruption (user stops, server error)
```

**Key metrics**: TTFT (Time to First Token) — perceived latency. Target < 500ms.
**Pattern**: separate prefill (slow, processes all input tokens) from decode (streaming, one token at a time).

---

## Pattern 8: Model Router

**Use when**: cost optimization across a range of query difficulties.

```
User Query
    │
    ▼
Complexity Classifier (fast, cheap)
    │
    ├── Simple (score < 0.3) ──▶ GPT-4o-mini / Haiku
    │                             (~$0.001 per query)
    │
    ├── Medium (0.3–0.7) ──────▶ GPT-4o / Sonnet
    │                             (~$0.005 per query)
    │
    └── Complex (> 0.7) ────────▶ o3-mini / Reasoning Model
                                   (~$0.02–0.10 per query)
```

**Classifier features**: query length, presence of reasoning keywords, question type (factual vs. multi-step), past query patterns for this user.

**Typical outcome**: 60% simple, 30% medium, 10% complex → 60–80% cost reduction vs. always using top model.

---

## System Design Interview: Common Follow-up Questions

| Question | What They're Testing |
|----------|---------------------|
| "What if the model hallucinates?" | Guardrails, verification, faithfulness checks |
| "What if traffic spikes 10×?" | Auto-scaling, queue depth monitoring, graceful degradation |
| "What about data freshness?" | Incremental indexing, TTL on cached responses, freshness metadata |
| "How do you handle PII in user queries?" | Input screening, redaction, data retention policies |
| "How do you know if quality degrades?" | Eval pipeline, LLM-as-judge, user signals monitoring |
| "How do you handle multiple languages?" | Multilingual embeddings, language detection, language-specific models |
| "What's your disaster recovery plan?" | Multi-region, fallback models, static response cache |
| "How do you handle long documents?" | Chunking strategy, hierarchical summarization, map-reduce patterns |

---

## Key Numbers to Remember

| Metric | Typical Value |
|--------|--------------|
| Semantic cache hit rate | 20–40% for consumer apps |
| LLM-as-judge vs human agreement | 70–90% (depends on task) |
| LoRA training params vs full model | 0.1–1% |
| Continuous batching throughput gain | 5–10× over static batching |
| KV cache memory (128K context, 8B model) | ~16 GB |
| Re-ranker latency overhead | +50–200ms for top-50 candidates |
| RAG retrieval Recall@5 (good system) | >85% |
| Hallucination rate (GPT-4o, no RAG) | ~10–20% on factual tasks |

---

📖 Related: [AI System Design deep dive](../topics/07-ai-system-design.md) | [LLM Formulas](llm-formulas.md) | [RAG Checklist](rag-checklist.md)
