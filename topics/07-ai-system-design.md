# AI System Design

[← Vector Databases](06-vector-databases.md) | [LLMOps & Production →](08-llmops-production.md)

Full design walkthroughs for the most common AI system design interview questions.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [System Design Framework](#system-design-framework)
- [Design: Customer Support Chatbot](#design-customer-support-chatbot)
- [Design: Enterprise Document Q&A](#design-enterprise-document-qa)
- [Design: Content Moderation System](#design-content-moderation-system)
- [Design: Real-Time AI Coding Assistant](#design-real-time-ai-coding-assistant)
- [Design: Recommendation System with LLMs](#design-recommendation-system-with-llms)
- [Scalability & Tradeoffs](#scalability--tradeoffs)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## System Design Framework

**Q: What framework should you use for AI system design interviews?** `[B]`

Use **REQUIRE**:

1. **R — Requirements**: clarify functional (what it does) and non-functional (scale, latency, availability) requirements
2. **E — Estimation**: data volumes, QPS, storage, model inference budget
3. **Q — Quality Metrics**: what does success look like? Accuracy, latency P99, cost per query
4. **U — User Journey**: trace the end-to-end data flow for a single request
5. **I — Infrastructure**: data pipeline, model serving, storage, caching
6. **R — Reliability**: how do you handle failures, degradation, and edge cases
7. **E — Evaluation**: offline benchmarks, online metrics, monitoring

Always start with clarifying questions before drawing architecture:
- What's the scale? (queries/day, data size, user count)
- What's the latency budget? (streaming OK? P95 < 2s?)
- What's the accuracy requirement? (97% CSAT vs "best effort")
- Online vs batch? (real-time chat vs nightly reports)

---

**Q: RAG vs Fine-tuning vs In-context Learning — when to use which?** `[I]`

| | In-context Learning | RAG | Fine-tuning |
|---|---|---|---|
| **Knowledge source** | Model weights + context | External documents | Updated weights |
| **Knowledge freshness** | Static (training cutoff) | Real-time | Static (re-train to update) |
| **Personalization** | Limited (long prompt) | Dynamic retrieval | Static per version |
| **Latency** | Fast | Moderate (retrieval overhead) | Fastest (no retrieval) |
| **Cost** | High (large prompts) | Moderate | High upfront, low at inference |
| **When** | Few examples, quick prototyping | Large/dynamic knowledge base | Consistent format or style |

**Decision rule**: start with prompting → add RAG when knowledge is large or dynamic → add fine-tuning when consistent behavior or format is the primary requirement.

---

## Design: Customer Support Chatbot

**Q: Design an AI customer support chatbot for an e-commerce company handling 100K queries/day.** `[I]`

**Requirements clarification**:
- Real-time chat (streaming responses), P95 < 3s
- Must access order status, shipping, return policies
- Escalation to human agent when needed
- Multi-language support
- Compliance: no PII in LLM prompts; SOC 2

**Architecture**:

```
User Browser
    ↓ WebSocket
API Gateway (rate limiting, auth)
    ↓
Session Service (Redis — conversation history)
    ↓
Intent Router ──→ [Order Status] → Order DB API
                ↓ [Policy Q&A] → RAG Pipeline
                ↓ [Complaints] → Human Queue (Zendesk API)
                ↓ [General] → LLM with RAG
    ↓
LLM Service (streaming)
    ↓
Response Filter (PII masking, guardrails)
    ↓
User
```

**RAG pipeline**:
- Knowledge base: product docs, FAQs, return policies (Markdown → chunked → embedded)
- Vector DB: Pinecone with namespace per region/language
- Hybrid search: dense (semantic) + sparse (keyword) for product SKUs and order IDs

**PII protection**:
- Order ID / user email extracted from session auth token — never passed raw to LLM
- LLM only sees: "User with order status: SHIPPED, ETA: Dec 5". Never raw PII.

**Escalation logic**:
```python
if sentiment_score < threshold or confidence < 0.7 or "speak to agent" in query:
    route_to_human(session_id, conversation_summary)
```

**Monitoring**: CSAT score, deflection rate (% resolved without human), mean handle time, escalation rate, P95 latency.

---

## Design: Enterprise Document Q&A

**Q: Design a system that lets employees ask questions across 500K internal documents (PDFs, Word, Slack).** `[A]`

**Scale**: 500K docs, 50 avg chunks/doc = 25M chunks; 10K employees, 5K queries/day peak.

**Data ingestion pipeline**:
```
Document Sources (SharePoint, Slack, Confluence, GDrive)
    ↓ Connectors (incremental sync every 15min)
Document Processor:
    - Parse: PDF (PyMuPDF), DOCX (python-docx), Slack (API)
    - Extract metadata: author, date, department, access_control_list
    - Chunk: semantic chunking (512 tokens, 64 overlap)
    - Embed: text-embedding-3-large
    ↓
Vector DB (Qdrant with payload filters for access control)
    ↓
Search Index (Elasticsearch for keyword/metadata search)
```

**Access control** (critical):
- Store ACL (list of allowed user groups) as metadata per document chunk
- All queries filtered: `filter={"allowed_groups": {"contains": user.groups}}`
- Never return chunks the user's group is not in the ACL for

**Query pipeline**:
```
User query
    ↓ Query rewriting (LLM expands ambiguous queries)
    ↓ Hybrid search (semantic + keyword, filtered by user ACL)
    ↓ Re-ranking (cross-encoder)
    ↓ Context synthesis (LLM generates answer with citations)
    ↓ Source attribution (return doc name, section, link)
User sees: answer + "Source: [Document Name, Section 3.2]"
```

**Staleness handling**: each chunk stores doc_version and last_updated. Stale chunks are re-embedded during background sync. Show "last updated: 3 days ago" on sources.

---

## Design: Content Moderation System

**Q: Design an AI content moderation system for a platform with 1M posts/day.** `[A]`

**Requirements**: detect NSFW, hate speech, spam, misinformation; support text, images; < 500ms per post; 99.5% uptime.

**Multi-tier architecture** (balance speed vs accuracy):

```
Post submitted
    ↓
Tier 1: Fast blocking (< 10ms)
    - Keyword blocklist, regex patterns
    - Hash-match for known CSAM/illegal content (PhotoDNA)
    → if clear: pass to Tier 2
    → if flagged: block immediately + log

Tier 2: ML classifier (< 100ms)
    - Fine-tuned DistilBERT (text) or ViT (image)
    - High recall, accept some false positives
    → if score > 0.9: auto-reject + log
    → if score > 0.5: Tier 3 review queue
    → if score < 0.5: auto-approve + sample for audit

Tier 3: LLM review (async, < 10s)
    - GPT-4o / Claude for nuanced cases
    - Returns: decision + detailed reasoning
    → human review if LLM confidence < 0.7

Tier 4: Human review queue (async)
    - Prioritized by harm score
    - Decisions feed back as training data
```

**Handling scale**: 1M/day ≈ 12/sec average. Kafka for ingestion. Tier 1-2 inline (synchronous). Tier 3-4 async (post goes live with "under review" status, retroactively removed if violations found).

**Bias monitoring**: track false positive rate by demographic group; ensure system doesn't over-moderate minority dialects or languages.

---

## Design: Real-Time AI Coding Assistant

**Q: Design a VS Code AI coding assistant with code completion and chat.** `[A]`

**Two core features**:
1. **Inline completion**: triggered on keystroke, must return in <100ms
2. **Chat Q&A**: user asks questions about their code, returns in <5s

**Inline completion architecture**:
```
Keypress event → debounce (50ms) →
Context extraction:
    - 50 lines before cursor
    - 20 lines after cursor
    - File name, language (for system prompt)
    - Imported symbols (AST parsing)
    ↓
Request to completion endpoint (streaming)
    - Small, fast model (Codestral-7B, DeepSeek-Coder-1.3B)
    - FIM (Fill-in-the-Middle) format: <PRE>{context}<SUF>{suffix}<MID>
    ↓
Cache: check prefix/suffix hash → return cached if hit
    ↓
Ghost text displayed inline
```

**Chat architecture**:
- User selects code + types question
- Workspace indexer: AST-based code chunker → vector DB (Chroma local)
- Relevant code files retrieved by semantic search on user's question
- LLM receives: selected code + relevant context + question
- Larger, smarter model (Claude 3.5 Sonnet, GPT-4o)

**Latency optimization**:
- KV cache warming for system prompt (always same for inline completions)
- Speculative decoding for inline model
- Client-side debounce + cancellation (cancel in-flight request on next keypress)

---

## Design: Recommendation System with LLMs

**Q: When should you use LLMs in a recommendation system?** `[I]`

Traditional RecSys (collaborative filtering, matrix factorization) is fast and scales to billions of items. LLMs add value in specific scenarios:

**Use LLM for**:
- **Cold start**: no user history → LLM reasons from user profile/context to make recommendations
- **Explanation generation**: "You might like X because you enjoyed Y and both feature Z"
- **Conversational refinement**: "Show me something like the last book but shorter" → LLM interprets preference
- **Zero-shot cross-domain**: new catalog with no interaction history

**Don't use LLM as the core ranking model** for >1M items — too slow for real-time retrieval over large catalogs.

**Hybrid architecture**:
```
User query/profile
    ↓ Candidate generation (ANN over item embeddings, fast)
    ↓ Feature extraction (behavioral signals, contextual features)
    ↓ Traditional ranking model (GBM/DNN)
    ↓ LLM re-ranking (top 20 items → LLM ranks + explains top 5)
    ↓ Explanation generator (LLM)
```

---

## Scalability & Tradeoffs

**Q: How do you scale an LLM application from 100 to 10M users?** `[A]`

| Scale | Architecture |
|-------|-------------|
| **100 users (prototype)** | Single server, direct API calls, no caching, SQLite |
| **10K users (MVP)** | Auto-scaling API pods, Redis cache for embeddings/sessions, PostgreSQL, semantic cache for LLM responses |
| **1M users (growth)** | Multi-region deployment, CDN for static assets, async queue (Celery) for heavy workloads, dedicated vector DB cluster, rate limiting, circuit breakers |
| **10M users (scale)** | Global load balancing, sharded vector DB, LLM request batching, self-hosted fine-tuned models for cost reduction, tiered model routing (cheap models for simple queries) |

**Cost levers at scale**:
- Semantic caching (30–50% cache hit rate typical for repeat queries)
- Model routing (80% of queries to GPT-4o-mini, 20% to GPT-4o)
- Prompt compression (summarize history, remove irrelevant context)
- Batch embeddings during ingestion (vs one-by-one)

---

**Q: How do you design for LLM API failures?** `[I]`

LLM APIs have higher failure rates than traditional APIs: rate limits, model availability, high latency spikes.

**Resilience patterns**:
1. **Multi-provider failover**: if OpenAI fails, route to Anthropic; implement as a provider chain with health checks
2. **Circuit breaker**: after N consecutive failures, short-circuit for T seconds; serve cached/fallback response
3. **Exponential backoff with jitter**: retry with 2ˢ × random_jitter delay between attempts
4. **Timeout + degraded mode**: if model doesn't respond in 10s, return a partial cached answer or "I'm having trouble right now"
5. **Queue-based buffering**: for non-real-time tasks, put requests in a queue; process when LLM is available
6. **Rate limit budgeting**: distribute rate limit tokens across tenants; shed load from low-priority users first

---

## Troubleshooting Scenarios

**Q: Users are getting inconsistent answers to the same question. How do you debug this?** `[I]`

1. **Check temperature** — is temperature > 0? Non-determinism is by design. For factual consistency, set temperature = 0.
2. **Check context variability** — is the conversation history included? Different history lengths alter the model's context and therefore response.
3. **Check retrieval variability** — if using RAG, are different chunks being retrieved each time? Retrieval non-determinism propagates to generation. Fix: ensure deterministic chunking, stable vector index.
4. **Check prompt changes** — has the system prompt been modified? Even small changes can shift output significantly.
5. **Add output caching** — for high-confidence factual queries (same user, same question within session), cache the response.

---

**Q: Your AI system's P99 latency is 15 seconds, but requirement is 5 seconds. How do you optimize?** `[A]`

1. **Profile the pipeline** — add tracing (Langfuse, Arize) to find the slowest step. Is it LLM inference, retrieval, or embedding?
2. **Parallel retrieval** — if you run multiple searches sequentially, parallelize with `asyncio.gather()`
3. **Streaming response** — user perceives faster response even if total generation is the same; implement streaming and show tokens as they generate
4. **Smaller context** — reduce tokens in the prompt (shorter history, fewer retrieved chunks). Latency scales roughly linearly with prompt length.
5. **Smaller/faster model** — GPT-4o-mini or Claude Haiku are 5–10× faster than their larger siblings; often sufficient quality
6. **KV cache warming** — pre-populate KV cache for system prompts; first-token latency drops significantly
7. **vLLM / TGI for self-hosted** — continuous batching dramatically improves throughput and can reduce P99 latency

---

[← Vector Databases](06-vector-databases.md) | [LLMOps & Production →](08-llmops-production.md)
