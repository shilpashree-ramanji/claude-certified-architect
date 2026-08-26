# CCAF Day 2 — Part 2 Assessment
## MCP Integration

**Purpose:** Test your understanding of MCP architecture, components, capabilities, discovery, security, and design trade-offs.

**Rule:** Answer without looking at your Day 2 Part 2 notes.

---

# Section 1 — MCP Fundamentals

### Q1
What does **MCP** stand for?

### Q2
In your own words, what is MCP?

### Q3
Is MCP:

**A.** An LLM  
**B.** An AI agent  
**C.** A protocol  
**D.** A database

Explain your choice.

### Q4
Why do we need a standardized protocol such as MCP when AI applications need access to many external systems?

### Q5
Does using MCP automatically make an application an autonomous AI agent?

Why or why not?

---

# Section 2 — MCP Architecture

### Q6
Complete the basic MCP architecture:

```text
AI Application / Host
        ↓
__________
        ↓
__________
        ↓
External System
```

### Q7
What is the responsibility of the **MCP Host**?

### Q8
What is the responsibility of the **MCP Client**?

### Q9
What is the responsibility of the **MCP Server**?

### Q10
Explain the difference between:

```text
Host
Client
Server
```

in your own words.

---

# Section 3 — Tools, Resources & Prompts

### Q11
What are the three major types of capabilities/context we have discussed in MCP?

### Q12
Complete:

```text
Tools      → ______
Resources  → ______
Prompts    → ______
```

### Q13
Suppose an MCP server exposes:

```text
restart_pipeline()
get_pipeline_logs()
```

Are these tools, resources, or prompts?

### Q14
Suppose an MCP server provides access to:

```text
pipeline documentation
metadata
configuration information
```

Would these be better thought of as tools or resources?

Why?

### Q15
What is an MCP prompt?

Give a simple example.

---

# Section 4 — MCP and Tools

### Q16
How does MCP relate to the tools you learned in Day 2 Part 1?

### Q17
Suppose you have:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
```

What role could an MCP server play in making these capabilities available to an AI application?

### Q18
Does MCP itself decide which tool the agent should use?

Why or why not?

---

# Section 5 — Tool Discovery

### Q19
What is meant by **tool discovery** in an MCP environment?

### Q20
Why is tool discovery useful for an AI application?

### Q21
An MCP server exposes:

```text
search_jira()
get_jira_issue()
create_jira_issue()
delete_jira_issue()
```

The user asks:

> "What is the status of Jira ticket ABC-123?"

Which capability should the agent likely select?

### Q22
Should the agent call all four capabilities just because they are available?

Explain.

---

# Section 6 — MCP Security

### Q23
Why does MCP introduce important security considerations?

### Q24
What does **least privilege** mean when designing an MCP integration?

### Q25
An MCP server exposes:

```text
read_customer_data()
update_customer_data()
delete_customer_data()
```

Should all three have identical permissions and approval requirements?

Why or why not?

### Q26
Why are destructive operations especially sensitive?

### Q27
Where can Human-in-the-Loop controls fit into an MCP architecture?

---

# Section 7 — Error Handling

### Q28
Consider:

```text
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
External API
 ↓
TIMEOUT
```

What could the system do next?

### Q29
Should every MCP failure automatically be retried?

Why not?

### Q30
What factors should influence retry behavior?

Consider:

- Error type
- Idempotency
- Side effects
- Retry limit
- Timeout
- Cost

Explain.

### Q31
What happens if the MCP server itself becomes unavailable?

Give at least two possible system responses.

---

# Section 8 — Architecture Scenarios

## Q32 — Pipeline MCP Server

Your company wants an AI application to investigate pipeline failures.

The MCP server exposes:

```text
Tools:
- check_pipeline_status()
- get_pipeline_logs()
- compare_schemas()

Resources:
- pipeline documentation
- pipeline metadata
```

Explain the role of each capability.

---

## Q33 — DAM Investigation

User says:

> "Why did today's DAM pipeline fail?"

The architecture is:

```text
User
 ↓
AI Application
 ↓
MCP Client
 ↓
Pipeline MCP Server
 ↓
