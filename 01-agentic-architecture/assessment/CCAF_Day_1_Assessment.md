# CCAF Day 1 — Assessment

## Agentic Architecture & Orchestration

**Purpose:** Assess whether I can reason about Day 1 concepts and make architecture decisions.

**Target:** 8/10 or higher before moving to Day 2.

**Rule:** Complete this assessment without looking at notes first. Explain your reasoning, especially for architecture questions.

---

# Part 1 — Core Concepts

## Q1. AI Agent

In your own words, define an **AI agent**.

Then explain what makes an agent different from a system that simply generates an answer.

**Answer:**

---

## Q2. Agent vs Workflow

A system follows:

```text
Get file
 ↓
Validate
 ↓
Transform
 ↓
Load
 ↓
Notify
```

The sequence is fixed.

Is this primarily an **agent** or a **workflow**?

Explain why.

**Answer:**

---

## Q3. Agent Loop

Complete the basic agent loop:

```text
Goal
 ↓
________
 ↓
________
 ↓
________
 ↓
Decide whether goal is complete
```

Then explain each step.

**Answer:**

---

## Q4. Tools

What is a **tool/capability** in an agentic system?

Give three examples from a data engineering environment.

**Answer:**

---

## Q5. Autonomy

Define **autonomy** in the context of an AI agent.

Why does higher autonomy not automatically mean a better system?

**Answer:**

---

# Part 2 — Architecture Decisions

## Q6. Single Agent vs Multi-Agent

Requirement:

> "Check whether today's pipeline failed and tell the user the status."

Available tool:

```text
check_pipeline_status()
```

Would you choose:

**A. Single Agent**

**B. Multi-Agent**

Explain your choice and the architectural principle behind it.

**Answer:**

---

## Q7. Specialized Agents

Requirement:

> "Investigate a failed pipeline. The problem could be a data/schema issue or an infrastructure/credential issue."

Available specialists:

```text
Pipeline Agent
Data Agent
Infrastructure Agent
```

Would specialization provide value here?

Why?

**Answer:**

---

## Q8. Handoff

The Pipeline Agent discovers:

> "The source credentials have expired."

There is an Infrastructure Agent responsible for credentials.

What should happen?

**Answer:**

---

## Q9. Supervisor

You have:

```text
Manager Agent
Pipeline Agent
Data Agent
Infrastructure Agent
```

What is the Manager/Supervisor responsible for?

What is it **not necessarily** responsible for?

**Answer:**

---

## Q10. Task Decomposition

Break this goal into smaller tasks:

> "Investigate a failed pipeline, identify the root cause, determine the affected data, recommend a fix, and prepare a user-friendly explanation."

List at least five subtasks.

Then explain why task decomposition does not automatically mean multiple agents.

**Answer:**

---

# Part 3 — Execution Strategy

## Q11. Sequential vs Parallel

You have:

```text
Task A → Research Source A
Task B → Research Source B
Task C → Research Source C
```

All three are independent.

Should they run sequentially or potentially in parallel?

Why?

**Answer:**

---

## Q12. Dependencies

Now consider:

```text
Task A → Find customer ID
Task B → Retrieve customer's orders
```

Can these safely run in parallel?

Explain.

**Answer:**

---

## Q13. Planning

An agent receives:

> "Investigate today's pipeline failure."

It decides:

```text
1. Check status
2. If failed → get logs
3. Identify failure category
4. Investigate relevant area
5. Determine root cause
6. Report result
```

What concept does this demonstrate?

How could the agent's plan change during execution?

**Answer:**

---

# Part 4 — State & Memory

## Q14. State

During an investigation, the agent knows:

```text
Goal:
Find why pipeline failed

Status:
FAILED

Logs:
Schema mismatch

Next:
Compare schemas
```

What is this information called?

Why does the agent need it?

**Answer:**

---

## Q15. State vs Permanent Memory

True or False:

> "If information is part of an agent's state, it must be permanently stored for future conversations."

Explain.

**Answer:**

---

# Part 5 — Safety & Reliability

## Q16. Human-in-the-Loop

An agent determines:

> "The production table should be dropped and recreated."

The operation is destructive and irreversible.

Should the agent execute automatically?

What should the architecture do instead?

**Answer:**

---

## Q17. Retry

The agent calls:

```text
get_pipeline_logs()
```

and receives:

> Temporary network timeout.

Should it retry?

Should it retry forever?

Explain.

**Answer:**

---

## Q18. Safe Retry / Idempotency

The agent calls:

```text
create_payment()
```

The request times out.

The agent does not know whether the payment succeeded.

Should it blindly retry?

What concept is important here?

What could go wrong?

**Answer:**

---

# Part 6 — Orchestration

## Q19. Orchestration

Define **orchestration** in your own words.

Then explain how orchestration would help coordinate:

```text
Pipeline Agent
Data Agent
Infrastructure Agent
Reporting Agent
```

**Answer:**

---

## Q20. Architecture Trade-off

A team says:

> "We should always use multiple agents because multi-agent systems are more powerful."

Do you agree?

Explain the trade-offs.

**Answer:**

---

# Part 7 — Scenario Challenge

