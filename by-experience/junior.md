# Junior AI Engineer Study Path

[← Back to Home](../README.md) | [Mid-Level →](mid-level.md)

**For**: 0–2 years of experience · Recent graduates · Career switchers entering AI

---

## What Interviewers Expect at Junior Level

Junior AI interviews test **foundational understanding**, not years of production experience. You're expected to:
- Explain core concepts clearly (attention, embeddings, RAG, fine-tuning)
- Write clean Python code for ML tasks
- Show curiosity and learning mindset
- Demonstrate you can build and ship simple AI applications

You are **not** expected to have designed multi-region architectures or led teams. Focus on depth of fundamentals and ability to reason through problems.

---

## 8-Week Study Plan

### Weeks 1–2: LLM Fundamentals

**Must-know concepts**:
- [ ] Transformer architecture (encoder, decoder, encoder-decoder)
- [ ] What is attention and why does it matter?
- [ ] Tokenization (BPE, WordPiece) and why it matters
- [ ] What are embeddings?
- [ ] Temperature, top-p, top-k — what do they control?
- [ ] Context window — what is it, what are its limits?
- [ ] What is RLHF at a high level?

**Study**: [LLM Fundamentals →](../topics/01-llm-fundamentals.md)

**Practice questions**:
1. Explain self-attention in plain English to a software engineer who doesn't know ML.
2. What happens when a conversation exceeds the context window?
3. Why does temperature = 0 produce deterministic outputs?
4. What's the difference between GPT (decoder-only) and BERT (encoder-only)?

---

### Weeks 3–4: Prompt Engineering & RAG

**Must-know concepts**:
- [ ] Zero-shot, few-shot, chain-of-thought prompting
- [ ] What is RAG and why is it better than fine-tuning for factual knowledge?
- [ ] Chunking: why do we chunk documents?
- [ ] Vector embeddings and cosine similarity
- [ ] Basic retrieval pipeline: embed → store → query → retrieve → generate

**Study**: [Prompt Engineering →](../topics/02-prompt-engineering.md) | [RAG →](../topics/03-rag.md)

**Build this**: a simple RAG chatbot that answers questions over a PDF. Use LangChain or LlamaIndex + Chroma + OpenAI. This is the most common junior interview project.

```python
# Minimal RAG in ~30 lines
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

loader = PyPDFLoader("document.pdf")
docs = loader.load()
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(docs)

vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-4o-mini"),
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5})
)
answer = qa_chain.invoke("What is the main finding?")
```

---

### Weeks 5–6: Embeddings & Vector Databases

**Must-know concepts**:
- [ ] Why do we need vector databases?
- [ ] How does ANN (approximate nearest neighbor) search work?
- [ ] Cosine similarity vs. dot product
- [ ] Popular embedding models (text-embedding-3-small, BGE)
- [ ] Basic HNSW indexing intuition

**Study**: [Vector Databases →](../topics/06-vector-databases.md)

**Practice questions**:
1. You have 1 million product descriptions. How do you enable semantic search?
2. A user searches for "cheap flights" — why might keyword search fail and semantic search succeed?
3. What's the trade-off between higher-dimensional embeddings and search speed?

---

### Weeks 7–8: Evaluation & LLMOps Basics

**Must-know concepts**:
- [ ] Why is evaluating LLMs harder than evaluating traditional ML?
- [ ] What is LLM-as-judge?
- [ ] What is RAGAS and what does it measure?
- [ ] Basic observability: what to log in an LLM application
- [ ] Semantic caching concept

**Study**: [Evaluation & Testing →](../topics/09-evaluation-testing.md) | [LLMOps Basics →](../topics/08-llmops-production.md)

---

## Top 20 Junior Questions — With Answers

These are the questions most likely to appear at your level. Read the question, try to answer yourself, then expand for the model answer.

<details>
<summary><strong>1. Explain the Transformer architecture. What is self-attention?</strong></summary>

The Transformer processes sequences in parallel using **self-attention** — a mechanism that allows each token to look at all other tokens in the sequence to build contextual representations.

