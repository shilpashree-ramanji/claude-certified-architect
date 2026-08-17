# CCAF Day 1 — Scenario-Based Questions

## Agentic Architecture & Orchestration

**Purpose:** Test whether I can apply Day 1 concepts rather than simply recall definitions.

**Rule:** Do not look at notes while answering. For every question, explain **why** you chose the answer.

---

# How to Use This File

For each scenario:

1. Read the situation carefully.
2. Choose the best architecture/action.
3. Explain your reasoning.
4. Identify the relevant Day 1 concept.
5. Be prepared for follow-up questions that change one condition.

The goal is **architectural reasoning**, not memorization.

---

# Scenario 1 — Simple Task: Single Agent or Multi-Agent?

A company wants an AI system that:

> "Check whether today's data pipeline succeeded and tell me the status."

Available capability:

```text
check_pipeline_status()
```

Two designs are proposed.

### Design A

```text
User
 ↓
Single Agent
 ↓
check_pipeline_status()
 ↓
Answer
```

### Design B

```text
User
 ↓
Manager Agent
 ↓
Pipeline Agent
 ↓
Status Agent
 ↓
Manager
 ↓
Answer
```

### Questions

1. Which design would you choose?
2. Why?
3. What architectural principle supports your decision?
4. What unnecessary complexity would Design B introduce?

---

# Scenario 2 — Workflow or Agent?

A company has this fixed process:

```text
1. Read incoming file
2. Validate file name
3. Validate schema
4. Transform data
5. Load target table
6. Send success/failure notification
```

The sequence almost never changes.

### Questions

1. Would you design this primarily as a workflow or an autonomous agent?
2. Why?
3. If an LLM is added to generate a human-readable failure explanation, does the entire system automatically become an agent? Why or why not?

---

# Scenario 3 — Dynamic Investigation

A user says:

> "Find out why today's pipeline failed."

The system has these tools:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
query_metadata()
```

The agent does not know in advance what caused the failure.

### Questions

1. Why is agentic behavior potentially useful here?
2. What might the agent do after checking the pipeline status?
3. If the pipeline succeeded, should the agent continue checking logs? Why?
4. What part of the agent loop is demonstrated here?

---

# Scenario 4 — Tool Selection

An agent has access to:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
delete_production_table()
send_email()
```

User asks:

> "Tell me whether today's pipeline failed."

### Questions

1. Which tool should the agent initially use?
2. Should it call all five tools to be safe?
3. What does this demonstrate about tools/capabilities?
4. Does having access to a tool mean the agent should use it? Explain.

---

# Scenario 5 — Autonomy

Two systems are proposed.

### System A

The developer defines:

```text
Check status
 ↓
If failed → get logs
 ↓
If schema error → compare schemas
```

### System B

The developer gives the agent:

```text
Goal:
Investigate pipeline failures.

Tools:
- Check status
- Get logs
- Compare schemas
- Check credentials
- Query metadata

Constraints:
- No production modifications
- No data deletion
```

The agent decides which actions are needed.

### Questions

1. Which system has greater autonomy?
2. Why?
3. Does greater autonomy automatically mean System B is better?
4. What role do constraints play in autonomy?

---

# Scenario 6 — Role Specialization

A pipeline investigation system has three agents:

```text
Pipeline Agent
Data Agent
Infrastructure Agent
```

The Pipeline Agent discovers:

> "The pipeline failed because the source column `customer_id` changed from INTEGER to STRING."

### Questions

1. Which agent should investigate the schema mismatch?
2. Why?
3. What does specialization provide here?
4. Would you give every agent every available tool? Why or why not?

---

# Scenario 7 — Handoff

The Pipeline Agent investigates:

> "The pipeline failed."

It checks the logs and discovers:

> "Source credentials have expired."

There is an Infrastructure Agent responsible for credentials and access.

### Questions

1. Should the Pipeline Agent continue investigating credentials itself?
2. What should happen next?
3. Which concept does this demonstrate?
4. Why is the Infrastructure Agent a better fit?

---

# Scenario 8 — Supervisor vs Handoff

Consider this architecture:

```text
Manager Agent
     ↓
Pipeline Agent
     ↓
Schema mismatch
     ↓
Data Agent
```

### Questions

Identify which component is responsible for:

- **A.** Coordinating the overall investigation
- **B.** Discovering that the pipeline has a problem
- **C.** Transferring the schema investigation to the Data Agent
- **D.** Performing schema analysis

Then answer:

5. What is the difference between a supervisor and a handoff?

---

# Scenario 9 — Single Agent vs Multi-Agent

A company wants an agent to:

