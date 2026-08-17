# CCAF Day 1 --- Agentic Architecture & Orchestration

> **Certification:** CCAF --- Claude Certified Architect\
> **Day:** 1\
> **Domain:** Agentic Architecture & Orchestration\
> **Exam Weight:** 27%\
> **Status:** First-pass learning complete; revision + scenario practice
> pending

------------------------------------------------------------------------

## 🎯 Day 1 Goal

Understand the foundations of agentic systems and how to architect
agents to accomplish goals using tools, planning, state, orchestration,
and appropriate levels of autonomy.

The key approach for this day:

> **Understand the concept → connect it to a real Data Engineering
> example → reason through an architecture scenario.**

------------------------------------------------------------------------

# 1. What is an AI Agent?

An **AI agent** is an AI system that works toward a goal by deciding
what actions to take and using available tools or capabilities to
accomplish that goal.

### Example

User:

> "Find out why today's data pipeline failed."

The agent may:

``` text
Goal
  ↓
Check pipeline status
  ↓
FAILED
  ↓
Check logs
  ↓
Schema mismatch
  ↓
Compare schemas
  ↓
customer_id: INTEGER → STRING
  ↓
Root cause identified
  ↓
Return result
```

The important point is that the agent uses the result of one action to
decide what to do next.

### Key idea

> **Agent = Goal + Decision-making + Actions/Tools + Observation +
> Iteration**

------------------------------------------------------------------------

# 2. Agent vs Workflow

## Workflow

A workflow follows a predefined sequence.

``` text
Step 1
  ↓
Step 2
  ↓
Step 3
  ↓
Step 4
```

Example:

``` text
Get Metadata
     ↓
Validate File
     ↓
Transform
     ↓
Load Table
     ↓
Send Notification
```

The developer/application defines the path.

## Agent

An agent receives a goal and can dynamically decide the next action
based on what it discovers.

``` text
Goal
 ↓
Decide
 ↓
Act
 ↓
Observe
 ↓
Decide
 ↓
Act
 ↓
...
```

### Important distinction

> **Workflow:** Developer/application primarily defines the sequence.

> **Agent:** The agent has some authority to determine the next action
> based on its goal, observations, available capabilities, and
> constraints.

### Important nuance

An AI-powered workflow can contain an LLM and still be a workflow.

Do not memorize:

> Workflow = no AI

Instead ask:

> **Who determines the sequence of actions?**

------------------------------------------------------------------------

# 3. Agent Loop

A basic agent loop is:

``` text
Goal
 ↓
Decide
 ↓
Act
 ↓
Observe
 ↓
Decide
 ↓
Act
 ↓
Observe
 ↓
Goal complete?
 ↙        ↘
YES        NO
 ↓          ↓
STOP      Continue
```

Another way to remember it:

> **Act → Observe → Decide → Act → Observe → ... → Finish**

### Pipeline example

``` text
Goal:
Find why pipeline failed
        ↓
check_pipeline_status()
        ↓
FAILED
        ↓
Agent decides:
"I need the logs."
        ↓
get_pipeline_logs()
        ↓
Schema mismatch
        ↓
Agent decides:
"I need to compare schemas."
        ↓
compare_schemas()
        ↓
Root cause found
        ↓
STOP
```

### Important principle

The agent should not blindly execute every available tool.

It should continually ask:

> **"Have I achieved the goal? If not, what should I do next?"**

------------------------------------------------------------------------

# 4. Tools / Capabilities

A **tool** is a capability provided to an AI agent that allows it to
interact with something outside the model.

Examples:

``` text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
query_metadata()
read_source_file()
send_email()
```

These are not necessarily database tables.

A tool could internally:

-   Query a database
-   Call an API
-   Read a file
-   Search a system
-   Execute code
-   Interact with another service

### Important distinction

> **Tool ≠ necessarily a table.**

A tool is an interface/capability through which the agent can perform an
action or obtain information.

### Tool selection

The developer provides the available capabilities.

The agent can determine which relevant capability to use based on:

-   The goal
-   Current context
-   Previous observations
-   Constraints

------------------------------------------------------------------------

# 5. Autonomy