Core components of a decoder-only Transformer (like GPT/LLaMA):
- **Token embeddings**: convert token IDs to dense vectors
- **Self-attention**: each token attends to all previous tokens; produces contextual representation
- **Feed-forward network**: non-linear transformation applied to each position independently
- **Residual connections + RMSNorm**: stabilize training
- **Causal mask**: prevents attending to future tokens

Self-attention is "self" because the Q, K, V matrices all come from the same input sequence — the sequence is attending to itself. Formula: `Attention(Q, K, V) = softmax(QK^T / √d_k) V`.

📖 [LLM Fundamentals →](../topics/01-llm-fundamentals.md)
</details>

<details>
<summary><strong>2. What is RAG? When would you use it vs. fine-tuning?</strong></summary>

RAG (Retrieval-Augmented Generation) retrieves relevant documents from an external knowledge base and includes them in the LLM prompt at query time, grounding the response in external facts.

**Use RAG when**: you need factual, up-to-date, or private information; you want citable sources; knowledge changes frequently.

**Use fine-tuning when**: you need a specific output style/format/behavior that prompting can't reliably produce; not for knowledge injection.

Example: "Answer questions about our company's HR policies" → RAG (policies change; need to cite exact policy). "Write emails in our brand voice" → fine-tuning (style, not facts).
</details>

<details>
<summary><strong>3. What are embeddings? How are they used for semantic search?</strong></summary>

Embeddings are dense numerical vectors that represent text (or images, audio) in a high-dimensional space where **semantically similar inputs are close together geometrically**.

For semantic search:
1. Embed all documents and store in a vector database
2. When a query arrives, embed the query with the same model
3. Find the k document vectors closest to the query vector (nearest neighbor search)
4. Return those documents as search results

This finds documents similar in *meaning*, not just keyword matches. "What are the opening hours?" will match "We're open 9am to 5pm" even with no keyword overlap.
</details>

<details>
<summary><strong>4. What is prompt engineering? Give an example of few-shot prompting.</strong></summary>

Prompt engineering is designing inputs to LLMs to reliably produce desired outputs without modifying model weights.

**Few-shot prompting** includes examples in the prompt:

```
Classify the sentiment of the following tweets as Positive, Negative, or Neutral.

Tweet: "Love this product, works perfectly!" → Positive
Tweet: "Absolutely terrible, broke after 2 days" → Negative
Tweet: "Just received my package" → Neutral

Tweet: "This is the best thing I've bought this year" → 
```

The model learns the format and classification criteria from examples, then applies to the new input. More reliable than zero-shot for consistent formatting.
</details>

<details>
<summary><strong>5. What is a context window? What happens when it's exceeded?</strong></summary>

The context window is the maximum number of tokens an LLM can process at once — both input and output combined. It's the model's "working memory."

When exceeded:
- Most implementations **truncate** the oldest tokens (sliding window) — the model loses information from early in the conversation
- Some implementations throw an error
- Quality degrades even before the hard limit (models pay less attention to middle-of-context information)

Practical implication: for a chatbot with a 128K context, a conversation of ~90K tokens will start losing early messages. Solutions: summarize older turns, use vector memory to store important facts.
</details>

<details>
<summary><strong>6. Explain tokenization. Why can't LLMs process raw characters?</strong></summary>

LLMs work with discrete token IDs from a fixed vocabulary. They can't process raw characters because:
- Pure character-level: sequences become very long (slow attention) and letters lose meaning
- Pure word-level: vocabulary explodes (100K+ words) and handles unknown words poorly

**BPE tokenization** (dominant approach):
1. Start with character vocabulary
2. Iteratively merge the most frequent adjacent pair
3. Result: common words are single tokens; rare words split into subword pieces

"tokenization" → ["token", "ization"] | "ChatGPT" → ["Chat", "G", "PT"]

Tokenization is why LLMs can struggle with character-level tasks ("count the letters in 'strawberry'") — they never see individual letters, only subword tokens.
</details>

<details>
<summary><strong>7. What is cosine similarity? Why is it used for vector search?</strong></summary>

Cosine similarity measures the **angle** between two vectors, regardless of their magnitude:

$$\cos(\theta) = \frac{A \cdot B}{|A||B|}$$

