# CCAF Day 2 — Part 1 Assessment
## Tool Design & Tool Use

**Purpose:** Test whether you can reason about tools as an AI architect, not merely recall definitions.

**Rule:** Answer without looking at your Day 2 Part 1 notes.

---

## Section 1 — Core Concepts

### Q1
In your own words, what is a tool in an AI agent?

### Q2
What is the difference between an agent and a tool?

### Q3
Why can't an LLM simply access a company's production pipeline without a tool or another integration mechanism?

### Q4
What is the difference between a tool and a workflow?

### Q5
Does using an LLM inside a workflow automatically make the workflow an agent? Explain.

---

## Section 2 — Inputs, Outputs, Schemas & Descriptions

### Q6
Consider:

```text
check_pipeline_status(
    pipeline_name,
    run_date
)
```

What are the inputs, and why should they be explicitly defined?

### Q7
What is a tool schema?

### Q8
What is the difference between a tool's **schema** and its **description**?

### Q9
Why might this tool description be poor?

```text
get_logs()
```

What would you improve?

### Q10
A tool requires `pipeline_name` and `run_id`, but the agent provides only `pipeline_name`.

What kind of problem can a good schema help prevent?

---

## Section 3 — Tool Selection

An agent has these tools:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
restart_pipeline()
```

### Q11
The user asks:

> "Did today's DAM pipeline fail?"

Which tool should the agent probably use first? Why?

### Q12
The status comes back:

```text
SUCCESS
```

Should the agent automatically call all the remaining tools? Why or why not?

### Q13
The status comes back:

```text
FAILED
```

What would be a reasonable next tool? Explain your reasoning rather than giving only the tool name.

---

## Section 4 — Tool Granularity

### Q14
Which design is more fine-grained?

**A**
```text
investigate_pipeline_failure()
```

**B**
```text
check_status()
get_logs()
compare_schema()
check_credentials()
```

Explain.

### Q15
Give one advantage and one disadvantage of fine-grained tools.

### Q16
Give one advantage and one disadvantage of a coarse-grained tool.

### Q17
You have:

```text
investigate_pipeline_failure()
```

It internally checks status, gets logs, compares schemas, checks credentials, and generates a report.

Would you always prefer this over separate tools? Why or why not?

---

## Section 5 — Permissions & Safety

### Q18
Classify these tools as **Read**, **Write**, or **Destructive**:

```text
get_pipeline_logs()
restart_pipeline()
delete_production_table()
query_metadata()
```

### Q19
What does **least privilege** mean in the context of agent tools?

### Q20
A pipeline agent only needs to investigate failures.

Should it have permission to:

```text
read logs
restart pipeline
delete production table
```

Explain.

### Q21
Why should destructive tools generally have stronger controls than read-only tools?

---

## Section 6 — Errors & Retries

### Q22
The agent calls:

```text
get_pipeline_logs()
```

and receives:

```text
TIMEOUT
```

Should it immediately conclude that the pipeline logs don't exist? Why?

### Q23
Which is more likely to be a transient failure?

**A.** Network timeout  
**B.** Invalid credentials  
**C.** Invalid tool input

Explain your choice.

### Q24
The agent retries a tool forever because it keeps failing.

What problems can this create?

### Q25
What factors should influence whether a failed tool call is retried?

---

## Section 7 — Idempotency

### Q26
What does **idempotency** mean in simple terms?

### Q27
Which operation is generally safer to retry?

**A**
```text
get_pipeline_status()
```

**B**
```text
create_payment()
```

Why?

### Q28
Why is idempotency especially important when designing automatic retry behavior?

---

## Section 8 — Human-in-the-Loop

### Q29
An agent discovers that deleting a production table would solve a problem.

Should it automatically execute:

```text
delete_production_table()
```

Why or why not?

### Q30
Design a safer flow for a high-impact operation.

For example:

```text
Agent
 ↓
?
 ↓
?
 ↓
Execute
```

Fill in the missing steps.

---

## Section 9 — Scenario Challenges

### Q31 — Pipeline Investigation

User says:

> "Find why today's DAM pipeline failed."

Available tools:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
restart_pipeline()
```