**Autonomy** is the degree of freedom an agent has to decide what
actions or tools to use to accomplish a goal.

### Low autonomy

The developer defines most of the sequence:

``` text
Check status
   ↓
If failed → Check logs
   ↓
If schema issue → Compare schemas
```

### Higher autonomy

The developer provides:

``` text
Goal:
Investigate today's pipeline failure

Tools:
- Check status
- Get logs
- Compare schemas
- Query metadata
- Check source

Constraints:
- Cannot modify production
- Cannot delete data
```

The agent determines what to do next.

### Key definition

> **Autonomy = freedom to decide, within constraints.**

### Important principle

More autonomy is **not automatically better**.

High-impact actions may require restrictions or human approval.

------------------------------------------------------------------------

# 6. Agent Roles and Specialization

A **role** can define the responsibility of an agent.

Example:

``` text
Pipeline Agent
→ Investigates pipeline execution problems

Data Agent
→ Investigates data and schema problems

Infrastructure Agent
→ Investigates credentials, access, and infrastructure

Reporting Agent
→ Produces summaries/reports
```

### Why specialize?

A complex system can be easier to manage when responsibilities are
separated.

``` text
                  Manager
                 /   |   \
                ↓    ↓    ↓
          Pipeline  Data  Infra
           Agent   Agent  Agent
```

Each agent can have:

-   Different responsibilities
-   Different instructions
-   Different tools
-   Different constraints

### Important nuance

> **Role does not necessarily mean a separate agent.**

A role can describe the responsibility/behavior of an agent.

------------------------------------------------------------------------

# 7. Handoff

A **handoff** occurs when responsibility for the next part of a task is
transferred from one agent to another specialized agent.

Example:

``` text
Pipeline Agent
      ↓
Finds schema mismatch
      ↓
HANDOFF
      ↓
Data Agent
      ↓
Compare schemas
      ↓
Identify changed column
```

### Example with credentials

``` text
Pipeline Agent
      ↓
Finds expired credentials
      ↓
Infrastructure / Access Agent
```

The issue should go to the agent with the appropriate responsibility.

### Key principle

> **The next agent should be selected based on the responsibility
> required by the current problem.**

------------------------------------------------------------------------

# 8. Why Use Multiple Specialized Agents?

A single agent could potentially have many tools:

``` text
                    ONE AGENT
                        │
       ┌────────────────┼────────────────┐
       ↓                ↓                ↓
   Pipeline          Data           Infrastructure
    Tools            Tools               Tools
```

For a complex system, this can become difficult to manage.

Specialization can provide:

1.  Separation of responsibilities
2.  Better tool organization
3.  Easier maintenance
4.  Easier debugging
5.  Better handling of complex tasks

### But multi-agent systems have costs

More agents can introduce:

-   More coordination
-   More communication
-   More latency
-   More cost
-   More failure points
-   More architectural complexity

### Key principle

> **Use the simplest architecture that can reliably accomplish the
> goal.**

Do not use multiple agents simply because you can.

------------------------------------------------------------------------

# 9. Supervisor / Manager Pattern

A **Supervisor/Manager Agent** coordinates specialized agents.

Example:

``` text
                 Manager Agent
                      ↓
             Investigate failure
                      ↓
                Pipeline Agent
                      ↓
              What is the issue?
                ↙           ↘
        Schema issue     Credential issue
             ↓                 ↓
        Data Agent       Infrastructure Agent
```

The supervisor can:

-   Coordinate work
-   Decide which agent should handle a task
-   Route tasks
-   Collect results
-   Determine when the overall task is complete
-   Produce or coordinate the final response

### Important distinction

**Supervisor pattern:**

> Supervisor coordinates which agent handles the next part.

**Handoff pattern:**

> One agent transfers responsibility to another agent.

These patterns can be combined.

------------------------------------------------------------------------

# 10. Single-Agent Architecture

A single agent may have several relevant tools:

``` text
                 Agent
          ┌───────┼────────┐
          ↓       ↓        ↓
       Pipeline   SQL    Schema
        Tools    Tools    Tools
```

Example:

> "Investigate why the pipeline failed."

The single agent may:

``` text
Check status
    ↓
Read logs
    ↓
Check source
    ↓
Compare schema
    ↓
Explain root cause
```

### When to prefer it

Use a single-agent approach when:

-   The task is relatively simple
-   One agent can reliably handle the task
-   Specialization is unnecessary
-   Multi-agent coordination would add unnecessary complexity

------------------------------------------------------------------------

# 11. Multi-Agent Architecture

A multi-agent architecture uses multiple agents with different
responsibilities.

``` text
                   Manager
                 /    |     \
                ↓     ↓      ↓
           Pipeline   Data   Infra
             Agent   Agent   Agent
```

### When it may help

-   Complex tasks
-   Naturally divisible tasks
-   Different areas of expertise
-   Different tools/responsibilities
-   Need for specialized reasoning

### Trade-off

Multi-agent architecture introduces additional coordination overhead.

Therefore:

> **Multi-agent ≠ automatically better.**

------------------------------------------------------------------------

# 12. Task Decomposition

**Task decomposition** means breaking a large goal into smaller tasks.

Example:

> "Analyze this sales dataset, identify the top 5 products, compare
> their performance across regions, explain the results, and create a
> management summary."

Can become:

``` text
Main Goal
   │
   ├── Analyze sales
   ├── Identify top 5 products
   ├── Compare regions
   ├── Explain results
   └── Create summary
```

### Important distinction

> **Task decomposition does not automatically mean multiple agents.**

One agent can perform multiple decomposed tasks.

Decomposition answers:

> **"What smaller tasks make up this goal?"**

It does not automatically answer:

> **"Which agent performs each task?"**

------------------------------------------------------------------------

# 13. Sequential vs Parallel Execution

After decomposing a task, we need to consider execution order.

## Sequential

``` text
Task A
  ↓
Task B
  ↓
Task C
```

Use when one task depends on another.

Example:

``` text
Find customer ID
       ↓
Get customer's orders
```

Task B needs Task A's output.

## Parallel

``` text
       ┌→ Task A
Goal ──┼→ Task B
       └→ Task C
```

Use when tasks are sufficiently independent.

Example:

``` text
Research Company A
Research Company B
Research Company C
```

These can potentially happen in parallel.

### Key rule

> **Independent tasks → consider parallel execution.**

> **Dependent tasks → execute in the required sequence.**

### Benefit of parallelism

Parallel execution can reduce latency.

------------------------------------------------------------------------

# 14. Planning

**Planning** is determining how to accomplish a goal through a useful
sequence of actions.

Example:

``` text
Goal:
Find why pipeline failed

Plan:
1. Check status
2. If failed → get logs
3. Identify failure category
4. Investigate specific cause
5. Report root cause
```

### Planning vs task decomposition

**Task decomposition:**

> Break the large goal into smaller tasks.

**Planning:**

> Determine how/when those tasks should be accomplished.

### Dynamic planning

An agent can adapt its plan based on observations.

Example:

``` text
Check status
     ↓
SUCCESS
     ↓
No failure investigation needed
     ↓
STOP
```

The agent doesn't blindly execute a previously imagined plan.

### Important nuance

Not every task needs an elaborate plan.

Simple task:

``` text
Check pipeline status
     ↓
Return result
```

Complex task:

``` text
Investigate
   ↓
Decompose
   ↓
Plan
   ↓
Execute
   ↓
Observe
   ↓
Re-plan if necessary
```

------------------------------------------------------------------------

# 15. State

**State** is information about the current situation and progress of an
agent's task.

Example:

``` text
Goal:
Find why pipeline failed

Pipeline status:
FAILED

Log result:
Schema mismatch

Next investigation:
Compare schemas
```

This information helps the agent understand:

> **"Where am I in the task, and what do I already know?"**

### State can include

``` text
Goal
Current task
Previous actions
Tool results
Intermediate findings
Decisions
Progress
```

### Important distinction

State does **not necessarily mean permanent memory**.

A task's state may exist only while the task is running.

------------------------------------------------------------------------

# 16. Human-in-the-Loop (HITL)

Human-in-the-loop means intentionally involving a human in an agent's
decision/action process.

Example:

``` text
Agent
  ↓
Decides:
"Drop production table"
  ↓
High-impact action
  ↓
Human approval
  ↓
Approve?
 ↙      ↘
YES      NO
 ↓        ↓
Execute  Stop/revise
```

### When HITL is useful

Especially for:

-   Destructive actions
-   Production changes
-   Financial transactions
-   Security-sensitive actions
-   High-impact decisions
-   Irreversible operations
-   Compliance-sensitive actions

### Important principle

Not every action requires approval.

Low-risk actions can remain automated:

``` text
Read logs        → automatic
Check status     → automatic
Compare schemas  → automatic
```

High-impact actions may require approval:

``` text
Delete production data → human approval
Modify production      → human approval
```

------------------------------------------------------------------------

# 17. Failure Handling & Retries

Agents operate in systems where tools and external services can fail.

Example:

``` text
get_pipeline_logs()
        ↓
TIMEOUT
```

A reasonable response may be:

``` text
Retry
  ↓
Success
  ↓
Continue
```

### Transient failure

A temporary failure such as:

-   Network timeout
-   Temporary service unavailable
-   Temporary connection issue

may be appropriate for controlled retry.

### Persistent/structural failure

Example:

``` text
Permission denied
```

Repeated retries may not help.

Better response:

``` text
Permission denied
      ↓
Stop retrying
      ↓
Report / escalate
```

### Retry limits

Avoid unlimited retries.

``` text
Attempt 1 → fail
Attempt 2 → fail
Attempt 3 → fail
      ↓
Stop / escalate
```

### Important principle

> **Not every failure should be retried.**

Consider:

1.  Is the failure likely transient?
2.  Is retrying safe?
3.  Is there a retry limit?

------------------------------------------------------------------------

# 18. Idempotency and Retries

Before retrying an action, consider whether repeating it is safe.

Example:

``` text
get_pipeline_status()
```

is generally a read operation and can usually be repeated safely.

But:

``` text
create_payment()
```

could potentially create duplicate side effects if repeated incorrectly.

### Architectural principle

> **Retry behavior should consider the possibility of repeated side
> effects.**

This is why idempotency matters in agentic systems.

------------------------------------------------------------------------

# 19. Orchestration

**Orchestration** is the coordination of agents, tools, tasks, execution
flow, state, and decisions so the overall goal can be completed.

Example:

``` text
                    USER GOAL
                        ↓
                 ORCHESTRATION
                        ↓
                Pipeline Agent
                        ↓
                  Schema issue
                        ↓
                   Data Agent
                        ↓
                 Find mismatch
                        ↓
                Manager/Supervisor
                        ↓
                  Final result
```

An orchestrator can coordinate:

-   Which agent works
-   Which tool is called
-   When work happens
-   How results are passed
-   How tasks are sequenced or parallelized
-   When the overall task is complete
-   What happens after failure

### Simple mental model

> **Orchestration = who does what, when, and how the results flow
> through the system.**

------------------------------------------------------------------------

# 20. Core Day-1 Mental Model

Put the concepts together:

``` text
                         USER GOAL
                             ↓
                     ┌───────────────┐
                     │    AGENT      │
                     └───────┬───────┘
                             ↓
                         Understand
                             ↓
                    Decompose if needed
                             ↓
                          Plan
                             ↓
                    Choose next action
                             ↓
                          Use Tool
                             ↓
                         Observe
                             ↓
                      Update State
                             ↓
                    Goal complete?
                       ↙         ↘
                     YES          NO
                      ↓            ↓
                    STOP       Re-plan / Act
```

For complex systems:

``` text
                         SUPERVISOR
                        /     |      \
                       ↓      ↓       ↓
                  Pipeline   Data    Infra
                   Agent    Agent    Agent
                       \      |      /
                        \     |     /
                         ↓    ↓    ↓
                         RESULTS
                            ↓
                       FINAL ANSWER
```

------------------------------------------------------------------------

