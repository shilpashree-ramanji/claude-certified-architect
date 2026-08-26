# CCAF Day 2 — Combined Scenario-Based Questions
## Tool Design + MCP Integration

**Purpose:** Scenario-based practice covering Day 2 Part 1 (Tool Design & Tool Use) and Part 2 (MCP Integration).

**How to use:** Don't memorize definitions. For each scenario, explain your reasoning, tool choice, architecture, safety considerations, and what should happen next.

---

# Part A — Tool Design

## Scenario 1 — Pipeline Investigation

You are designing an agent whose goal is:

> "Find out why today's DAM pipeline failed."

Available capabilities:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
```

The agent first checks the pipeline and gets:

```text
FAILED
```

Questions:

1. Which tool should it use next?
2. Why?
3. Should it call all remaining tools?
4. How does the previous observation influence the next decision?

---

## Scenario 2 — Designing the Right Tools

Your architect proposes one tool:

```text
pipeline_debugger()
```

It can:

```text
check status
read logs
compare schemas
check credentials
restart pipeline
modify configuration
```

Questions:

1. Would you approve this tool?
2. What problems could this design create?
3. Would you split it into multiple tools?
4. How would you decide the granularity?

---

## Scenario 3 — Tool Schema

You design:

```text
get_pipeline_logs()
```

Another engineer says:

> "The agent can just figure out what parameters to pass."

Questions:

1. What should the tool schema contain?
2. Why does the agent need the schema?
3. What could happen with a poorly defined schema?

---

## Scenario 4 — Tool Description

You have:

```text
compare()
```

The agent frequently calls it when it shouldn't.

The description is:

> "Compare two things."

Question:

How would you redesign the tool name, inputs, and description so the agent can select it more reliably?

---

## Scenario 5 — Read vs Write

Your agent has:

```text
get_pipeline_status()
get_pipeline_logs()
restart_pipeline()
update_pipeline_config()
delete_pipeline_data()
```

The user says:

> "Investigate the failure."

Questions:

1. Which tools should initially be available?
2. Which should be restricted?
3. Which should potentially require human approval?
4. Why?

---

## Scenario 6 — Retry

The agent calls:

```text
get_pipeline_logs()
```

and receives:

```text
TIMEOUT
```

It retries and gets another timeout.

Questions:

1. Should it retry again?
2. What factors determine the answer?
3. How does idempotency matter?
4. What should happen if the failure continues?

---

## Scenario 7 — Dangerous Retry

The agent calls:

```text
restart_pipeline()
```

The request times out.

The agent doesn't know whether the restart actually happened.

Question:

Should it immediately call `restart_pipeline()` again?

Explain the risk.

---

# Part B — MCP

## Scenario 8 — Introducing MCP

Your company has:

```text
AI Application
GitHub
Jira
Data Platform
Internal APIs
```

Every AI application currently has custom integrations.

The architect proposes using MCP.

Questions:

1. What problem is MCP solving?
2. What does MCP standardize?
3. What does MCP NOT do?

---

## Scenario 9 — Host, Client, Server

A developer draws:

```text
User
 ↓
MCP Server
 ↓
Database
```

Question:

What's missing?

Redesign the architecture using:

```text
Host
Client
Server
External System
```

Then explain each component.

---

## Scenario 10 — MCP Tools

Your Pipeline MCP Server exposes:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
```

The user says:

> "Why did my pipeline fail?"

Questions:

1. How does the AI application discover these capabilities?
2. Which capability should it use first?
3. Does the MCP server decide the investigation strategy?
4. Who decides what to do next?

---

## Scenario 11 — MCP Resources

Your MCP server exposes:

```text
Tools:
check_pipeline_status()
get_pipeline_logs()

Resources:
pipeline documentation
incident runbook
pipeline metadata
```

User asks:

> "What is our company's standard process for handling schema mismatch?"

Question:

Should the agent primarily use a tool or resource?

Why?

---

## Scenario 12 — MCP Prompt

Your MCP server exposes:

```text
Prompt:
analyze_pipeline_incident
```

and:

```text
Tool:
get_pipeline_logs()
```

Question:

What's the conceptual difference between these two?

---

# Part C — Combined Tool + MCP Scenarios

## Scenario 13 — Full Pipeline Investigation

Architecture:

```text
User
 ↓
AI Host
 ↓
MCP Client
 ↓
Pipeline MCP Server
 ↓
Tools
```

Tools:

```text
check_status()
get_logs()
compare_schema()
```

User:

> "Find why today's pipeline failed."

Results:

```text
check_status()
→ FAILED

get_logs()
→ SCHEMA_MISMATCH
```

Questions:

1. What's the next action?
2. Which tool?
3. Why?
4. Does the MCP Client make this decision?
5. Does the MCP Server make this decision?
6. Or does the agent make it?

---

## Scenario 14 — Multiple MCP Servers

You have:

```text
Pipeline MCP Server
Data MCP Server
Infrastructure MCP Server
Jira MCP Server
```

The pipeline logs say:

```text
AUTH_ERROR
```

Pipeline server has:

```text
check_status()
get_logs()
```

Infrastructure server has:

```text
check_credentials()
check_service_health()
```

Questions:

1. Which server should the agent interact with next?
2. Which tool?
3. Why?
4. Does this require a handoff to another agent, or can the same agent use the infrastructure capability?

---

## Scenario 15 — Specialist Agents + MCP

Architecture:

```text
Manager Agent
      ↓
Pipeline Agent
      ↓
MCP Client
      ↓
Pipeline MCP Server
```

Pipeline Agent discovers:

```text
SCHEMA_MISMATCH
```

There is also:

```text
Data Agent
 ↓
Data MCP Server
 ↓
compare_schema()
```

Question:

Would you:

- **A.** Let Pipeline Agent investigate further
- **B.** Hand off to Data Agent
- **C.** Ask the user immediately
- **D.** Call every available capability

Explain your architectural reasoning.

---

## Scenario 16 — MCP Security

Your MCP server exposes:

```text
get_pipeline_logs()
restart_pipeline()
update_pipeline_config()
delete_pipeline_data()
```

The agent's goal is:

> "Investigate failures."

Question:

How would you design permissions for these four capabilities?

---

## Scenario 17 — Human Approval

The agent determines:

> "Changing production configuration will fix the pipeline."

MCP exposes:

```text
update_pipeline_config()
```

The user originally said:

> "Investigate and fix the issue."

Question:

Can the agent automatically execute the tool?

What additional considerations should determine the answer?

---

## Scenario 18 — MCP Server Failure

During an investigation:

```text
Pipeline MCP Server
        ↓
UNAVAILABLE
```

The agent can no longer access:

```text
check_status()
get_logs()
```

Questions:

1. What should the agent do?
2. Should it keep retrying forever?
3. Should it tell the user?
4. Could it use another capability?
5. What information should it report?

---

# Part D — Architecture Challenges

## Scenario 19 — Design the Tool Layer

You're asked to design the capabilities for a production pipeline agent.

Goal:

> "Investigate failures and recommend safe remediation."

Design the tool layer.

You must decide:

- Tool names
- Tool responsibilities
- Inputs
- Outputs
- Permissions
- Which tools are read-only
- Which tools require approval

---

## Scenario 20 — Design the MCP Layer

Now expose those capabilities through MCP.

Design:

```text
AI Host
   ↓
MCP Client
   ↓
MCP Server(s)
   ↓
Tools
   ↓
External Systems
```

Decide:

- One MCP server or multiple?
- Which tools belong together?
- What resources should be exposed?
- What prompts might be useful?
- Where should security boundaries exist?

---

# 🔥 Scenario 21 — Final Day 2 Challenge

You're the architect.

User says:

> **"Investigate today's customer ingestion failure. If it's safe, fix it. Otherwise tell me what needs to be done."**

Architecture:

```text
AI Host
   ↓
MCP Client
   ↓
┌───────────────┬────────────────┬─────────────────┐
↓               ↓                ↓
Pipeline MCP    Data MCP        Infra MCP
Server          Server           Server
```

Available tools:

```text
Pipeline:
check_status()
get_logs()
restart_pipeline()

Data:
compare_schema()
validate_source_data()

Infra:
check_credentials()
check_service_health()
```

Resources:

```text
pipeline documentation
incident runbook
data platform documentation
```

The agent discovers:

```text
Pipeline = FAILED

Logs = SCHEMA_MISMATCH

Schema comparison =
Source column INTEGER
Target column STRING
```

The recommended fix is:

> "Modify the production target schema."

### Your challenge

Walk through the **entire decision process**.

Explain:

1. What does the agent know?
2. What does it need to know?
3. Which tool does it select?
4. Why?
5. Does it need the Data Agent?
6. Does it need another MCP server?
7. Should it modify production automatically?
8. Where does human approval enter?
9. What happens if the tool fails?
10. When does the agent stop?
11. What does it report to the user?

---

# 🎯 What These Scenarios Test

The goal is to connect:

```text
Agent
  +
Goal
  +
Planning
  +
Tools
  +
Tool Design
  +
Autonomy
  +
MCP
  +
Security
  +
Handoffs
  +
Human Approval
  +
Failure Handling
```

into **one coherent architecture**.

## Assessment Rule

Don't answer these by memorizing definitions.

For every scenario, ask:

```text
What does the agent know?
        ↓
What does it need?
        ↓
Which capability is appropriate?
        ↓
Why this capability?
        ↓
What did the result tell us?
        ↓
What should happen next?
        ↓
Is the action safe?
        ↓
Should we continue, hand off, ask for approval, or stop?
```

> **Target:** Be able to defend your architecture decisions, not merely identify the correct tool.