> "Investigate a failed pipeline, identify the root cause, determine affected tables, check infrastructure impact, and prepare a management report."

Potential architecture:

### Option A

```text
One Agent
 ↓
All tools
 ↓
Complete task
```

### Option B

```text
Manager
 ├── Pipeline Agent
 ├── Data Agent
 ├── Infrastructure Agent
 └── Reporting Agent
```

### Questions

1. Which architecture would you consider first?
2. Why might specialization help?
3. What are the disadvantages of Option B?
4. What additional information would you want before choosing B?

---

# Scenario 10 — Task Decomposition

User asks:

> "Analyze our sales data, identify the top five products, compare their performance across regions, explain the major differences, and create an executive summary."

### Questions

Break this into smaller tasks.

Write at least **4–5 subtasks**.

Then answer:

1. Have you created multiple agents simply by decomposing the task?
2. What is the difference between task decomposition and multi-agent architecture?

---

# Scenario 11 — Sequential or Parallel?

You have three tasks:

```text
A → Research Company A
B → Research Company B
C → Research Company C
```

Each research task is independent.

### Questions

1. Should these tasks be sequential or potentially parallel?
2. Why?
3. What benefit could parallel execution provide?
4. What condition must be true before parallel execution is appropriate?

---

# Scenario 12 — Dependency

Now the tasks change:

```text
A → Find customer ID
B → Retrieve customer's orders
```

### Questions

1. Can A and B safely run in parallel?
2. Why or why not?
3. What execution pattern is more appropriate?
4. What does this tell you about task dependencies?

---

# Scenario 13 — Planning

User asks:

> "Investigate today's pipeline failure and identify the root cause."

A possible approach is:

```text
1. Check pipeline status
2. If failed → read logs
3. Identify failure category
4. Investigate the relevant area
5. Determine root cause
6. Report result
```

### Questions

1. What concept does this represent?
2. Is this the same thing as task decomposition?
3. If the pipeline status is SUCCESS, should the agent blindly continue with the rest of the plan?
4. What does this tell you about dynamic planning?

---

# Scenario 14 — State

During an investigation, the agent has learned:

```text
Goal:
Find why pipeline failed

Status:
FAILED

Logs:
Schema mismatch

Next step:
Compare source and target schemas
```

### Questions

1. What is this information collectively helping the agent maintain?
2. Why is this useful?
3. Does state necessarily mean permanent memory?
4. Give one example of information that could be part of an agent's state.

---

# Scenario 15 — Human-in-the-Loop

An agent investigates a production incident.

It concludes:

> "The fastest fix is to delete the target table and recreate it."

The operation is destructive and irreversible.

### Questions

1. Should the agent automatically execute the action?
2. What architecture pattern would you introduce?
3. Why?
4. Should the agent require human approval for every tool call? Why or why not?

---

# Scenario 16 — Failure and Retry

The agent calls:

```text
get_pipeline_logs()
```

The system responds:

> `Temporary network timeout`

### Questions

1. Should the agent consider retrying?
2. Why?
3. Should it retry forever?
4. What would you consider before retrying?

---

# Scenario 17 — Retry Is Not Always Correct

The agent calls:

```text
create_payment()
```

The request times out.

The agent does not know whether the payment was actually processed.

### Questions

1. Should the agent blindly retry?
2. Why is this different from retrying `get_pipeline_status()`?
3. What concept is relevant here?
4. What could go wrong with an unsafe retry?

---

# Scenario 18 — Orchestration

A complex investigation uses:

```text
Manager Agent
Pipeline Agent
Data Agent
Infrastructure Agent
Reporting Agent
```

The system needs to:

- Decide which agent works next
- Pass relevant information between agents
- Handle dependencies
- Handle failures
- Collect results
- Know when the overall task is complete

### Questions

1. What is the overall coordination of these activities called?
2. What is the role of orchestration?
3. Does orchestration mean that the orchestrator performs every task itself?

---

# Scenario 19 — Architecture Trade-off

A team proposes:

> "Let's use 10 specialized agents because multi-agent systems are more powerful."

The actual requirement is:

> "Read a CSV and calculate the average sales value."

### Questions

1. Would you agree with the architecture?
2. Why or why not?
3. What principle applies?
4. What risks does unnecessary multi-agent complexity introduce?

---

# Scenario 20 — Combined Architecture Challenge

You are designing an AI system for this requirement:

> "Investigate failed data pipelines. Identify the root cause. If the issue is related to data/schema, analyze it. If it is related to infrastructure or credentials, investigate that. For destructive production fixes, obtain human approval. Produce a final explanation for the user."

Available specialists:

