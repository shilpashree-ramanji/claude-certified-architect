# CCAF Domain 1 — Subagent Invocation, Context Passing & Spawning

**Task Statement 1.3**

## 1. Big Picture

```text
Coordinator Agent
       ↓
Decompose the work
       ↓
Choose the right specialist
       ↓
Spawn subagent
       ↓
Pass required context
       ↓
Subagent performs task
       ↓
Result returns to coordinator
       ↓
Coordinator decides next step
```

**Core principle:** A subagent does not automatically inherit the coordinator's conversation history. The coordinator must explicitly provide the context the subagent needs.

---

## 2. Task Tool

The **Task tool** is the mechanism the coordinator uses to spawn a subagent and assign work.

Think:

> **Task tool = programmatic mechanism for starting a specialist subagent and giving it a task.**

Example:

```text
Coordinator
     ↓
Task tool
     ↓
Document Analysis Agent
     ↓
"Analyze this document and return the key findings."
```

### CCAF point

The coordinator needs `Task` in its allowed tools to spawn subagents.

---

## 3. AgentDefinition

An **AgentDefinition** configures the specialist agent's role and behavior.

Typical configuration includes:

- Description
- System prompt
- Tool restrictions

Example:

```text
Research Agent
├── Role: Research and analyze sources
├── Instructions: Research-focused
└── Tools: Only required research tools
```

The goal is specialization and controlled capability.

---

## 4. Explicit Context Passing

A subagent does **not** automatically receive everything the coordinator knows.

Example:

```text
Coordinator knows:
- User wants a report
- Focus on the last 12 months
- Compare cost and performance
- Include sources
```

Bad delegation:

```text
"Research this topic."
```

Better delegation:

```text
Task:
Research this topic.

Requirements:
- Focus on the last 12 months
- Compare cost and performance
- Include cited findings
```

### Key principle

> **Explicitly pass the context required for the subagent to complete its assigned task.**

Do not assume inherited conversation history.

---

## 5. Passing Findings Between Agents

When one subagent's findings are needed by another:

```text
Subagent A
    ↓
Findings
    ↓
Coordinator
    ↓
Subagent B
```

The coordinator should explicitly pass the relevant findings.

Structured findings can contain:

```json
{
  "finding": "...",
  "source": "...",
  "page": "...",
  "relevance": "high"
}
```

Structured data helps preserve:

- Findings
- Sources
- Attribution
- Page/document information
- Other metadata

---

## 6. Allowed Tools

Give each subagent only the tools it needs.

Example:

```text
Research Agent
→ Search tools

Document Agent
→ Document-reading tools

Synthesis Agent
→ Synthesis-related tools
```

Avoid giving every subagent every available tool.

### Why?

```text
Too many tools
      ↓
More choices
      ↓
More complexity
      ↓
Greater chance of incorrect tool selection
```

**Principle:**

> **Specialized role → appropriately scoped tools.**

---

## 7. Parallel Spawning

Independent tasks can be executed in parallel.

```text
             Coordinator
            /     |                 ↓      ↓       ↓
      Research A Research B Analysis C
           \      |       /
            \     |      /
             ↓    ↓     ↓
              Coordinator
                   ↓
               Synthesis
```

If there are no dependencies between tasks, parallel execution can reduce overall latency.

The CCAF blueprint specifically expects knowledge of spawning parallel subagents by emitting multiple Task tool calls in a single coordinator response.

---

## 8. Coordinator Prompt Design

A good coordinator prompt should provide:

- Goal
- Relevant context
- Constraints
- Expected outcome
- Quality criteria

Avoid unnecessarily prescribing every step when the specialist can adapt to findings.

### Good

```text
Investigate the issue and return the root cause,
supporting evidence, and recommended next action.
```

### Less flexible

```text
First search A.
Then search B.
Then open C.
Then perform step D.
```

The specialist should have enough direction to achieve the goal while retaining appropriate flexibility.

---

## 9. Fork-Based Session Management

Fork-based sessions allow different approaches to start from a shared baseline while keeping their subsequent work separate.

```text
Shared baseline
       ↓
   ┌───┴───┐
   ↓       ↓
Approach A Approach B
```

Useful when comparing alternative approaches without mixing their contexts.

---

## 10. Complete Example

### User request

> "Research whether architecture A or architecture B is better and recommend one."

### Coordinator

Breaks the work into independent tasks:

```text
Coordinator
   ↓
 ┌───────────────┬────────────────┐
 ↓               ↓                ↓
Research A    Research B      Cost Analysis
   ↓               ↓                ↓
 └───────────────┴────────────────┘
                 ↓
            Coordinator
                 ↓
         Synthesis Agent
                 ↓
          Recommendation
```

Each subagent receives explicit:

- Goal
- Scope
- Constraints
- Expected output
- Quality criteria

The coordinator then aggregates the findings and passes the relevant information to the synthesis agent.

---

## 11. CCAF Exam Traps

### Trap 1 — Automatic context inheritance

> "Subagents automatically inherit the coordinator's full conversation."

**Wrong.**

Context must be explicitly passed.

### Trap 2 — Give every agent every tool

**Wrong.**

Tools should be scoped to the agent's role.

### Trap 3 — Sequentially run all independent tasks

Not necessarily.

Independent tasks can be spawned in parallel.

### Trap 4 — Let subagents communicate arbitrarily

In the hub-and-spoke architecture, communication is routed through the coordinator.

### Trap 5 — Give only a vague task

**Wrong.**

Pass the relevant context and requirements.

---

## 12. Quick Mental Model

```text
TASK
  ↓
Spawn the specialist

AGENT DEFINITION
  ↓
Define the specialist's role

CONTEXT
  ↓
Give the specialist what it needs

ALLOWED TOOLS
  ↓
Give only required capabilities

RESULT
  ↓
Return findings

COORDINATOR
  ↓
Aggregate / replan / delegate again
```

---

## 13. CCAF Checklist

- [ ] Understand the Task tool
- [ ] Know that `Task` must be allowed for spawning
- [ ] Understand AgentDefinition
- [ ] Know that subagents have isolated context
- [ ] Know that context must be explicitly passed
- [ ] Pass findings to downstream agents
- [ ] Understand structured findings and metadata
- [ ] Restrict tools by agent role
- [ ] Understand parallel subagent spawning
- [ ] Understand coordinator prompt design
- [ ] Understand fork-based session management

## One-Line Memory Aid

> **Task spawns → AgentDefinition specializes → Context informs → allowedTools restricts → Result returns → Coordinator decides what happens next.**
