# CCAF Domain 1 — Agentic Architecture & Orchestration

**Domain weight: 27%**

## Study Plan

### 1. Agentic Loop ⭐
- Send request to Claude
- Inspect `stop_reason`
- `tool_use` → execute requested tools
- Append tool results to conversation
- Send the next request
- `end_turn` → terminate

**Anti-patterns**
- Don't parse natural-language signals such as “done”
- Don't use arbitrary iteration caps as the primary stopping mechanism
- Don't use assistant text as the completion indicator

### 2. Multi-Agent Coordinator / Subagent Architecture
- Coordinator / hub-and-spoke pattern
- Isolated subagent context
- Task decomposition and delegation
- Result aggregation
- Dynamic subagent selection
- Iterative refinement
- Coordinator-mediated communication

**Key point:** subagents do not automatically inherit the coordinator's conversation history.

### 3. Subagent Invocation & Spawning
- `Task`
- `allowedTools`
- `AgentDefinition`
- Explicit context passing
- Parallel spawning
- Fork-based session management

### 4. Workflow Enforcement & Handoffs
- Programmatic enforcement vs prompt guidance
- Hooks and prerequisite gates
- Deterministic compliance for mandatory steps
- Structured handoffs containing relevant facts, root cause, and recommended action

### 5. Agent SDK Hooks
- `PostToolUse`
- Tool-call interception
- Data normalization
- Deterministic policy enforcement

Use hooks when a business rule requires guaranteed enforcement rather than relying only on prompts.

### 6. Task Decomposition
**Fixed/sequential:** predictable workflows and prompt chaining.

**Dynamic/adaptive:** open-ended investigations where intermediate findings determine the next tasks.

### 7. Session State, Resumption & Forking
- `--resume`
- `fork_session`
- Resume when prior context is still valid
- Start fresh with a structured summary when previous tool results are stale
- Inform resumed sessions about relevant file changes

---

## Agentic Loop Scenario

Claude responds:

```text
stop_reason = "tool_use"
```

and requests:

```text
get_customer(customer_id="123")
```

**Question:** What should the application do?

A. Return Claude's response and stop  
B. Execute `get_customer`, append the tool result to the conversation, and call Claude again  
C. Ignore the tool call  
D. Execute the tool but don't send the result back

**Answer: B**

Reason:

```text
tool_use
  ↓
Execute tool
  ↓
Append tool result
  ↓
Call Claude again
```

---

## Exam Mindset

For every scenario, ask:

1. What is the agent's goal?
2. What does it know now?
3. What information/capability is needed next?
4. Which tool or subagent should handle it?
5. Is the workflow fixed or adaptive?
6. Is a prerequisite required?
7. Does the receiving agent have the required context?
8. Should the system continue, hand off, or terminate?
9. Is the session state still valid?

---

## Progress

- [ ] Agentic Loop
- [ ] Multi-Agent Coordinator / Subagent Architecture
- [ ] Subagent Invocation & Spawning
- [ ] Workflow Enforcement & Handoffs
- [ ] Agent SDK Hooks
- [ ] Task Decomposition
- [ ] Session State, Resumption & Forking
- [ ] Domain 1 Scenario Assessment
- [ ] Domain 1 Architecture Challenge