```text
Pipeline Agent
Data Agent
Infrastructure Agent
Reporting Agent
```

### Your task

Design the architecture.

Answer these:

1. Would you use a single agent or multiple agents?
2. Would you use a Manager/Supervisor? Why?
3. What would the Pipeline Agent be responsible for?
4. When would the Data Agent be involved?
5. When would the Infrastructure Agent be involved?
6. Where would a handoff occur?
7. Where would Human-in-the-Loop be introduced?
8. What information would need to be maintained as state?
9. Which tasks could potentially run in parallel?
10. How would you handle a temporary tool failure?
11. How would you prevent unsafe retries?
12. What would orchestration be responsible for?

Draw the architecture in your own words.

---

# 🔥 Challenge Round — Defend Your Architecture

For the combined scenario above, do not stop at:

> "I would use multiple agents."

You must explain:

- Why?
- Why not a single agent?
- Why do we need a supervisor?
- Why should the Pipeline Agent not do everything?
- When would you **not** use multiple agents?
- What are the costs of your architecture?
- What happens if one specialist is unavailable?
- What happens if the agent wants to perform a destructive action?

---

# 🧠 CCAF Trap Questions

These are designed to catch memorization without understanding.

## Trap 1

> "A system uses an LLM, therefore it is an agent."

**True or False? Explain.**

## Trap 2

> "A multi-agent system is always better than a single-agent system for complex tasks."

**True or False? Explain.**

## Trap 3

> "If a tool is available to an agent, the agent should use it whenever possible."

**True or False? Explain.**

## Trap 4

> "Task decomposition means creating multiple agents."

**True or False? Explain.**

## Trap 5

> "Parallel execution is always faster."

**True or False? Explain.**

## Trap 6

> "Every failed tool call should be retried."

**True or False? Explain.**

## Trap 7

> "State is the same thing as permanent memory."

**True or False? Explain.**

## Trap 8

> "Human-in-the-loop means a human must approve every action."

**True or False? Explain.**

## Trap 9

> "Higher autonomy is always preferable."

**True or False? Explain.**

## Trap 10

> "A handoff and a supervisor are exactly the same thing."

**True or False? Explain.**

---

# 🏆 Final Day-1 Architecture Challenge

Design an agentic system for:

> **"An enterprise AI assistant that monitors data pipelines, investigates failures, identifies root causes, checks data and infrastructure issues, recommends remediation, and only performs high-impact production changes after human approval."**

Your answer should contain:

```text
1. Overall architecture
2. Agents
3. Roles
4. Tools
5. Autonomy boundaries
6. Task decomposition
7. Planning
8. State
9. Handoffs
10. Supervisor/orchestration
11. Sequential vs parallel decisions
12. Failure/retry strategy
13. Human-in-the-loop
14. Final response flow
```

### Draw it

Try to produce something similar to:

```text
                    USER
                     ↓
              SUPERVISOR / MANAGER
                     ↓
              ┌──────┼──────┐
              ↓      ↓      ↓
          Pipeline  Data   Infra
           Agent   Agent   Agent
              │      │      │
              └──────┼──────┘
                     ↓
                Findings
                     ↓
              Remediation
                     ↓
             High-impact?
              ↙         ↘
            YES          NO
             ↓            ↓
        Human approval  Execute
             ↓            ↓
             └──────┬─────┘
                    ↓
                Reporting
                    ↓
                  USER
```

**Do not copy the above architecture as your answer.** Use it only as a visual prompt. Design your own.

---

# 📊 Self-Assessment

After completing the questions, rate yourself:

| Skill | Score /10 |
|---|---:|
| Agent fundamentals | |
| Agent vs workflow | |
| Tool selection | |
| Autonomy | |
| Roles & specialization | |
| Handoffs | |
| Supervisor pattern | |
| Single vs multi-agent | |
| Task decomposition | |
| Sequential vs parallel | |
| Planning | |
| State | |
| HITL | |
| Failure/retry | |
| Orchestration | |
| Overall Day-1 confidence | |

### Target

**8/10+ before moving to Day 2.**

---

# 🚫 Important Rule for the Challenge

Do **not** try to memorize the answer.

For every scenario, use this reasoning:

```text
What is the goal?
        ↓
How complex is it?
        ↓
What capabilities are needed?
        ↓
Does the agent need autonomy?
        ↓
Are specialized responsibilities needed?
        ↓
Are tasks dependent or independent?
        ↓
What state is needed?
        ↓
What can fail?
        ↓
What actions need human approval?
        ↓
What is the simplest reliable architecture?
```

> **The goal is to think like an architect, not to recite definitions.**
