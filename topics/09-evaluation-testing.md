# Evaluation & Testing

[← LLMOps & Production](08-llmops-production.md) | [AI Safety & Ethics →](10-ai-safety-ethics.md)

25+ questions on evaluating LLM systems, from automated metrics to human evaluation.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Evaluation Fundamentals](#evaluation-fundamentals)
- [Automated Metrics](#automated-metrics)
- [LLM-as-Judge](#llm-as-judge)
- [RAG Evaluation](#rag-evaluation)
- [Evaluation Frameworks](#evaluation-frameworks)
- [Red Teaming & Safety Evaluation](#red-teaming--safety-evaluation)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Evaluation Fundamentals

**Q: Why is evaluating LLM applications hard?** `[B]`

Traditional ML evaluation: compare model prediction to ground truth label → compute accuracy/F1. Simple and deterministic.

LLM evaluation challenges:
1. **No single ground truth**: "How are you?" has hundreds of acceptable responses
2. **Subjective quality**: helpfulness, tone, creativity are human judgments
3. **Non-determinism**: same prompt → different response each time
4. **Multidimensional**: a response can be factually correct but unhelpful, or helpful but verbose
5. **Task diversity**: evaluation for a customer support bot ≠ evaluation for a coding assistant

**Result**: LLM evaluation requires a combination of automated metrics, LLM-as-judge, and human evaluation. No single metric captures full quality.

---

**Q: What is the evaluation hierarchy for LLM systems?** `[I]`

From cheapest/fastest to most reliable:

| Level | Method | Speed | Cost | Reliability |
|-------|--------|-------|------|-------------|
| 1 | Unit tests (output format, schema validation) | Instant | Free | High for structure |
| 2 | Automated metrics (BLEU, ROUGE, BERTScore) | Fast | Free | Low-medium for quality |
| 3 | LLM-as-judge | Fast | Moderate | Medium-high |
| 4 | Human evaluation | Slow | High | High |
| 5 | Online metrics (user feedback, CSAT) | Delayed | Operational cost | Highest (real-world) |

Use all levels together: automated metrics for regression detection, LLM-as-judge for comprehensive offline evaluation, human evaluation for calibration, and online metrics as the ultimate signal.

---

## Automated Metrics

**Q: What is BLEU and when should (not) you use it?** `[B]`

**BLEU (Bilingual Evaluation Understudy)**: measures n-gram overlap between model output and reference text.

$$\text{BLEU} = BP \cdot \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)$$

Where BP = brevity penalty, pₙ = n-gram precision.

**When to use**: machine translation (comparing translation to professional reference), structured text generation with single correct forms.

**When NOT to use**: open-ended generation, QA with multiple acceptable answers, chatbot responses. BLEU penalizes paraphrases that are semantically equivalent but lexically different — a perfect answer not using the same words scores near 0.

---

**Q: What is ROUGE and how does it differ from BLEU?** `[B]`

**ROUGE (Recall-Oriented Understudy for Gisting Evaluation)**: measures n-gram overlap from a recall perspective.

- **ROUGE-N**: recall of n-grams from reference that appear in the model output
- **ROUGE-L**: longest common subsequence — captures sentence-level structure
- **ROUGE-1**: unigram overlap (word-level)
- **ROUGE-2**: bigram overlap (phrase-level)

| Metric | Orientation | Best for |
|--------|-------------|----------|
| BLEU | Precision | Translation quality |
| ROUGE | Recall | Summarization (coverage) |
| F1 | Balanced | When both coverage and precision matter |

**Limitation**: same as BLEU — no semantic understanding; paraphrases fail.

---

**Q: What is BERTScore?** `[I]`

BERTScore (Zhang et al., 2019) computes token-level similarity using contextual embeddings rather than n-gram overlap:

1. Embed each token in reference and candidate using BERT
2. For each candidate token, find the most similar reference token (soft alignment)
3. Compute precision, recall, F1 over these soft alignments

**Advantages over BLEU/ROUGE**:
- Captures semantic similarity — paraphrases score high
- Handles synonyms and word order flexibility
- Correlates better with human judgments than BLEU

**Disadvantage**: slower (requires BERT forward pass); more opaque; can still be gamed.

---

**Q: What are model-based metrics and what are their limitations?** `[I]`

Model-based metrics use a trained model (not string matching) to score LLM outputs:

- **G-Eval** (Liu et al., 2023): GPT-4 evaluates responses on a rubric (coherence, fluency, consistency) using chain-of-thought reasoning + probability scoring
- **BARTScore**: BART model scores responses for faithfulness
- **Perplexity**: how surprised the base model is by the output — low perplexity ≈ fluent

**Limitations**:
- **Model bias**: GPT-4-as-judge favors GPT-4 outputs (verbosity bias, style preferences)
- **Position bias**: LLMs rate the first response higher in pairwise comparisons
- **Cost**: using GPT-4 for every eval response adds up
- **Calibration**: a score of 7/10 from one judge doesn't mean the same as 7/10 from another

---

## LLM-as-Judge

**Q: What is LLM-as-judge and how do you set it up correctly?** `[I]`

LLM-as-judge uses a capable LLM (GPT-4, Claude) to evaluate the output of another LLM according to a rubric.

**Best practices for high-quality LLM evaluation**:

1. **Detailed rubric**: define criteria precisely. Don't say "helpful" — say "Does the response directly address the user's question without requiring follow-up? (0=No, 1=Partially, 2=Yes)"

2. **Chain-of-thought reasoning before score**: require the judge to explain its reasoning before giving a score — reduces random scoring, improves consistency

3. **Calibration examples**: provide few-shot examples of responses and their scores with reasoning

4. **Pairwise comparison over absolute scoring**: "Is response A or response B better for this query?" is more reliable than "Rate response A 1-10"

5. **Reduce position bias**: if doing pairwise, run A vs B and B vs A; average results

6. **Use multiple judges**: ensemble scores from 3 judges; flag when judges disagree significantly

---

**Q: How do you build a regression suite to prevent quality regressions?** `[I]`

A regression suite ensures that changes to prompts, models, or retrieval don't degrade quality.

**Building the suite**:
1. Collect 200–500 representative real queries (or synthetic if no live traffic yet)
2. For each query, define an evaluation rubric (not a single expected answer — a rubric the judge uses to score)
3. Get baseline scores by running current production system through the eval
4. Set regression thresholds: alert if overall score drops >2%, or if any specific category drops >5%

**In CI/CD pipeline**:
```
PR opened → eval suite runs → compare to baseline scores
PR shows: "Helpfulness: +1.3%, Accuracy: -0.4%, Latency: +200ms"
Merge only if no metric drops below threshold
```

**Key insight**: a regression suite with even 100 queries catches most regressions before they reach production.

---

## RAG Evaluation

**Q: What is RAGAS and what metrics does it provide?** `[I]`

RAGAS (Retrieval-Augmented Generation Assessment) is a framework for evaluating RAG pipelines end-to-end.

**Core metrics**:

| Metric | Measures | How |
|--------|---------|-----|
| **Faithfulness** | Is the answer grounded in the context? | LLM checks if each claim in the answer can be traced to retrieved context |
| **Answer Relevancy** | Does the answer address the question? | LLM generates questions from the answer; compare to original |
| **Context Precision** | Are retrieved chunks relevant? | Fraction of retrieved chunks that are relevant to the question |
| **Context Recall** | Are all relevant chunks retrieved? | Fraction of ground-truth answer components that appear in retrieved context |

**Composite score**: RAGAS score = harmonic mean of all four metrics.

**Usage**:
```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision

results = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevancy, context_precision]
)
```

---

**Q: How do you evaluate retrieval quality in a RAG system?** `[I]`

**Offline retrieval evaluation**:
1. **Build eval dataset**: 100–200 (question, ground_truth_answer, relevant_doc_ids) triples
2. **Metrics**:
   - **Recall@k**: fraction of relevant docs in top-k retrieved
   - **Precision@k**: fraction of top-k retrieved that are relevant
   - **MRR (Mean Reciprocal Rank)**: reciprocal of rank of first relevant doc
   - **NDCG**: normalized discounted cumulative gain — accounts for rank order

**Online retrieval evaluation**:
- Track click-through on citations — do users click cited sources? (proxy for retrieval relevance)
- Track "no useful results" feedback
- Monitor answer faithfulness (poor faithfulness → poor retrieval or chunk quality)

---

## Evaluation Frameworks

**Q: What are the major LLM evaluation frameworks?** `[I]`

| Framework | Strengths | Best For |
|-----------|-----------|---------|
| **RAGAS** | RAG-specific metrics, no labeled data needed | RAG systems |
| **LangSmith** | LangChain native, tracing + evals integrated | LangChain apps |
| **Langfuse** | Open source, LLM-agnostic, human annotation UI | Production eval + human feedback |
| **Arize AI** | Enterprise MLOps + LLM observability | Large-scale prod monitoring |
| **EleutherAI lm-evaluation-harness** | Academic benchmarks, reproducible | Base model evaluation |
| **DeepEval** | pytest-like, unit test style | CI/CD integration |
| **OpenAI Evals** | OpenAI-native, eval templates | OpenAI models |

For most teams: **Langfuse** (open source, free, flexible) + **RAGAS** (for RAG evaluation) is a strong combination.

---

## Red Teaming & Safety Evaluation

**Q: What is red teaming for LLM systems?** `[I]`

Red teaming systematically tests an LLM system for safety vulnerabilities and harmful outputs by adversarially probing it.

**Red teaming dimensions**:
1. **Jailbreaks**: prompts designed to bypass safety training ("DAN", "roleplay as evil AI", etc.)
2. **Prompt injection**: injecting instructions through untrusted content
3. **Bias and stereotyping**: demographic bias, racial/gender stereotyping
4. **Privacy violations**: leakage of training data, PII extraction
5. **Hallucination induction**: questions designed to elicit confident wrong answers
6. **Misuse**: using the system for harmful purposes (harm instruction generation, CSAM, etc.)

**Automated vs manual**:
- Manual red teaming: security researchers with creative adversarial thinking
- Automated: use an LLM (attacker model) to generate adversarial prompts against the target model
- Hybrid: automated generation of candidates + human selection and refinement

---

**Q: What is a golden dataset and how do you build one?** `[I]`

A golden dataset is a curated, high-quality evaluation dataset used as a stable benchmark for a system.

**For RAG systems**: (question, context, ground_truth_answer) triples where context = the chunks that were manually verified to be relevant and answer = human-written ideal response.

**Building the golden dataset**:
1. Collect 200–500 real queries from production logs or user research
2. Stratify by query type, difficulty, and topic
3. For each query: retrieve the ideal answer from source documents; write it as a human would
4. Quality review by domain expert
5. Version and maintain the dataset — update as product evolves

**Pitfalls**:
- Train/eval contamination: don't include golden dataset queries in the retrieval index being tested
- Staleness: golden datasets go stale as product changes — schedule quarterly reviews
- Narrow coverage: ensure coverage of edge cases and failure modes, not just easy examples

---

## Troubleshooting Scenarios

**Q: Your LLM-as-judge scores are inconsistent across runs. How do you stabilize them?** `[I]`

1. **Set temperature=0** for the judge model — eliminates randomness in scoring
2. **Require chain-of-thought reasoning** before the score — anchors the judge's evaluation process
3. **Structured output**: enforce JSON output schema `{"reasoning": "...", "score": 1-5}` — prevents parsing failures
4. **Calibration examples**: add 3–5 scored examples to the judge prompt showing the expected scoring behavior
5. **Ensemble judges**: run each example through 3 different judge calls (or 3 different models); take the median score; flag if standard deviation > 1

---

**Q: Your eval suite shows great offline scores but users are unhappy in production. What's wrong?** `[I]`

This is a classic **evaluation-production distribution mismatch**:

1. **Eval dataset is not representative**: collected from cherry-picked examples, not real user queries. Real queries are messier, more ambiguous.
2. **Metric doesn't capture what users care about**: LLM-judge scoring accuracy; users care about helpfulness and brevity. Different things.
3. **Context mismatch**: eval runs queries in isolation; in production, users have multi-turn conversations, and context matters.
4. **Recency**: eval dataset is stale; user queries have evolved (new product features, new terminology).

**Fix**:
- Continuously sample real production queries to refresh the eval dataset
- Add user-feedback-correlated metrics (thumbs-up rate, follow-up question rate) to eval suite
- Add multi-turn conversation eval scenarios

---

[← LLMOps & Production](08-llmops-production.md) | [AI Safety & Ethics →](10-ai-safety-ethics.md)
