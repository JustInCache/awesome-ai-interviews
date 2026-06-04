# AI Engineer Interview Question

## Topic: Retrieval-Augmented Generation (RAG) Quality

---

### Question

> Your RAG system retrieves relevant documents but the final answers are still wrong or incomplete. The retrieval metrics look good, but users complain the answers are bad. What's happening?

---

### Answer

This is the **retrieval-generation gap**—one of the trickiest problems in RAG systems. Good retrieval doesn't automatically mean good answers. The problem usually lies in what happens *after* retrieval.

---

#### Diagnostic Framework

I'd break down the RAG pipeline and check each stage:

```
Query → Retrieval → Context Assembly → Generation → Answer
          ✓              ?                 ?           ✗
```

If retrieval is good but answers are bad, the problem is downstream.

---

#### Common Failure Modes

**1. Retrieved but Not Used**

The model has the relevant information in context but doesn't use it. Why?

- **Context too long**: Important info buried among less relevant chunks
- **Conflicting information**: Model sees contradictory statements and picks wrong one
- **Instruction following failure**: Model relies on parametric knowledge instead of context

How to detect: Compare answers with and without retrieval. If they're similar, context isn't being used.

Fix: Better context ordering (most relevant first), explicit instructions ("Answer ONLY based on the provided documents"), chunk relevance scoring.

**2. Chunk Boundary Problems**

The answer spans multiple chunks, but each chunk alone is incomplete.

Example: User asks "What's the refund policy for premium members?"
- Chunk 1: "Refund policy: 30 days for standard members..."
- Chunk 2: "Premium members receive extended benefits including..."
- Neither chunk contains the complete answer.

Fix: Overlapping chunks, larger chunk sizes, hierarchical retrieval (document → section → paragraph), or query decomposition.

**3. Wrong Granularity**

Chunks contain the right topic but wrong level of detail.

Example: User asks for specific API parameters, retrieval returns high-level overview docs.

Fix: Multiple index levels (summaries vs. details), query classification to route to appropriate index.

**4. Synthesis Failure**

Information is spread across multiple documents that need to be combined, but the model fails to synthesize.

Example: "Compare feature X across product lines A and B"
- Retrieves docs about A and docs about B
- Model summarizes each separately instead of comparing

Fix: Explicit synthesis prompts, multi-hop reasoning chains, or pre-processing to create comparison structures.

**5. Hallucination Despite Context**

Model generates plausible-sounding content that contradicts or isn't supported by retrieved documents.

Fix: Citation requirements ("Quote the specific text that supports your answer"), lower temperature, verification step that checks answer against sources.

---

#### My Debugging Process

**Step 1: Error analysis**

Manually review 20-30 bad answers:
- What information was retrieved?
- Was the answer in the retrieved content?
- How did the model fail? (Ignored context? Wrong synthesis? Hallucinated?)

Categorize failures—different causes need different fixes.

**Step 2: Instrumentation**

Log intermediate steps:
- Retrieved chunks with scores
- Final prompt sent to model
- Model's "reasoning" if using chain-of-thought

This lets you pinpoint where the pipeline breaks.

**Step 3: Targeted fixes**

Based on failure categories:

| Failure Type | Fix |
|--------------|-----|
| Context not used | Better prompting, shorter context |
| Chunk boundaries | Larger chunks, overlap, hierarchical |
| Wrong granularity | Multiple indexes, query routing |
| Synthesis failure | Explicit reasoning steps |
| Hallucination | Citations, verification, lower temperature |

---

#### Evaluation Beyond Retrieval

Retrieval metrics (recall@k, MRR) aren't enough. Add:

- **Answer correctness**: Is the final answer right? (Requires human eval or reference answers)
- **Faithfulness**: Is the answer supported by retrieved context? (Can be automated)
- **Completeness**: Does the answer address all parts of the question?

---

### What This Question Tests

- Understanding of end-to-end RAG systems
- Systematic debugging approach
- Knowledge of RAG-specific failure modes
- Ability to instrument and measure complex pipelines