# 🧠 Key Definitions for Revision

  -----------------------------------------------------------------------
  Concept                             Simple meaning
  ----------------------------------- -----------------------------------
  **Agent**                           AI system that works toward a goal
                                      by deciding actions

  **Tool**                            Capability available to an agent

  **Agent loop**                      Decide → Act → Observe → Decide →
                                      Stop

  **Workflow**                        Predefined sequence of actions

  **Autonomy**                        Freedom to decide within
                                      constraints

  **Role**                            Responsibility/behavior assigned to
                                      an agent

  **Specialization**                  Agents focus on specific
                                      responsibilities

  **Handoff**                         Transfer of responsibility between
                                      agents

  **Supervisor**                      Coordinates specialized agents

  **Single-agent**                    One agent handles the task

  **Multi-agent**                     Multiple specialized agents
                                      collaborate

  **Task decomposition**              Break a large goal into smaller
                                      tasks

  **Sequential**                      Tasks execute in dependency order

  **Parallel**                        Independent tasks execute
                                      concurrently

  **Planning**                        Determine how to accomplish the
                                      goal

  **State**                           Current task information and
                                      progress

  **HITL**                            Human approval/participation at
                                      selected points

  **Retry**                           Repeat an operation after a
                                      suitable failure

  **Idempotency**                     Safety of repeating an operation
                                      without unwanted duplicate effects

  **Orchestration**                   Coordination of agents, tools,
                                      tasks, and execution
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 🔥 Architecture Principles to Remember

### Principle 1

> **Use the simplest architecture that can reliably accomplish the
> goal.**

### Principle 2

> **More autonomy is not automatically better.**

### Principle 3

> **More agents are not automatically better.**

### Principle 4

> **Independent tasks can potentially run in parallel.**

### Principle 5

> **Dependent tasks usually need sequential execution.**

### Principle 6

> **Use specialized agents when responsibilities naturally differ.**

### Principle 7

> **Use human approval for high-impact or risky actions.**

### Principle 8

> **Don't retry every failure blindly.**

### Principle 9

> **State represents current task progress; it does not necessarily mean
> permanent memory.**

### Principle 10

> **Agentic behavior comes from dynamically deciding actions based on
> goals, observations, tools, and constraints.**

------------------------------------------------------------------------

# 📝 Day-1 Learning Notes

## What I understand well

-   [ ] AI Agent
-   [ ] Agent vs Workflow
-   [ ] Agent Loop
-   [ ] Tools
-   [ ] Autonomy
-   [ ] Roles
-   [ ] Handoffs
-   [ ] Supervisor
-   [ ] Single vs Multi-agent
-   [ ] Task Decomposition
-   [ ] Sequential vs Parallel
-   [ ] Planning
-   [ ] State
-   [ ] Human-in-the-loop
-   [ ] Failure Handling
-   [ ] Orchestration

## Concepts to revise at EOD

> Add topics here that feel less than 8/10 after the day's assessment.

------------------------------------------------------------------------

# 🎯 Day-1 Exam Thinking Framework

When given an architecture scenario, ask:

### 1. What is the goal?

What is the system actually trying to accomplish?

### 2. Is the task simple or complex?

If simple, avoid unnecessary agent complexity.

### 3. Does the task need dynamic decision-making?

If the sequence is fixed, a workflow may be more appropriate.

### 4. What tools/capabilities are required?

Only provide relevant capabilities.

### 5. How much autonomy is appropriate?

Higher-risk actions should have stronger constraints.

### 6. Can tasks run independently?

If yes, consider parallel execution.

### 7. Does one task depend on another?

If yes, sequence them appropriately.

### 8. Does specialization provide real value?

If not, a single agent may be preferable.

### 9. Is human approval required?

Consider risk, reversibility, and impact.

### 10. What happens when something fails?

Consider retryability, retry limits, escalation, and side effects.

------------------------------------------------------------------------

# 📚 Day-1 Status

**First-pass learning:** ✅ Complete

**Scenario practice:** ⏳ Next

**Revision:** ⏳ EOD

**Mini assessment:** ⏳ EOD

**Target understanding:** 8+/10

------------------------------------------------------------------------

## 🔜 Next Step

After saving this file to GitHub, continue with:

> **CCAF Day 1 --- Architecture Scenario Practice**

The goal is no longer to learn new definitions.

The goal is to **apply the concepts like an architect**.