Range: -1 (opposite) to 1 (identical direction).

Why cosine for text embeddings:
- Text embeddings encode *meaning* in the direction of vectors, not their length
- A long document and its one-sentence summary should be similar — cosine handles this; Euclidean distance would give them different distances due to length
- Most embedding models L2-normalize vectors, so cosine = dot product (very fast to compute)
</details>

<details>
<summary><strong>8. What is HNSW? Why use approximate search instead of exact search?</strong></summary>

**HNSW (Hierarchical Navigable Small World)** is a graph-based algorithm for approximate nearest neighbor (ANN) search. It builds a multi-layer graph where the top layer is sparse (for fast navigation) and the bottom layer is dense (for accurate results).

Why approximate instead of exact:
- **Exact search**: compare query to every vector — O(N × d) time. At 10M vectors with 1536 dims, that's 15 billion multiplications per query. Too slow.
- **HNSW approximate**: O(log N) query time with >99% recall — finds the *correct* nearest neighbors almost all the time, just not guaranteed.

For RAG: if HNSW retrieves 49 out of 50 relevant documents, that's good enough. The 2% of misses are not worth the 1000× slowdown of exact search.
</details>

<details>
<summary><strong>9. What is chain-of-thought (CoT) prompting? When does it help?</strong></summary>

CoT prompting adds "Let's think step by step" (or few-shot reasoning examples) to the prompt, encouraging the model to produce intermediate reasoning steps before the final answer.

**When it helps**: multi-step reasoning tasks — math word problems, logic puzzles, code debugging, complex decision-making.

**When it doesn't help**: simple factual recall ("What's the capital of France?"), creative tasks, or very short answers. CoT adds latency and tokens, so only use when quality matters.

Example:
```
Q: If a store sells 12 apples at $0.50 each, and I buy 5, how much change from $10?

Without CoT: "$7.50"
With CoT: "12 apples × $0.50 = total stock value. I buy 5 apples × $0.50 = $2.50. 
           Change from $10 = $10 - $2.50 = $7.50"
```
Both get the right answer here, but CoT reduces errors significantly on harder problems.
</details>

<details>
<summary><strong>10. What is LLM temperature? What value would you use for a factual chatbot?</strong></summary>

Temperature scales the logits before softmax sampling. Lower = more deterministic, higher = more random:

- **temperature = 0**: always pick the highest-probability token (greedy, fully deterministic)
- **temperature = 0.7**: moderate randomness, natural-sounding text
- **temperature = 1.0**: standard sampling
- **temperature > 1.5**: very creative/random

For a **factual chatbot**: temperature = 0 to 0.1. You want consistent, deterministic answers. A user asking "What's the return policy?" should get the same answer every time.

For **creative writing**: temperature 0.7–1.0 produces more varied, interesting outputs.
</details>

<details>
<summary><strong>11. What is hallucination in LLMs? How can you reduce it?</strong></summary>

Hallucination: the model generates fluent, confident-sounding text that is factually incorrect or unsupported. It happens because LLMs are trained to produce plausible text, not to reason from verified facts.

**Mitigation strategies**:
1. **RAG**: ground responses in retrieved documents; "answer only based on the provided context"
2. **Lower temperature**: less random = less creative = fewer invented facts
3. **Prompt constraints**: "Say 'I don't know' if you're unsure"
4. **Structured output**: force citations — each claim must reference a source
5. **Verification step**: second LLM call fact-checks the first answer against source

No technique eliminates hallucinations entirely. For high-stakes domains (medical, legal), always combine multiple mitigations.
</details>

<details>
<summary><strong>12. What is the difference between GPT (decoder-only) and BERT (encoder-only)?</strong></summary>

| | GPT / LLaMA (decoder-only) | BERT (encoder-only) |
|--|--------------------------|---------------------|
| Attention | Causal (can only see past tokens) | Bidirectional (sees all tokens) |
| Training | Next-token prediction | Masked language modeling |
| Best for | Text generation, chat | Classification, embeddings, NER |
| Output | Generated text | Representations / classifications |