The agent starts by calling:

```text
check_pipeline_status()
```

Result:

```text
FAILED
```

Then it calls:

```text
get_pipeline_logs()
```

Result:

```text
AUTH_ERROR
```

What should it do next?

Would you:

- **A.** Compare schemas
- **B.** Check credentials
- **C.** Restart the pipeline immediately
- **D.** Call every tool

Explain your choice.

### Q32 — Schema Failure

The logs say:

```text
SCHEMA_MISMATCH
Expected: customer_id
Received: customerId
```

The agent has:

```text
compare_schemas()
```

What should it do?

Would you immediately modify the production schema?

Explain the safety considerations.

### Q33 — No Files

User says:

> "Ingest today's DAM files."

The agent checks the source and finds:

```text
0 files
```

What should the agent do?

Should it continue calling schema and ingestion tools? Why?

### Q34 — Tool Selection Challenge

The agent has:

```text
list_files()
get_file_metadata()
validate_schema()
load_data()
send_email()
```

User says:

> "Are there any DAM files available today?"

Which tool should it select first?

What should it **not** do yet?

### Q35 — Tool Design Challenge

You are asked to create one tool called:

```text
pipeline_helper()
```

It can:

- Check status
- Get logs
- Compare schemas
- Restart pipelines
- Delete pipeline configuration

Would you approve this tool design?

If not, redesign it.

Explain your reasoning.

---

## Section 10 — Architecture Challenge

### Q36

You are designing a production troubleshooting agent.

The agent should:

1. Check whether a pipeline failed.
2. Investigate logs.
3. Identify whether the issue is schema, authentication, or missing data.
4. Hand off to the appropriate specialist when necessary.
5. Never perform destructive actions without approval.

Design the tool set.

You may start with:

```text
Tool 1:
Tool 2:
Tool 3:
Tool 4:
...
```

Then explain why you chose that granularity.

---

## Section 11 — Trap Questions

### Q37
True or False:

> "Every tool should perform as many operations as possible so the agent has fewer tools to choose from."

Explain.

### Q38
True or False:

> "If a tool fails, the agent should always retry."

Explain.

### Q39
True or False:

> "An LLM inside a system automatically means the system is agentic."

Explain.

### Q40
True or False:

> "More autonomous agents should always have more permissions."

Explain.

---

## Section 12 — Final Challenge

### Q41

A manager asks:

> "Find why today's customer ingestion pipeline failed and fix it if it is safe."

Available tools:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
check_credentials()
restart_pipeline()
update_pipeline_config()
delete_pipeline_data()
```

The agent discovers:

```text
Pipeline: FAILED

Logs:
SCHEMA_MISMATCH

Schema comparison:
Source column changed from INTEGER to STRING

Fix:
Changing the target production schema would resolve the mismatch.
```

### Your task

Explain the agent's next steps.

Consider:

- Tool selection
- Reasoning
- Tool permissions
- Safety
- Human approval
- Whether the agent should fix automatically
- Whether another specialist should be involved
- What should be reported to the user

There is no single magic answer. **Your architecture and reasoning matter.**

---

## Scoring Guide

| Score | Interpretation |
|---|---|
| 90–100% | Excellent — ready for advanced scenarios |
| 80–89% | Strong — revise weak areas |
| 70–79% | Good foundation — targeted revision needed |
| 60–69% | Partial understanding — revisit concepts |
| <60% | Relearn Part 1 before moving on |

For scenario questions, prioritize **reasoning quality** over exact wording.

---

## Self-Reflection

After completing the assessment, answer:

1. Which concept was easiest?
2. Which concept was hardest?
3. Where did I hesitate?
4. Which concepts can I explain without notes?
5. Can I design safe tools for an agent without being given the answer?

---

## Assessment Completion Rule

Do **not** move to Day 2 Part 2 merely because you can define the terms.

Move on when you can:

```text
Understand
   ↓
Explain
   ↓
Apply
   ↓
Defend your architecture choice
```

> **The goal is not to memorize tools. The goal is to understand how tool design affects agent behavior, reliability, autonomy, and safety.**
