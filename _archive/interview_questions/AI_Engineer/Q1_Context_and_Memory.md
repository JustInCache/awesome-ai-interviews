# AI Engineer Interview Question

## Topic: Context & Memory Management

---

### Question

> A user says your AI assistant forgets important details mentioned earlier in a long conversation. What's going on, and how do you fix it?

---

### Answer

This is a **context window limitation**, not a model failure.

LLMs are stateless and can only attend to a fixed number of tokens—say, 8K, 32K, or 128K depending on the model. Once the conversation exceeds that limit, older information is either dropped entirely or compressed in ways that lose detail.

**The fix is architectural, not model-related.**

Here are the main strategies:

#### 1. Summarization Memory
Periodically compress the conversation history into a summary. The model sees the summary plus recent messages, preserving continuity without hitting token limits.

- **Pros**: Simple to implement, low latency
- **Cons**: Lossy—specific details may be forgotten

#### 2. Vector Memory (RAG-style)
Store conversation turns as embeddings in a vector database. When the user asks something, retrieve relevant past context on demand.

- **Pros**: Precise recall of specific details
- **Cons**: Adds retrieval latency, requires embedding infrastructure

#### 3. Hybrid Memory
Combine both: use summaries for general continuity and vector retrieval for important facts (user preferences, key decisions, etc.).

- **Pros**: Best of both worlds
- **Cons**: More complex to tune and maintain

---

### How I'd Approach This in Practice

First, I'd instrument the system to understand *what* users are forgetting about:
- Is it facts they stated explicitly?
- Is it implicit preferences?
- Is it task context from many turns ago?

Then I'd choose the memory strategy based on the use case:
- **Customer support bot** → Summarization is usually enough
- **Personal assistant with long-term memory** → Vector memory with persistence
- **Complex multi-turn workflows** → Hybrid with structured state management

---

### How to Know It's Working

If the system can recall relevant information without:
- Significant latency increase
- Ballooning costs from huge context windows
- Users having to repeat themselves

...then the memory strategy is working.

---

### What This Question Tests

- Understanding of LLM limitations (statelessness, context windows)
- Ability to design practical solutions around model constraints
- Trade-off thinking (latency vs. accuracy vs. complexity)
- Production mindset—monitoring, iteration, user experience