**Intuition**: BERT "reads" the whole sentence at once to understand it deeply — great for understanding. GPT generates one token at a time, conditioning only on what came before — great for generation.

For embedding-based search, BERT-style models (BGE, E5) are better. For chat/generation, decoder-only (GPT, LLaMA, Claude) is the standard.
</details>

<details>
<summary><strong>13. Explain RLHF in simple terms.</strong></summary>

RLHF (Reinforcement Learning from Human Feedback) is how chatbots like ChatGPT are trained to be helpful and safe, not just to predict text.

**Three-step process**:
1. **SFT**: fine-tune the base model on examples of humans writing good responses to instructions
2. **Reward model**: collect human comparisons (response A vs B — which is better?) → train a model to predict human preference scores
3. **RL training**: use the reward model as a training signal — reinforce responses the reward model scores highly, penalize low-scoring responses

**Why it matters**: a base LLM trained only on next-token prediction has no concept of "being helpful." RLHF teaches the model to produce responses that humans actually prefer — concise, accurate, appropriate, not harmful.

DPO (Direct Preference Optimization) achieves similar results without the RL complexity and is now the dominant approach.
</details>

<details>
<summary><strong>14. What is chunking in RAG? What chunk size would you start with?</strong></summary>

Chunking splits large documents into smaller pieces for embedding and retrieval. You can't embed a 50-page PDF as one unit because:
- Embedding quality degrades with very long text
- You want to retrieve the *relevant portion*, not the whole document
- A single embedding can only capture one main topic

**Start with**: 512 tokens with 10–15% overlap (e.g., 50 tokens). The overlap prevents cutting a key concept at a chunk boundary.

**Adjust based on**:
- Short, dense technical docs → smaller chunks (256)
- Long narrative/prose documents → larger chunks (1024)
- "Answers are too incomplete" → increase chunk size
- "Too many irrelevant chunks retrieved" → decrease chunk size

Measure: check retrieval recall on a set of known Q&A pairs. If the relevant text keeps getting split across two chunks, increase overlap or chunk size.
</details>

<details>
<summary><strong>15. What is hybrid search? When is it better than pure semantic search?</strong></summary>

Hybrid search combines **BM25 keyword search** (sparse) with **vector semantic search** (dense) and merges results.

**BM25 excels at**: exact terms, proper nouns, codes, abbreviations, rare technical terms ("RFC 7231", "HIPAA Section 164.310").

**Vector search excels at**: meaning, synonyms, paraphrases, concept-matching.

Hybrid wins when users search with a mix of patterns. In practice:
- "What is contrastive learning?" → semantic search handles this well
- "CUDA error 700 fix" → keyword search is essential
- "How to prevent CUDA error 700 in deep learning?" → hybrid beats both

**Implementation**: run both searches, merge results with Reciprocal Rank Fusion (RRF). Virtually all production RAG systems use hybrid search.
</details>

<details>
<summary><strong>16. What is a vector database? Name 3 examples.</strong></summary>

A vector database stores high-dimensional vectors and enables fast similarity search via approximate nearest neighbor (ANN) algorithms. Unlike SQL databases that query by equality/range, vector DBs query by "which vectors are most similar to this query vector?"

**Core operations**:
- `insert(id, vector, metadata)` — store a vector
- `query(vector, k=10, filter={...})` — find k nearest vectors

**Popular choices**:
- **Chroma** — simple, local-first, great for development
- **Qdrant** — open-source, high performance, good metadata filtering
- **Pinecone** — managed service, simple API, production-ready
- **pgvector** — PostgreSQL extension, good if you're already on Postgres
- **Weaviate** — open-source, supports multimodal, GraphQL API
</details>

<details>
<summary><strong>17. What is LLM-as-judge? What are its limitations?</strong></summary>

LLM-as-judge uses a language model (typically a strong one like GPT-4) to evaluate the outputs of another LLM. Used because human evaluation is slow and expensive.

**Example**:
```
Judge prompt: "Given the question and context below, is this answer 
faithful to the context (i.e., does it only use information from the context)? 
Score 1-5. Question: [Q]. Context: [C]. Answer: [A]."
```

