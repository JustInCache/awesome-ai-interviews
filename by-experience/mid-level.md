# Mid-Level AI Engineer Study Path

[← Junior](junior.md) | [Senior/Staff →](senior-staff.md)

**For**: 2–5 years of experience · Production AI system builders · SDE-II to Senior-II transition

---

## What Interviewers Expect at Mid-Level

Mid-level AI interviews test **production depth and end-to-end ownership**. You're expected to:
- Design and implement complete AI pipelines (RAG, agents, fine-tuning)
- Diagnose and fix production issues independently
- Make and defend architectural trade-off decisions
- Measure and improve system quality with evaluation frameworks
- Understand the "why" behind architectural choices, not just the "what"

You are expected to have opinions and defend them.

---

## Core Topics to Master

### 1. Advanced RAG Patterns

Go beyond basic RAG. Know every technique in the table below and when to use each:

| Problem | Solution |
|---------|---------|
| Retrieval misses relevant chunks | Hybrid search (dense + BM25 + RRF) |
| Chunks lack context | Parent-child chunking |
| Simple query misses complex intent | Query decomposition / HyDE |
| Irrelevant chunks still in top-k | Cross-encoder re-ranking |
| Answer contradicts retrieved context | Faithfulness guardrail |
| RAG for structured data | Text-to-SQL |
| Multi-hop reasoning required | GraphRAG / Agentic RAG |

**Study**: [RAG — All Sections](../topics/03-rag.md)

**Key questions**:
- Walk me through a complete RAG pipeline from document ingestion to answer generation.
- Your RAG system has good retrieval but still gives wrong answers. How do you debug it?
- When would you use GraphRAG over standard vector RAG?

---

### 2. AI Agents

Full production agents, not just ReAct demos:

- [ ] ReAct, Plan-and-Execute, Reflexion patterns
- [ ] Memory types: working, episodic, semantic
- [ ] Tool design principles and MCP protocol
- [ ] Multi-agent orchestration (LangGraph, orchestrator pattern)
- [ ] Agent safety: irreversible actions, HITL, loop detection
- [ ] Agent evaluation: trajectory eval, environment simulation

**Study**: [AI Agents — All Sections](../topics/04-ai-agents.md)

**Key questions**:
- How do you prevent an agent from taking irreversible actions?
- Your agent is stuck in a loop. How do you detect and break it?
- Design a multi-agent system for automated code review.

---

### 3. Fine-Tuning

Know when and how to fine-tune:

- [ ] LoRA: rank, alpha, target modules, merge
- [ ] QLoRA: 4-bit quantization + LoRA on consumer GPUs
- [ ] SFT vs. DPO vs. RLHF — when to use which
- [ ] Dataset preparation: format, quality, size guidelines
- [ ] Catastrophic forgetting and how LoRA prevents it
- [ ] Evaluating fine-tuned models: task-specific + general capability preservation

**Study**: [Fine-Tuning — All Sections](../topics/05-fine-tuning.md)

**Key questions**:
- When should you fine-tune instead of prompting?
- Explain LoRA. How is it different from full fine-tuning?
- Your fine-tuned model forgot how to do math. What went wrong?

---

### 4. LLMOps & Evaluation

Production ownership requires understanding the full lifecycle:

- [ ] Prompt versioning and CI/CD for prompts
- [ ] LLM tracing: Langfuse, Arize — what to capture in traces
- [ ] A/B testing LLM features: metrics, sample sizes, gotchas
- [ ] Semantic caching and cost optimization strategies
- [ ] Building a regression evaluation suite
- [ ] LLM-as-judge: setup, calibration, bias mitigation
- [ ] RAGAS metrics: faithfulness, answer relevancy, context precision/recall

**Study**: [LLMOps](../topics/08-llmops-production.md) | [Evaluation & Testing](../topics/09-evaluation-testing.md)

**Key questions**:
- How do you ensure a prompt change doesn't degrade quality?
- Walk me through your approach to monitoring a production LLM system.
- Your LLM cost tripled this week. How do you investigate?

---

### 5. Vector Databases (Advanced)

- [ ] HNSW vs. IVF-PQ trade-offs
- [ ] Multi-tenant architecture patterns
- [ ] Quantization: scalar (INT8), binary, product quantization
- [ ] Embedding drift and migration strategies
- [ ] Choosing and benchmarking embedding models

**Study**: [Vector Databases — All Sections](../topics/06-vector-databases.md)

---

## Top 30 Mid-Level Questions

1. Design a RAG pipeline for 500K enterprise documents with access control.
2. Your RAG system retrieves relevant chunks but still gives wrong answers. Debug it.
3. What is hybrid search? How does Reciprocal Rank Fusion work?
4. What is a cross-encoder reranker? When would you add one to RAG?
5. Explain LoRA and how it differs from full fine-tuning.
6. When should you fine-tune vs. use RAG vs. use prompting?
7. What is DPO? How does it compare to RLHF?
8. What is catastrophic forgetting? How does LoRA help?
9. What is PagedAttention and how does vLLM use it?
10. What is continuous batching? Why does it improve GPU utilization?
11. How do you version prompts in production?
12. How do you implement A/B testing for an LLM feature?
13. What is semantic caching? What are its pitfalls?
14. What is LLM tracing? What should you capture in a trace?
15. What is RAGAS? Explain its four core metrics.
16. What is LLM-as-judge? What are its biases?
17. How do you build a regression evaluation suite?
18. What is a guardrail in LLM applications? Name 3 types.
19. How do you detect and prevent prompt injection?
20. What is the ReAct agent pattern? Trace through an example.
21. What memory types does an AI agent use?
22. How do you prevent an agent from taking harmful irreversible actions?
23. What is multi-agent coordination? What are the main topologies?
24. What is HNSW? Why is it the dominant ANN algorithm?
25. How do you handle embedding model migration?
26. What is multi-tenant vector search? How do you enforce access control?
27. Your P99 latency spiked from 2s to 15s. How do you debug?
28. How do you reduce LLM API costs by 50%?
29. What is KV cache? How does PagedAttention improve KV cache efficiency?
30. What is speculative decoding?

---

## System Design Practice

At mid-level, you should be able to design complete AI systems. Practice these:

1. **Customer support chatbot** — handle 100K queries/day, multi-language, PII-safe
2. **Enterprise document Q&A** — 500K docs, access control, staleness handling
3. **AI coding assistant** — inline completion + chat, <100ms completion latency
4. **Content moderation** — 1M posts/day, text + images, multi-tier architecture

**Study**: [AI System Design](../topics/07-ai-system-design.md)

**Approach every design with**:
1. Clarify requirements and scale
2. Estimate storage, QPS, token budget
3. Draw the data flow end-to-end
4. Identify failure modes and how you'd handle them
5. Discuss monitoring and evaluation

---

## Resources

- 📖 [All topics pages](../README.md#topics)
- 📖 [AI Engineer role questions](../roles/ai-engineer.md)
- 🛠️ Build: a production-grade RAG system with hybrid search and evaluation
- 🛠️ Build: a LoRA fine-tune of a 7B model on domain-specific data
- 🛠️ Instrument: add Langfuse tracing to an existing LLM application

---

[← Junior](junior.md) | [Senior/Staff →](senior-staff.md)
