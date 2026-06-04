# AI Agents & Agentic Systems

[← RAG](03-rag.md) | [Fine-Tuning →](05-fine-tuning.md)

40+ questions covering agent architectures, memory, tool use, multi-agent systems, and production safety.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Agent Fundamentals](#agent-fundamentals)
- [Agent Architectures](#agent-architectures)
- [Memory Management](#memory-management)
- [Tool Use & MCP](#tool-use--mcp)
- [Multi-Agent Systems](#multi-agent-systems)
- [Safety & Reliability](#safety--reliability)
- [Evaluation](#evaluation)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Agent Fundamentals

**Q: What is an AI agent vs a simple LLM call?** `[B]`

An LLM call is stateless: input → output, done. An AI agent wraps an LLM in a loop:

```
while not done:
    action = LLM(observations, memory, tools)
    observation = execute(action)
    memory.update(observation)
```

Key additions:
| Feature | LLM Call | AI Agent |
|---------|---------|---------|
| State | Stateless | Maintains state across steps |
| Execution | Returns text | Executes real actions |
| Iteration | One pass | Multiple reasoning-action cycles |
| Tools | None | Accesses APIs, databases, code |
| Memory | Context only | Short + long-term memory |

Agents unlock tasks impossible with a single LLM call: web research, code execution, database queries, multi-step workflow automation.

---

**Q: What is the agent loop?** `[B]`

The agent loop is the core execution model:

1. **Perceive** — receive user input or tool observations
2. **Reason** — LLM decides what to do next (Thought)
3. **Act** — execute the chosen action (tool call, code execution, message)
4. **Observe** — receive the action's result
5. **Repeat** — continue until task is complete or a stopping condition is met

**Stopping conditions**: task completion detected, max steps reached, user interruption, budget exceeded, unrecoverable error.

---

**Q: What is Context Engineering?** `[I]`

Context engineering is the discipline of managing what information goes into an agent's context window at each step — what to include, exclude, compress, or retrieve.

At each agent step, the context window typically contains:
- System prompt (role, rules, tools)
- Conversation history (possibly compressed)
- Current task state
- Recent tool observations
- Retrieved memory (if using vector memory)

**The challenge**: everything competes for limited context. Poorly managed context → agent loses track of the goal, repeats actions, or ignores important earlier information. Context engineering is often the key differentiator between a toy agent and a production one.

---

## Agent Architectures

**Q: What is ReAct prompting and why is it the foundation of most agents?** `[I]`

ReAct (Yao et al., 2022) interleaves reasoning and acting in a single prompt:

```
Thought: I need to find the current weather in Tokyo.
Action: weather_api({"city": "Tokyo"})
Observation: {"temp": 22, "condition": "Sunny", "humidity": 45}
Thought: I have the weather data. I can now answer the user.
Answer: Tokyo is currently 22°C and sunny.
```

**Why it works as a foundation**:
- Explicit reasoning trace = easier debugging
- Model articulates intent before acting = better tool selection
- Observations are visible in trace = model can recognize failure
- Standardized format works across LLMs and tools

**Variants**: ReAct + self-reflection (Reflexion), ReAct + planning (Plan-and-Execute), ReAct + RAG (Agentic RAG).

---

**Q: What is the Plan-and-Execute agent pattern?** `[I]`

Standard ReAct agents decide one step at a time — can be myopic for long-horizon tasks. Plan-and-Execute separates planning from execution:

```
PLAN PHASE (expensive model):
User: "Research competitors and write a strategic memo"
Plan:
  1. Search for top 5 competitors
  2. For each competitor, retrieve key product info and pricing
  3. Compare features across competitors
  4. Identify our differentiation
  5. Write executive memo in prescribed format

EXECUTE PHASE (can use cheaper model):
  Step 1: [execute] → results
  Step 2: [execute for each competitor] → results
  ...
  Step 5: [write memo using all gathered data] → final output
```

**Benefits**: better decomposition for complex tasks; planner can be GPT-4, executor can be GPT-4o-mini (cost savings); progress is trackable; individual step failures don't derail the whole plan if there's a re-planning mechanism.

---

**Q: What is the Reflexion agent pattern?** `[A]`

Reflexion (Shinn et al., 2023) adds a self-reflection mechanism: after an agent fails or completes a task, it generates a verbal reflection on what went wrong and what to do differently. This reflection is stored in memory and used in the next attempt.

```
Attempt 1: Agent tries task → fails at step 3
Reflection: "I failed because I searched for X but the correct search term was Y. Next time I should..."
Attempt 2: Agent reads reflection → uses different strategy → succeeds
```

Reflexion effectively enables agents to learn from mistakes within a session without gradient-based training. Significant performance improvement on coding benchmarks (HumanEval, HotpotQA).

---

**Q: What is agent orchestration?** `[I]`

Orchestration coordinates the flow of tasks across agents, tools, and human interactions:

- **Deciding which agent to call** based on the task type
- **Passing outputs** from one agent as inputs to the next
- **Managing state** shared across agents
- **Handling errors** — if one agent fails, decide whether to retry, fallback, or escalate
- **Budget tracking** — enforce token/cost limits across the pipeline

Implementation options:
- **LangGraph** — graph-based stateful workflows; nodes = agents/tools, edges = transitions
- **Prefect / Airflow** — DAG-based orchestration for less dynamic workflows
- **Custom orchestrator** — LLM-based supervisor that reads task state and decides next action

---

## Memory Management

**Q: What are the different types of agent memory?** `[I]`

| Memory Type | Storage | Duration | Example |
|-------------|---------|----------|---------|
| **Working** | Context window | Current session only | Recent messages, tool results |
| **Episodic** | Vector DB | Long-term, retrievable | Past conversation summaries |
| **Semantic** | Vector DB / KG | Long-term | Domain facts, user preferences |
| **Procedural** | Model weights | Permanent | How to perform tasks (fine-tuning) |

---

**Q: How do you implement conversation memory for a chatbot?** `[I]`

**Option 1 — Buffer memory**: keep all turns in context. Simple; runs out of context window quickly.

**Option 2 — Sliding window**: keep last N turns. Loses early context abruptly.

**Option 3 — Summarization memory**:
```python
if len(history) > threshold:
    summary = llm.summarize(history[:-recent_k])
    context = [summary_message] + history[-recent_k:]
```

**Option 4 — Vector memory**: embed each turn, retrieve by relevance to current query. Precise recall of specific facts; adds latency.

**Option 5 — Hybrid** (recommended for production):
- Last K turns verbatim (short-term, recency)
- Summary of earlier session (medium-term, general context)
- Vector store of important facts (long-term, specific recall)

---

## Tool Use & MCP

**Q: What is tool use (function calling) in LLMs?** `[B]`

Function calling lets the model request execution of external tools by outputting structured JSON with tool name and parameters:

```json
{
  "tool": "get_weather",
  "parameters": {"city": "Tokyo", "units": "celsius"}
}
```

The application executes the tool and returns the result. The model reasons over the result and either calls another tool or produces a final answer.

Modern LLMs (GPT-4, Claude, Gemini) are trained to generate syntactically correct function calls. You define available tools as a schema (name, description, parameter types) — the LLM selects the right tool and extracts parameters from the user's natural language request.

---

**Q: What is the Model Context Protocol (MCP)?** `[I]`

MCP (Anthropic, 2024) is an open standard that defines how AI applications connect to tools and data sources — "HTTP for AI tool use":

```
[MCP Host: Claude/Cursor/VS Code]
       ↕ MCP Protocol
[MCP Server: GitHub, Slack, Database, File System...]
```

**Three primitives**:
- **Tools** — functions the model can call (like function calling)
- **Resources** — data sources the model can read (files, database records)
- **Prompts** — reusable prompt templates the host can invoke

**Why it matters**: before MCP, every AI app wrote custom integrations for every tool. With MCP, any MCP-compatible model can use any MCP-compatible server without custom glue code. Growing ecosystem of community MCP servers.

---

**Q: How do you design and define tools for an AI agent?** `[I]`

Good tool design determines agent quality. Principles:

**1. Single responsibility**: each tool does exactly one thing. Not `search_and_analyze()`, but `search()` and `analyze()` separately.

**2. Descriptive names and docstrings**: the model selects tools based on the description. Be explicit about when to use the tool and what it returns.

**3. Structured inputs/outputs**: use typed schemas (Pydantic, JSON Schema). Validation prevents the model from passing wrong parameter types.

**4. Fail informatively**: return structured error messages the model can reason about, not stack traces.

**5. Idempotency for safe tools**: read operations should always be safe to retry. Distinguish read from write tools clearly.

```python
@tool
def search_knowledge_base(query: str, top_k: int = 5) -> list[dict]:
    """
    Search the internal knowledge base for information relevant to the query.
    Use this when you need factual information about company products, policies, or procedures.
    Returns a list of relevant document chunks with source metadata.
    Do NOT use this for real-time data like stock prices or weather.
    """
```

---

## Multi-Agent Systems

**Q: What is a multi-agent system and when should you use one?** `[I]`

A multi-agent system has multiple specialized agents collaborating on a shared task, each with a defined role.

**Use multi-agent when**:
- Task requires parallel execution (research multiple topics simultaneously)
- Different subtasks need different capabilities (one agent for code, one for data analysis)
- Task is too complex for a single agent's context window
- Checks and balances are needed (critic agent reviews generator agent's output)

**Avoid multi-agent when**:
- Single agent can accomplish the task — multi-agent adds coordination overhead
- You need deterministic, auditable output — more agents = more unpredictability

---

**Q: How do you prevent contradictions in multi-agent systems?** `[A]`

Contradictions arise from overlapping responsibilities, context fragmentation, and no arbitration mechanism:

**1. Strict role boundaries**: each agent has one clear responsibility with explicit input/output schemas. Retriever doesn't reason; reasoner doesn't retrieve.

**2. Shared context store**: all agents read/write a shared state object — user preferences, established facts, prior decisions. Prevents independent re-derivation of conflicting conclusions.

**3. Orchestrator pattern**: a coordinator agent manages workflow and prevents parallel agents from racing to answer the same question.

**4. Verifier agent**: dedicated agent checks for conflicting claims in the combined output; arbitrates based on source quality, recency, or confidence scores.

**5. Explicit output contracts**: define Pydantic schemas for each agent's output — structured outputs make contradictions detectable programmatically.

---

**Q: What communication topologies exist for multi-agent systems?** `[I]`

| Pattern | Structure | Best For |
|---------|-----------|----------|
| **Orchestrator-Worker** | Central coordinator routes tasks | Complex workflows with diverse subtasks |
| **Sequential Pipeline** | Output of A → input of B | Document processing, multi-step transformation |
| **Parallel Fan-out** | Coordinator spawns N workers → aggregates | Research, comparison tasks, parallelizable work |
| **Peer-to-peer (blackboard)** | Agents publish to/read from shared state | Dynamic, emergent collaboration |
| **Hierarchical** | Agents manage sub-agents | Large-scale tasks requiring recursive decomposition |

---

## Safety & Reliability

**Q: How do you prevent agents from taking irreversible harmful actions?** `[I]`

This is the most critical safety challenge for deployed agents.

**1. Minimal capability**: give agents only the tools they need for their specific task. A research agent doesn't need write access to production databases.

**2. Action classification**:
- Reversible: read operations, draft creation → execute freely
- Recoverable: writes with undo → log before execution
- Irreversible: delete, send, deploy → require explicit confirmation

**3. Human-in-the-loop checkpoints**: for irreversible actions, pause and request user confirmation. Use risk scoring to decide which actions require it.

**4. Dry-run mode**: before executing a consequential action, show the user what *would* happen. User confirms before real execution.

**5. Sandboxing**: execute code in isolated containers; use read replicas for database queries; never give production credentials to an agent running user-provided code.

**6. Audit logging**: every action logged with full context — what was executed, why, and by which agent. Essential for incident response.

---

**Q: How do you handle agent failures and implement error recovery?** `[I]`

Agents fail for predictable reasons: tool errors, invalid tool calls, context window overflow, infinite loops, budget exhaustion.

**Error handling strategies**:
```python
try:
    result = tool.execute(params)
except ToolError as e:
    # Feed error back to agent
    observation = f"Tool failed with error: {e}. Try an alternative approach."
    agent.step(observation)
except MaxRetriesExceeded:
    # Escalate to human or fallback
    return escalate_to_human(task)
```

**Loop detection**: track the last N (action, observation) pairs. If the same action repeats with similar observations, the agent is stuck — interrupt and re-plan.

**Budget enforcement**: track token consumption and tool call count per task. If either exceeds budget, gracefully terminate and return partial results.

---

**Q: What is the human-in-the-loop (HITL) pattern?** `[I]`

HITL pauses agent execution at certain checkpoints to request human input or approval:

```
Agent: "I'm about to send this email to 10,000 customers. Please review: [preview]"
Human: [approves / edits / cancels]
Agent: [proceeds or aborts based on human response]
```

**When to apply HITL**:
- Irreversible actions (send, publish, delete, execute in production)
- High-stakes decisions (financial transactions, medical decisions)
- Confidence below threshold — agent explicitly flags uncertainty
- Novel situations outside the agent's training distribution

**UX consideration**: too many checkpoints defeats automation's purpose. Use risk scoring to calibrate HITL frequency; start conservative, loosen based on trust data.

---

## Evaluation

**Q: How do you evaluate and test AI agents?** `[I]`

Agent evaluation is harder than single-turn LLM evaluation because tasks are multi-step, success criteria are often fuzzy, and non-determinism makes reproducibility hard.

**Metrics**:
- **Task completion rate** — did the agent achieve the goal?
- **Step efficiency** — how many steps taken vs. theoretical minimum?
- **Tool accuracy** — correct tool selected + correct parameters extracted?
- **Error recovery rate** — when a step fails, does the agent recover?
- **Hallucination rate** — does the agent fabricate tool results or observations?

**Testing approaches**:
- **Trajectory evaluation** — evaluate each step in the trace, not just the final output
- **Environment simulation** — mock tool responses deterministically for repeatable tests
- **LLM-as-judge** — evaluate reasoning quality, plan coherence, whether actions match stated goals
- **End-to-end regression suite** — golden tasks with known solutions, run on every code change

---

## Troubleshooting Scenarios

**Q: Your AI agent is stuck in an infinite loop. How do you detect and break it?** `[I]`

**Detection**:
1. **Action hash tracking**: maintain a set of (action_type, params_hash) tuples for the last N steps. If the same action appears 3+ times, flag as a loop.
2. **Progress scoring**: LLM self-assesses after each N steps: "Am I making progress toward the goal? Yes/No/Unclear." Stop if "No" for 2 consecutive assessments.
3. **Max step limit**: hard cap on total steps per task; terminate gracefully if exceeded.

**Recovery**:
1. Inject a meta-observation: "You have taken this action before without success. Choose a different approach."
2. Re-plan from scratch: clear the action history, ask the agent to re-approach the goal differently.
3. Escalate to human with a summary of what was attempted.

---

**Q: Your AI agent burns too many tokens per task. How do you reduce token consumption?** `[I]`

1. **Compress tool observations**: truncate or summarize long tool outputs before adding to context. A 10,000-token search result → 500-token summary.
2. **Prune conversation history**: keep last K turns + initial task description; drop intermediate reasoning steps once a subtask is complete.
3. **Use smaller models for subtasks**: route simple tool-call decisions to a fast/cheap model; reserve expensive models for complex reasoning.
4. **Parallel tool calls**: if multiple independent information lookups are needed, batch them in one model call rather than N sequential calls.
5. **Plan first, then execute**: a good upfront plan reduces wasted exploratory steps.
6. **Cache tool results**: if the agent might call the same tool with the same parameters multiple times, cache results within a session.

---

**Q: Your AI agent hallucinates tool capabilities and passes wrong inputs. How do you fix it?** `[I]`

1. **Improve tool descriptions**: explicit constraints on input types, ranges, and what the tool does NOT accept. Include examples of valid and invalid calls.
2. **Strict parameter validation**: validate parameters before execution; return structured error messages the agent can act on.
3. **Few-shot examples in tool docs**: show 2–3 examples of correct tool calls in the tool description.
4. **Output schema enforcement**: use function-calling APIs (not free-form text parsing) to get guaranteed valid JSON tool calls.
5. **Fine-tune on tool-use data**: if the base model consistently misuses your custom tools, fine-tune on examples of correct usage.

---

[← RAG](03-rag.md) | [Fine-Tuning →](05-fine-tuning.md)