**Limitations**:
- **Positional bias**: judges tend to prefer whichever response appears first
- **Length bias**: judges favor longer, more verbose responses
- **Self-preference**: GPT-4 may unfairly rate GPT-4-style responses higher
- **Calibration**: scores are relative, not absolute — a 4/5 from one judge ≠ a 4/5 from another
- **Doesn't replace humans** for safety-critical evaluations

Best practice: use LLM-as-judge for high-volume automatic screening; maintain a human eval set for calibration.
</details>

<details>
<summary><strong>18. What is semantic caching? How does it differ from exact caching?</strong></summary>

**Exact caching**: cache the response to an exact query string. "What's the weather in NYC?" and "What's the weather in New York City?" are different cache keys — no hit.

**Semantic caching**: embed the query; if a new query's embedding is close enough (cosine similarity > threshold) to a cached query, return the cached response.

"What's the weather in NYC?" and "New York City weather today?" would likely both hit a cached response about NYC weather.

**Production benefit**: cache hit rates of 20–40% are common for consumer apps — significant cost and latency savings.

**Gotcha**: freshness sensitivity. "What are today's headlines?" must never be served from cache. Always filter by topic before applying semantic caching.
</details>

<details>
<summary><strong>19. What is fine-tuning? When should you fine-tune instead of prompting?</strong></summary>

Fine-tuning updates model weights on your specific dataset, teaching the model new behaviors or styles.

**Fine-tune when prompting fails**:
- **Consistent format**: you need outputs in an exact JSON schema every time, and GPT-4 occasionally drifts — fine-tune
- **Brand voice**: you need every response to sound exactly like your brand — fine-tune on examples
- **Cost reduction**: a fine-tuned 7B model matches GPT-4 on your narrow task at 100× lower cost
- **Latency**: smaller fine-tuned model is faster than large prompted model

**Don't fine-tune for knowledge**: "teach the model about our product catalog" — use RAG. Fine-tuning doesn't reliably memorize specific facts.

Starting point: always try prompting + few-shot first. Fine-tune only when prompting is insufficient.
</details>

<details>
<summary><strong>20. What is a system prompt? How is it different from a user message?</strong></summary>

In chat-format LLMs, messages are labeled with roles:
- **System**: instructions to the model — its persona, task, constraints, format requirements
- **User**: what the human is asking
- **Assistant**: what the model has said (included for conversation history)

```python
messages = [
    {"role": "system", "content": "You are a helpful customer service agent for Acme Corp. 
                                   Always respond in 2–3 sentences. Never discuss competitors."},
    {"role": "user", "content": "How do I return a product?"},
    # assistant response would be appended here
]
```

**Key difference**: the system prompt defines the model's behavior; user messages are individual requests. System prompts are meant to be invisible to the end user and are where you configure the AI application's rules.

Note: system prompts are not truly secure — prompt injection can reveal or override them. Don't put secrets in system prompts.
</details>

---

## Coding Interview Prep

At junior level, you'll likely be asked to write code. Practice these:

**Python tasks to master**:
- Call an LLM API (OpenAI, Anthropic) and handle responses
- Implement basic RAG: load → chunk → embed → store → query
- Write a function that calls multiple LLM responses and picks the best one
- Parse JSON output from an LLM and handle malformed output gracefully
- Implement basic semantic search with cosine similarity in numpy

**Data structures/algorithms** (still applies):
- Arrays, hashmaps, sorting, basic graph search
- Python: list comprehensions, generators, async/await

---

## Resources to Study

- 📖 [LLM Fundamentals](../topics/01-llm-fundamentals.md) — start here
- 📖 [Prompt Engineering](../topics/02-prompt-engineering.md)
- 📖 [RAG](../topics/03-rag.md) — up through "Core Concepts" and "Chunking" sections
- 📖 [Vector Databases](../topics/06-vector-databases.md) — fundamentals sections
- 🛠️ Build: [LangChain Quickstart](https://python.langchain.com/docs/get_started/quickstart)
- 🛠️ Build: [OpenAI Cookbook](https://cookbook.openai.com/)

---

[← Back to Home](../README.md) | [Mid-Level →](mid-level.md)
