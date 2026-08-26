# CCAF Domain 1 — Multi-Agent Orchestration

**Domain Weight:** 27%

## 1. What Is Multi-Agent Orchestration?

Multi-agent orchestration uses multiple specialized agents to work together on a larger task.

```text
                 Coordinator
                /     |      \
          Research  Analyze  Review
             Agent    Agent    Agent
                \      |      /
                 Coordinator
                      ↓
                 Final Result
```

The coordinator manages the overall task while specialists handle focused responsibilities.

## 2. Coordinator / Hub-and-Spoke

The coordinator:

- Understands the overall user goal
- Decomposes the goal into subtasks
- Selects appropriate subagents
- Passes required context
- Receives results
- Handles errors
- Aggregates findings
- Decides whether more work is needed
- Produces or requests final synthesis

**Memory aid:** Coordinator = orchestrates; specialist = executes a focused responsibility.

## 3. Why Use Multiple Agents?

Multiple agents provide specialization and controlled responsibility.

Example:

```text
User
 ↓
Coordinator
 ↓
 ┌───────────────┬────────────────┐
 ↓               ↓                ↓
Research A    Research B      Cost Analysis
 └──────────────┴────────────────┘
                 ↓
            Coordinator
                 ↓
          Synthesis Agent
                 ↓
          Recommendation
```

Benefits:
- Specialization
- Separation of responsibilities
- Parallel independent work
- Controlled tool access
- Better handling of complex tasks

## 4. Task Decomposition

The coordinator first understands the overall goal and determines required subtasks.

**Important:** Do not blindly invoke every available subagent. Select the agents needed for the request.

## 5. Dynamic Subagent Selection

Simple request:

```text
User → Coordinator → One specialist → Answer
```

Complex request:

```text
User → Coordinator → Multiple specialists → Aggregate → Synthesis
```

The coordinator should avoid unnecessary agents.

## 6. Isolated Subagent Context

**Critical concept:** Subagents do not automatically inherit the coordinator's full conversation history.

```text
Coordinator knows:
"Compare A and B using the last 12 months."

Subagent does not automatically know this.
```

The coordinator must explicitly provide the relevant context.

**Memory aid:**

> Coordinator knows it ≠ subagent knows it.

## 7. Explicit Context Passing

Useful context includes:

- User goal
- Specific subtask
- Constraints
- Relevant findings
- Expected output
- Quality criteria
- Important evidence

Avoid too little context or excessive irrelevant context.

## 8. Communication Through the Coordinator

In a hub-and-spoke architecture:

```text
Agent A
   ↓
Coordinator
   ↓
Agent B
```

Rather than:

```text
Agent A → Agent B
```

Routing through the coordinator provides centralized orchestration, controlled information flow, clearer ownership, and better observability.

## 9. Result Aggregation

```text
Research A ──┐
Research B ──┼──→ Coordinator → Synthesis
Cost Agent ──┘
```

The coordinator can combine findings, resolve inconsistencies, request clarification, investigate further, or replan.

## 10. Parallel vs Sequential Execution

### Parallel

Use when tasks are independent:

```text
Coordinator
 ├── Agent A ──┐
 ├── Agent B ──┼──→ Coordinator
 └── Agent C ──┘
```

This can reduce latency.

### Sequential

Use when one task depends on another:

```text
Agent A → Result → Agent B → Result → Agent C
```

Example: Research → Analysis → Synthesis.

## 11. Error Handling

The coordinator owns the overall workflow.

If one subagent fails, the coordinator can decide whether to:

- Retry
- Use another subagent
- Continue with partial results
- Replan
- Escalate
- Inform the user

The correct action depends on the task and failure.

## 12. Coordinator vs Specialist

| Coordinator | Specialist |
|---|---|
| Owns overall goal | Owns assigned subtask |
| Decomposes work | Performs specialized work |
| Selects agents | Focuses on its role |
| Passes context | Uses provided context |
| Aggregates results | Returns findings |
| Decides next step | Reports result |

## 13. CCAF Exam Traps

### Trap 1 — Invoke every subagent
**Wrong:** Call all available agents for every request.

### Trap 2 — Assume context inheritance
**Wrong:** A subagent automatically sees the coordinator's conversation.

### Trap 3 — Arbitrary direct agent-to-agent communication
In the hub-and-spoke pattern, route communication through the coordinator.

### Trap 4 — One agent does everything
Use specialization when the task naturally divides into focused responsibilities.

### Trap 5 — Parallelize dependent tasks
If Agent B requires Agent A's result, the tasks have a dependency and should not be treated as independent parallel work.

## 14. Complete Example

```text
                 Coordinator
                /     |      \
               ↓      ↓       ↓
          Logs Agent  Deploy Agent  Config Agent
               \      |       /
                \     |      /
                 ↓    ↓     ↓
                 Coordinator
                       ↓
                Root Cause Agent
                       ↓
                 Final Answer
```

The coordinator:
1. Understands the goal.
2. Decomposes the investigation.
3. Selects relevant specialists.
4. Passes required context.
5. Runs independent investigations in parallel when appropriate.
6. Collects findings.
7. Resolves conflicting evidence.
8. Sends relevant findings for synthesis.
9. Produces the final result.

## 15. CCAF Checklist

- [ ] Multi-agent orchestration
- [ ] Coordinator / hub-and-spoke
- [ ] Coordinator responsibilities
- [ ] Specialist responsibilities
- [ ] Task decomposition
- [ ] Dynamic subagent selection
- [ ] Isolated subagent context
- [ ] Explicit context passing
- [ ] Coordinator-mediated communication
- [ ] Result aggregation
- [ ] Parallel vs sequential execution
- [ ] Subagent failure handling
- [ ] Common architecture traps

## One-Line Memory Aid

> **Coordinator decides → specialists work → results return → coordinator aggregates → coordinator decides what happens next.**
