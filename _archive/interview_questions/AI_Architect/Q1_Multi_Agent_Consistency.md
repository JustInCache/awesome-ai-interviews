# AI Architect Interview Question

## Topic: Multi-Agent Consistency

---

### Question

> In a multi-agent system, different agents start giving contradictory answers. Why does this happen and how do you prevent it?

---

### Answer

Contradictions in multi-agent systems happen when agents are designed as **independent reasoners without clear role boundaries**. Each agent interprets the task through its own context, and without coordination, they arrive at different conclusions.

---

#### Why This Happens

**1. Overlapping responsibilities**

When two agents both think they're responsible for answering a question, they'll each generate their own answer. If they have different context or different prompts, they'll often disagree.

Example: A "research agent" and a "response agent" both try to answer a factual question. One uses retrieved documents, one uses parametric knowledge. They contradict.

**2. Context fragmentation**

Each agent sees only part of the picture. Without shared state, they make locally rational decisions that are globally inconsistent.

Example: Agent A learns the user prefers formal tone. Agent B never receives this information. Their outputs conflict stylistically.

**3. No ground truth arbitration**

When agents disagree, there's no mechanism to determine which is correct. The system outputs whichever speaks last, or combines them incoherently.

**4. Prompt/instruction inconsistency**

Different agents have different system prompts that encode different assumptions or priorities. These implicit disagreements become explicit in outputs.

---

#### Prevention Strategies

**1. Define explicit roles with clear boundaries**

Each agent should have a single responsibility:

| Agent | Responsibility | Inputs | Outputs |
|-------|---------------|--------|---------|
| Retriever | Find relevant information | Query | Documents |
| Reasoner | Analyze and synthesize | Documents + Query | Analysis |
| Verifier | Check factual accuracy | Analysis + Sources | Verified claims |
| Responder | Generate user-facing output | Verified analysis | Response |

Agents don't overlap. The retriever doesn't reason. The reasoner doesn't retrieve.

**2. Introduce a controller/orchestrator**

A central orchestrator manages workflow:
- Decides which agents to invoke and when
- Passes outputs from one agent as inputs to another
- Ensures no parallel conflicting work

This prevents agents from racing to answer the same question independently.

**3. Enforce contracts on inputs and outputs**

Define explicit schemas for what each agent produces:

```
Reasoner output:
- claim: string
- confidence: high | medium | low
- evidence: list of source references
- caveats: list of limitations
```

When outputs are structured, contradictions become detectable—you can compare claims programmatically.

**4. Use a verifier agent to resolve conflicts**

When contradictions occur (and they will), have a dedicated agent that:
- Detects conflicting claims
- Evaluates evidence for each
- Arbitrates based on source quality, confidence, or explicit rules
- Produces a single consistent output

**5. Shared context layer**

Maintain a shared state that all agents read from and write to:
- User preferences
- Established facts from earlier in the conversation
- Decisions already made

This prevents agents from independently re-deriving (and potentially disagreeing on) settled information.

---

#### Architecture Pattern I'd Recommend

```
┌─────────────────────────────────────────────────┐
│                  Orchestrator                   │
│  (routes tasks, manages state, detects conflicts)│
└──────────────┬───────────────┬──────────────────┘
               │               │
    ┌──────────▼───┐    ┌──────▼──────┐
    │  Retriever   │    │   Reasoner  │
    │  (search,    │    │  (analyze,  │
    │   fetch)     │    │   synthesize)│
    └──────────────┘    └─────────────┘
               │               │
               └───────┬───────┘
                       │
              ┌────────▼────────┐
              │    Verifier     │
              │ (fact-check,    │
              │  resolve)       │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │   Responder     │
              │ (format for     │
              │  user)          │
              └─────────────────┘
```

---

#### Key Principle

**Multi-agent systems need governance, not autonomy.**

The appeal of agents is autonomy—let them figure it out! But without coordination, autonomous agents become contradictory agents. 

The right balance:
- Autonomy **within** an agent's role
- Coordination **between** agents
- Arbitration **when** conflicts arise

Coordination matters more than individual agent intelligence.

---

### What This Question Tests

- System design thinking for complex AI architectures
- Understanding of distributed system coordination patterns
- Ability to anticipate failure modes and design against them
- Practical experience with multi-component systems