Tools
```

The agent calls:

```text
check_pipeline_status()
```

Result:

```text
FAILED
```

What should logically happen next?

Explain the reasoning.

---

## Q34 — Schema Issue

The logs returned through the MCP server say:

```text
SCHEMA_MISMATCH
```

The MCP server exposes:

```text
compare_schemas()
```

Should the agent call it?

Why?

---

## Q35 — Authentication Issue

The logs say:

```text
AUTH_ERROR
```

But the Pipeline MCP Server does not expose any credential-related capability.

What should the agent do?

Would you:

**A.** Randomly call other tools  
**B.** Retry the same tool forever  
**C.** Report/escalate because the required capability is unavailable  
**D.** Pretend the root cause is known

Explain.

---

# Section 9 — One MCP Server vs Multiple Servers

### Q36

An enterprise has:

```text
GitHub
Jira
Data Platform
Slack
```

Would you automatically put everything behind one MCP server?

Why or why not?

### Q37
Give two reasons why separate MCP servers might be useful.

### Q38
Give two disadvantages of having many separate MCP servers.

### Q39
What architectural factors should influence the decision?

Consider:

- Security boundaries
- Ownership
- Deployment
- Permissions
- Scalability
- Failure isolation
- Maintainability

---

# Section 10 — Architecture Challenge

## Q40

Design an MCP architecture for this requirement:

> "An AI assistant should investigate production data-pipeline failures, read relevant documentation, and create a Jira ticket when a genuine incident is identified."

Available systems:

```text
Pipeline platform
Database
Documentation repository
Jira
```

Design:

```text
AI Host:
MCP Client:
MCP Server(s):
Tools:
Resources:
Prompts:
```

Then explain why you designed it that way.

---

# Section 11 — Security Challenge

## Q41

Your MCP server exposes:

```text
check_pipeline_status()
get_pipeline_logs()
restart_pipeline()
update_pipeline_config()
delete_pipeline_data()
```

The AI assistant is intended mainly for **investigation**.

Which capabilities should be:

- Freely available?
- Restricted?
- Human approval required?

Explain your reasoning.

---

# Section 12 — Trap Questions

### Q42
True or False:

> "MCP is an alternative to an AI model."

Explain.

### Q43
True or False:

> "MCP Server is the same thing as MCP Host."

Explain.

### Q44
True or False:

> "If an MCP server exposes a tool, the agent must use it."

Explain.

### Q45
True or False:

> "MCP automatically provides authorization for every external system."

Explain.

### Q46
True or False:

> "Using multiple MCP servers is always better than using one."

Explain.

---

# Section 13 — Final End-to-End Challenge

## Q47

You are designing an enterprise AI system.

User says:

> "Investigate today's customer ingestion failure. If it is safe, resolve it. If not, tell me what needs to be done."

Architecture available:

```text
AI Host
MCP Client
Pipeline MCP Server
Data MCP Server
Infrastructure MCP Server
Jira MCP Server
```

Capabilities include:

```text
Pipeline:
- check_pipeline_status()
- get_pipeline_logs()
- restart_pipeline()

Data:
- compare_schemas()
- query_metadata()
- validate_source_data()

Infrastructure:
- check_credentials()
- check_service_health()
- update_access_policy()

Jira:
- search_issue()
- create_issue()
```

Resources include:

```text
Pipeline documentation
Data platform documentation
Incident runbooks
```

### Scenario

The agent discovers:

```text
Pipeline = FAILED

Logs = SCHEMA_MISMATCH

Schema comparison =
Source column changed from INTEGER to STRING

Recommended remediation =
Update target schema
```

### Your task

Explain the complete architecture-level response.

Address:

1. Which MCP server(s) should be involved?
2. Which tools should the agent use?
3. Which information should come from resources?
4. Should the agent update the target schema automatically?
5. Where does Human-in-the-Loop fit?
6. Should it restart the pipeline?
7. Should it create a Jira ticket?
8. What should happen if the required capability is unavailable?
9. How should the result be reported to the user?

There is no single magic answer.

**Your reasoning and architectural justification matter more than the exact wording.**

---

# Scoring Guide

| Score | Interpretation |
|---|---|
| 90–100% | Excellent — strong MCP architecture understanding |
| 80–89% | Strong — revise a few weak areas |
| 70–79% | Good foundation — targeted revision needed |
| 60–69% | Partial understanding — revisit MCP architecture |
| <60% | Relearn Part 2 before moving to advanced scenarios |

For architecture questions, prioritize:

```text
Correct mental model
+
Clear responsibilities
+
Security reasoning
+
Appropriate tool selection
+
Good architecture trade-offs
```

---

# Self-Reflection

After completing the assessment:

### 1.
Can I explain MCP without saying only:

> "It's a protocol"?

### 2.
Can I explain Host → Client → Server in my own words?

### 3.
Can I distinguish Tools, Resources, and Prompts?

### 4.
Can I explain why MCP does not automatically make something an agent?

### 5.
Can I reason about MCP security and permissions?

### 6.
Can I design one vs multiple MCP servers based on requirements?

### 7.
Can I connect MCP architecture to the Agent Loop from Day 1?

---

# Assessment Completion Rule

Do not move on just because you can memorize:

```text
Host
 ↓
Client
 ↓
Server
```

You should be able to reason about:

```text
Goal
 ↓
Agent
 ↓
Capability needed
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tool / Resource / Prompt
 ↓
External System
 ↓
Result
 ↓
Agent decides next action
```

> **Goal:** Be able to design and defend an MCP architecture, not merely define MCP.
