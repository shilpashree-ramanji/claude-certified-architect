# CCAF Day 2 — Part 1: Tool Design & Tool Use

## Goal

Understand how tools give AI agents the ability to interact with external systems and how to design those tools so agents can use them reliably, safely, and appropriately.

---

## 1. What is a Tool?

A **tool** is a capability an AI agent can invoke to perform an action or retrieve information from an external system.

Examples from Data Engineering:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
query_metadata()
read_source_file()
```

A tool may internally query a database, call an API, read a file, search a system, execute code, or trigger an operation.

> **A tool is not necessarily a database table.**

Example:

```text
Agent
  ↓
query_metadata()
  ↓
SQL Database
  ↓
Metadata Table
```

---

## 2. Tool vs Agent

**Agent:** decides what should happen next.

**Tool:** provides a specific capability/action.

```text
User
 ↓
Agent
 ↓
"I need to know whether the pipeline failed."
 ↓
check_pipeline_status()
 ↓
Pipeline System
 ↓
FAILED
 ↓
Agent decides next action
```

Mental model:

```text
Agent = decision maker
Tool  = capability/action
```

---

## 3. Tool Design Principles

A good tool should generally be:

- Clear
- Purpose-specific
- Predictable
- Well described
- Appropriately scoped
- Safe
- Easy for the agent to select

### Bad

```text
do_everything()
```

### Better

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
```

Each capability has a clearer responsibility.

---

## 4. Tool Inputs

Tools usually need information from the agent.

Example:

```text
check_pipeline_status(
    pipeline_name,
    run_date
)
```

Inputs:

```text
pipeline_name → string
run_date      → date
```

Example call:

```text
check_pipeline_status(
    pipeline_name="customer_ingestion",
    run_date="2026-08-18"
)
```

---

## 5. Tool Outputs

A tool should return useful, structured information.

Example:

```json
{
  "status": "FAILED",
  "run_id": "12345",
  "start_time": "2026-08-18T20:00:00",
  "error_code": "SCHEMA_MISMATCH"
}
```

The agent can reason from the result:

```text
Tool result
     ↓
Agent observes
     ↓
Agent decides next action
```

This connects directly to the **Agent Loop** from Day 1.

---

## 6. Tool Input + Output

Think of a tool as:

```text
                TOOL
                 │
        ┌────────┴────────┐
        ↓                 ↓
      INPUT             OUTPUT
        │                 │
        ↓                 ↓
  What it needs     What it returns
```

Example:

```text
check_pipeline_status()

INPUT:
- pipeline_name
- run_date

OUTPUT:
- status
- run_id
- error information
```

A well-designed contract makes the tool easier for the agent to use reliably.

---

## 7. Tool Schema

A **tool schema** describes the structured interface of a tool.

Example:

```text
Tool:
check_pipeline_status

Inputs:
- pipeline_name: string
- run_date: date

Returns:
- status
- run_id
- error_code
```

The schema tells the agent/application:

> **What arguments does this tool accept, and what structure should they have?**

Poor schemas can lead to:

- Missing required inputs
- Wrong data types
- Wrong parameter names
- Malformed tool calls

---

## 8. Tool Description

A schema explains **how** to call a tool.

A description helps explain **when and why** to use it.

Compare:

```text
get_logs()
```

with:

```text
get_pipeline_logs(
    pipeline_name,
    run_id
)
```

and a useful description:

> Retrieves execution logs for a specific pipeline run. Use this after a pipeline failure when investigating the root cause.

Mental model:

> **Schema = structure**

> **Description = meaning and usage guidance**

---

## 9. Tool Selection

An agent may have many available tools:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
query_metadata()
```

User asks:

> "Did today's pipeline fail?"

The agent should probably start with:

```text
check_pipeline_status()
```

It does not need to call every available tool.

Tool selection depends on:

- Goal
- Current state
- Previous observations
- Tool descriptions
- Tool schemas
- Constraints

Connection to Day 1:

```text
Goal
 ↓
Decide
 ↓
Select appropriate tool
 ↓
Act
 ↓
Observe
 ↓
Decide again
```

---

## 10. Tool Granularity

**Tool granularity** means how much responsibility one tool contains.

### Fine-grained

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
```

### Coarse-grained

```text
investigate_pipeline_failure()
```

which might internally:

```text
check status
    ↓
get logs
    ↓
analyze logs
    ↓
compare schemas
    ↓
check credentials
```

Neither is automatically correct.

Ask:

> **Does the agent need control over the individual steps?**

If yes, finer-grained tools may help.

If the operation should behave as one well-defined capability, a coarser tool may make sense.

---

## 11. Fine-Grained Tools

Potential advantages:

- More control
- Better composability
- Agent can choose individual operations
- Easier reuse
- Easier to apply different permissions

Potential disadvantages:

- More tools to understand
- More tool-selection decisions
- More coordination

---

## 12. Coarse-Grained Tools

Potential advantages:

- Simpler interface
- Fewer tool calls
- Less orchestration for a fixed operation

Potential disadvantages:

- Less control
- Less flexibility
- May perform unnecessary work
- Harder to reuse internal operations
- May hide important decisions

> **Tool granularity is an architectural trade-off.**

---

## 13. Tool Permissions & Safety

Not all tools should have the same level of authority.

```text
READ
├── check_pipeline_status()
├── get_pipeline_logs()
└── query_metadata()

WRITE
├── restart_pipeline()
└── update_configuration()

DESTRUCTIVE
└── delete_production_table()
```

The more powerful the operation, the stronger the controls may need to be.

Connection to Day 1:

```text
Autonomy
   +
Tool permissions
   +
Safety constraints
   ↓
Safe agent behavior
```

---

## 14. Least Privilege

A useful security principle:

> **Give the agent/tool only the permissions required to perform its job.**

For example, a Data Analysis Agent may need read access to schema and analytical data, but may not need permission to delete production data or alter production schemas.

Why?

If the agent makes a mistake, the potential impact is limited.

---

## 15. Tool Errors

Tools can fail:

```text
Agent
 ↓
get_pipeline_logs()
 ↓
TIMEOUT
```

Possible responses:

```text
Retry
Try alternative
Handle error
Ask user
Escalate
Stop
```

The correct response depends on the failure type.

---

## 16. Transient vs Persistent Failures

### Transient

Potentially temporary:

```text
Network timeout
Temporary service unavailable
Temporary connection failure
```

A controlled retry may make sense.

### Persistent / Structural

Examples:

```text
Permission denied
Invalid credentials
Invalid input
Unsupported operation
```

Blindly retrying may not solve the problem.

```text
Tool failure
    ↓
What type?
    ↓
Transient? ── YES → Controlled retry
    │
    NO
    ↓
Handle / escalate / change approach
```

---

## 17. Retry Limits

Never assume:

> "If the tool fails, keep trying forever."

Safer pattern:

```text
Attempt 1 → FAIL
Attempt 2 → FAIL
Attempt 3 → FAIL
        ↓
Stop / escalate
```

Retry policy can depend on:

- Error type
- Maximum attempts
- Timeout
- Cost
- Side effects
- Idempotency

---

## 18. Tool Idempotency

Ask:

> **What happens if this tool is called twice?**

Example:

```text
get_pipeline_status()
```

Repeated calls are generally safe.

But:

```text
create_payment()
```

could create duplicate side effects if called again incorrectly.

Another example:

```text
restart_pipeline()
```

may have side effects and needs more careful retry behavior.

> **Retrying a tool is both a reliability question and a safety question.**

---

## 19. Read vs Write vs Destructive Tools

A useful distinction:

```text
READ
↓
Get information

WRITE
↓
Change state

DESTRUCTIVE
↓
Irreversible/high-impact change
```

Examples:

```text
READ
check_pipeline_status()

WRITE
restart_pipeline()

DESTRUCTIVE
delete_production_table()
```

Risk level affects:

- Permissions
- Approval requirements
- Retry behavior
- Logging
- Monitoring
- Autonomy boundaries

---

## 20. Human Approval for High-Impact Tools

Suppose the agent has:

```text
delete_production_table()
```

A safer architecture may be:

```text
Agent
 ↓
Recommend action
 ↓
Human approval
 ↓
Approved?
 ↙        ↘
YES        NO
 ↓          ↓
Execute    Stop / revise
```

This connects directly to Day 1's **Human-in-the-Loop** concept.

---

## 21. Tool Design and Agent Autonomy

These concepts work together:

```text
                 AGENT
                   ↓
                  GOAL
                   ↓
              Decide action
                   ↓
              Select tool
                   ↓
                  TOOL
                   ↓
                 Result
                   ↓
                Observe
```

Agent autonomy is bounded by:

- Available tools
- Tool permissions
- System constraints
- Safety policies
- Human approval requirements

---

## 22. Pipeline Agent Example

Suppose we design tools for a pipeline troubleshooting agent.

### Tool 1

```text
check_pipeline_status(
    pipeline_name,
    run_date
)
```

Purpose:

> Determine whether a pipeline run succeeded or failed.

### Tool 2

```text
get_pipeline_logs(
    pipeline_name,
    run_id
)
```

Purpose:

> Retrieve logs for a specific pipeline execution.

### Tool 3

```text
compare_schemas(
    source,
    target
)
```

Purpose:

> Compare source and target schemas and identify differences.

### Tool 4

```text
check_credentials(
    connection_name
)
```

Purpose:

> Check whether the configured connection is valid and accessible.

The agent can compose these capabilities:

```text
Goal
 ↓
check_pipeline_status()
 ↓
FAILED
 ↓
get_pipeline_logs()
 ↓
Schema mismatch
 ↓
compare_schemas()
 ↓
Root cause
```

---

## 23. Tool Design Checklist

Before exposing a tool to an agent, ask:

```text
1. Is the purpose clear?
2. Are the inputs clear?
3. Are the outputs structured?
4. Is the tool description useful?
5. Is the scope appropriate?
6. Is the tool too broad?
7. Is the tool too granular?
8. What permissions does it need?
9. What side effects can it cause?
10. Is it safe to retry?
11. What happens when it fails?
12. Does it require human approval?
```

---

# 🧠 Part 1 Mental Model

```text
                    AGENT
                      ↓
                    GOAL
                      ↓
              Choose capability
                      ↓
                     TOOL
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Inputs      Schema     Description
          ↓
       Execute
          ↓
        Output
          ↓
       Observe
          ↓
       Continue
```

For safe architecture:

```text
Tool
 ├── Clear purpose
 ├── Good schema
 ├── Useful description
 ├── Appropriate granularity
 ├── Limited permissions
 ├── Known side effects
 ├── Error handling
 └── Safe retry strategy
```

---

# 📌 Key Definitions

| Concept | Simple meaning |
|---|---|
| **Tool** | Capability an agent can invoke |
| **Tool Input** | Information required by the tool |
| **Tool Output** | Result returned by the tool |
| **Tool Schema** | Structured definition of the tool interface |
| **Tool Description** | Explanation of what/when/why to use the tool |
| **Tool Selection** | Agent choosing an appropriate capability |
| **Tool Granularity** | How much responsibility a tool contains |
| **Fine-grained Tool** | Focused capability |
| **Coarse-grained Tool** | Larger capability containing multiple operations |
| **Tool Permission** | What the tool is allowed to access/change |
| **Least Privilege** | Give only required permissions |
| **Transient Failure** | Potentially temporary failure |
| **Retry** | Reattempt an operation |
| **Idempotency** | Safety of repeating an operation |
| **HITL** | Human approval for selected actions |

---

# 🔥 Architecture Principles

1. **A tool should have a clear purpose.**
2. **The agent should not call every available tool.**
3. **Tool schemas make tool invocation structured and predictable.**
4. **Tool descriptions help the agent understand when and why a tool should be used.**
5. **Tool granularity is an architectural trade-off.**
6. **Give tools only the permissions they need.**
7. **Do not blindly retry failed tools.**
8. **Consider side effects and idempotency before retrying.**
9. **High-impact tools may require human approval.**
10. **Tool design directly influences agent reliability and safety.**

---

# 🎯 Connection to Day 1

Day 1:

```text
Agent
 ↓
Goal
 ↓
Autonomy
 ↓
Planning
 ↓
State
 ↓
Decision
```

Day 2 Part 1:

```text
Decision
 ↓
Tool Selection
 ↓
Tool
 ↓
Input
 ↓
Execution
 ↓
Output
 ↓
Observation
 ↓
Agent continues
```

Full picture:

```text
                 USER GOAL
                     ↓
                   AGENT
                     ↓
                  DECIDE
                     ↓
              SELECT TOOL
                     ↓
                   TOOL
                     ↓
                  RESULT
                     ↓
                  OBSERVE
                     ↓
               UPDATE STATE
                     ↓
             Goal complete?
                ↙       ↘
              YES        NO
               ↓          ↓
             STOP      Continue
```

---

# 📝 Part 1 Revision Checklist

Before moving to MCP, I should be able to explain:

- [ ] What a tool is
- [ ] Tool vs agent
- [ ] Tool inputs
- [ ] Tool outputs
- [ ] Tool schema
- [ ] Tool description
- [ ] Tool selection
- [ ] Fine-grained vs coarse-grained tools
- [ ] Tool permissions
- [ ] Least privilege
- [ ] Tool errors
- [ ] Retry limits
- [ ] Idempotency
- [ ] Read vs write vs destructive tools
- [ ] Human approval for high-impact tools
- [ ] How tools connect to the Day 1 agent loop

---

# 🔜 Next

```text
Part 1 Learning
      ↓
Revision
      ↓
Tool Design Scenarios
      ↓
Part 1 Assessment
      ↓
PART 2 — MCP
```

> **Goal:** Don't just know what a tool is. Be able to decide whether a tool is well-designed, safe, appropriately scoped, and suitable for an agent.