## Q21. Design the Architecture

Design an AI system for:

> "Investigate failed data pipelines, identify the root cause, analyze data/schema issues, investigate infrastructure issues, recommend remediation, and require human approval before high-impact production changes."

Available specialists:

```text
Pipeline Agent
Data Agent
Infrastructure Agent
Reporting Agent
```

Your answer must cover:

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
11. Sequential vs parallel execution
12. Failure/retry strategy
13. Human-in-the-loop
14. Final response flow

**Answer:**

---

# Part 8 — Trap Questions

Answer **True or False** and explain.

## Q22

> "If a system uses an LLM, it is automatically an AI agent."

**Answer:**

---

## Q23

> "Multi-agent architecture is always better for complex problems."

**Answer:**

---

## Q24

> "Every tool available to an agent should be used when investigating a problem."

**Answer:**

---

## Q25

> "Task decomposition means creating multiple agents."

**Answer:**

---

## Q26

> "Independent tasks should always be executed in parallel."

**Answer:**

---

## Q27

> "Every failed tool call should be retried."

**Answer:**

---

## Q28

> "State is the same thing as permanent memory."

**Answer:**

---

## Q29

> "Human-in-the-loop means a human must approve every agent action."

**Answer:**

---

## Q30

> "Higher autonomy is always better."

**Answer:**

---

# Part 9 — Architect-Level Challenge

You are the architect.

A company wants:

> **"An autonomous enterprise AI assistant that monitors data pipelines, investigates failures, identifies root causes, routes specialized investigations, recommends remediation, and executes safe fixes automatically. Any destructive production change requires human approval."**

The system may encounter:

- Schema mismatch
- Missing files
- Invalid data
- Expired credentials
- Infrastructure failures
- Temporary API failures
- Production-impacting remediation

### Design the system.

Draw the architecture yourself.

Your answer should include:

```text
                    USER
                      ↓
                 __________
                      ↓
            ┌─────────┼─────────┐
            ↓         ↓         ↓
         ______     ______     ______
            ↓         ↓         ↓
            └─────────┼─────────┘
                      ↓
                 __________
                      ↓
              Human approval?
                 ↙        ↘
               YES         NO
                ↓           ↓
             ______       ______
                ↓           ↓
                └─────┬─────┘
                      ↓
                   USER
```

Fill in the architecture and explain every major decision.

---

# 📊 Scoring Rubric

Score yourself from **0–2** for each question.

### 0 — Incorrect / Guess

- Definition memorized incorrectly
- Architecture choice unsupported
- Cannot explain reasoning

### 1 — Partially understood

- Correct general direction
- Some reasoning
- Missing important trade-offs or conditions

### 2 — Strong understanding

- Correct answer
- Clear reasoning
- Understands trade-offs
- Can explain when the answer would change

---

# Score Sheet

| Area | Questions | Score |
|---|---:|---:|
| Agent fundamentals | Q1–Q5 | /10 |
| Architecture | Q6–Q10 | /10 |
| Execution strategy | Q11–Q13 | /6 |
| State | Q14–Q15 | /4 |
| Safety & reliability | Q16–Q18 | /6 |
| Orchestration | Q19–Q20 | /4 |
| Scenario design | Q21 | /2* |
| Trap questions | Q22–Q30 | /18 |
| Architect challenge | Q31 | /2* |

**Note:** Q21 and Q31 should be evaluated qualitatively rather than treated as simple 0–2 questions.

---

# 🎯 Readiness Levels

### 0–50%

Go back through the concepts.

### 50–70%

Basic understanding, but architecture reasoning needs work.

### 70–80%

Good foundation. Continue with targeted revision.

### 80–90%

Strong Day-1 understanding.

### 90%+

Excellent. Focus on edge cases, trade-offs, and exam-style wording.

---

# 🔍 Final Self-Reflection

After completing the assessment, answer:

### 1. Which concept was easiest for me?

**Answer:**

### 2. Which concept was hardest?

**Answer:**

### 3. Which concepts do I confuse with each other?

**Answer:**

### 4. Can I explain Agent vs Workflow without using memorized definitions?

**Yes / No**

### 5. Can I explain why I would choose single-agent vs multi-agent?

**Yes / No**

### 6. Can I explain autonomy and its boundaries?

**Yes / No**

### 7. Can I design a basic multi-agent architecture?

**Yes / No**

### 8. Can I reason about retries and safety?

**Yes / No**

### 9. Can I explain orchestration in my own words?

**Yes / No**

### 10. Overall Day-1 confidence:

**__/10**

---

# 🚫 Assessment Rule

Do not judge yourself only by whether you selected the correct option.

The most important question is:

> **"Can I explain why?"**

For architecture questions, a good answer should consider:

```text
Goal
 ↓
Complexity
 ↓
Required capabilities
 ↓
Autonomy
 ↓
Specialization
 ↓
Dependencies
 ↓
State
 ↓
Failure handling
 ↓
Safety / HITL
 ↓
Orchestration
 ↓
Simplest reliable architecture
```

> **Day 1 success = being able to reason about an architecture, not just define agentic concepts.**
