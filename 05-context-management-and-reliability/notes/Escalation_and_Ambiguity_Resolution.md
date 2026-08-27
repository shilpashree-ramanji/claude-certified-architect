# Escalation & Ambiguity Resolution

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

**Escalation** is the mechanism for moving a task to a higher-level agent, a human, or another workflow when the current agent cannot safely or reliably resolve it.

**Ambiguity resolution** is the process of identifying unclear requirements, conflicting information, or insufficient evidence and determining what clarification or decision is needed.

The core principle is:

> **Do not guess when ambiguity materially affects the outcome.**

A reliable system should distinguish between:

- Problems the current agent can resolve locally
- Problems that need clarification
- Problems that require a coordinator
- Problems that require human judgment

---

## Local Resolution vs Escalation

Not every uncertainty needs to be escalated.

### Resolve locally when:

- The answer can be determined from available evidence.
- The ambiguity is minor and does not change the outcome.
- A documented rule clearly determines the correct action.
- A transient failure can be recovered locally.

Example:

```text
API request times out once
        ↓
Retry locally
        ↓
Success
```

No coordinator intervention is necessary.

### Escalate when:

- Required information is missing.
- Sources conflict.
- The decision has significant consequences.
- The agent lacks permission or authority.
- Repeated recovery attempts fail.
- The available evidence is insufficient to choose safely.

Example:

```text
Two contract sections specify different payment terms
        ↓
Cannot safely choose one
        ↓
Flag conflict
        ↓
Human review
```

---

## Ambiguity Is Not the Same as Error

An error means an operation failed.

Ambiguity means the available information does not clearly determine one answer.

### Error

```text
Database connection timed out.
```

The operation failed.

### Ambiguity

```text
Contract says:
Section 3 → Net 30
Section 8 → Net 60
```

The document was successfully read, but the information conflicts.

The correct response is not to pretend one value is definitely correct.

---

## The Safe Ambiguity Pattern

When ambiguity is material:

```text
Detect ambiguity
      ↓
Identify conflicting / missing information
      ↓
Preserve available evidence
      ↓
Determine whether rules resolve it
      ↓
If not → escalate
```

A structured result might look like:

```json
{
  "status": "ambiguous",
  "issue": "conflicting_payment_terms",
  "values": [
    "Net 30",
    "Net 60"
  ],
  "evidence": [
    "Section 3",
    "Section 8"
  ],
  "requires_review": true
}
```

This gives the next decision-maker enough context to resolve the issue.

---

## Escalation Context

An escalation should contain enough information for the recipient to make a decision without repeating the entire investigation.

A useful escalation package contains:

1. **What was being attempted**
2. **What is ambiguous or failed**
3. **Relevant evidence**
4. **What has already been tried**
5. **Partial results**
6. **Potential alternatives**
7. **Why escalation is necessary**

Example:

```text
Task:
Determine the contract payment terms.

Problem:
Two sections contain conflicting terms.

Evidence:
- Section 3: Net 30
- Section 8: Net 60

Attempted:
Checked the contract for a later amendment.

Result:
No amendment was found.

Alternative:
Human reviewer should determine which clause governs.

Reason for escalation:
The available evidence does not establish a single reliable answer.
```

---

## Escalation Should Be Structured

Avoid vague escalation messages such as:

```text
I don't know. Please handle this.
```

Instead:

```json
{
  "escalation_reason": "conflicting_source_values",
  "task": "determine_payment_terms",
  "evidence": [
    {
      "location": "section_3",
      "value": "Net 30"
    },
    {
      "location": "section_8",
      "value": "Net 60"
    }
  ],
  "attempts": [
    "Checked for amendment"
  ],
  "recommended_action": "human_review"
}
```

Structured escalation allows the coordinator or human reviewer to act quickly.

---

## Ambiguity Resolution Through Clarifying Questions

Sometimes the best escalation target is the **user**.

Suppose the user says:

```text
Deploy the application to production.
```

But the system has two production environments:

```text
production-us
production-eu
```

The agent should not guess.

A good clarification is:

```text
Which production environment should I deploy to:
production-us or production-eu?
```

The question should be:

- Specific
- Minimal
- Actionable
- Focused on the missing decision

---

## Avoid Unnecessary Clarification

Do not ask the user about every tiny ambiguity.

If the answer can be safely determined from established project conventions, resolve it locally.

For example:

```text
User:
Run the tests.

Repository convention:
npm test
```

If the project clearly defines the standard test command, the agent can follow it rather than asking:

```text
Which test command should I use?
```

### Principle

> **Ask only when the ambiguity materially affects the outcome.**

---

## Material vs Non-Material Ambiguity

### Non-material ambiguity

```text
User:
Create a summary of the report.
```

The exact heading style may be unspecified, but the task can proceed.

### Material ambiguity

```text
User:
Delete the production database.
```

If there are multiple production databases, guessing is dangerous.

The agent should clarify:

```text
Which production database do you mean:
customer-db or analytics-db?
```

The higher the impact, the lower the tolerance for guessing.

---

## Confidence and Escalation

Confidence can help determine when escalation is appropriate.

Example:

```text
Confidence ≥ 0.90
→ Proceed

Confidence 0.60–0.89
→ Additional validation

Confidence < 0.60
→ Consider escalation
```

These numbers are illustrative, not universal thresholds.

The important principle is:

> **Confidence can help route decisions, but explicit rules should define when escalation is required.**

For high-risk actions, a high model confidence score should not override a mandatory human-approval rule.

---

## Risk-Based Escalation

Escalation should consider both **uncertainty and impact**.

Consider this matrix:

| Confidence | Risk | Recommended action |
|---|---|---|
| High | Low | Proceed |
| High | High | Follow required controls / approval |
| Low | Low | Validate or clarify |
| Low | High | Escalate |

This prevents the system from treating every uncertain situation identically.

---

## Example: Financial Approval

Suppose an agent is processing a refund.

Policy:

```text
Automatic refunds ≤ $500
Refunds > $500 require manager approval
```

User requests:

```text
Refund $1,000
```

Even if the model is highly confident that the user requested $1,000, it must not automatically approve the refund.

The correct flow is:

```text
Request
 ↓
Policy validation
 ↓
Amount > $500
 ↓
Human approval required
```

This is a **policy-based escalation**, not merely a low-confidence escalation.

---

## Example: Document Analysis

An extraction agent finds:

```text
Invoice total: $1,250
Calculated line-item total: $1,150
```

The discrepancy is material.

The system should:

```text
Detect discrepancy
      ↓
Re-check source
      ↓
Retry if missing line item may explain it
      ↓
Validate again
      ↓
If unresolved → escalate
```

It should not silently choose `$1,250` or `$1,150`.

---

## Example: Web Research

A research agent finds two credible sources with conflicting dates.

```text
Source A → Event date: June 10
Source B → Event date: June 12
```

The agent should:

1. Check whether one source is more authoritative.
2. Check publication/update dates.
3. Look for an official source.
4. If the conflict remains unresolved, report the disagreement and escalate or qualify the result.

A bad response is:

```text
The event is definitely June 10.
```

when the evidence does not justify that certainty.

---

## Example: Codebase Exploration

An agent discovers two implementations of what appears to be the same service:

```text
src/services/order_service.ts
src/legacy/order_service.ts
```

It is unclear which one is active.

Instead of assuming:

```text
src/services/order_service.ts is definitely the active implementation.
```

the agent should investigate:

- Imports
- Callers
- Configuration
- Build references
- Tests
- Deployment configuration

If the evidence still does not resolve the ambiguity:

```text
status: ambiguous
reason: multiple active-looking implementations
recommended_action: human clarification
```

---

## Escalation in Multi-Agent Systems

In a multi-agent architecture:

```text
                  Coordinator
                 /     |                      /      |               Research   Document   Validation
          Agent      Agent      Agent
```

A sub-agent should not necessarily escalate every problem directly to a human.

A common flow is:

```text
Sub-agent encounters ambiguity
          ↓
Local resolution
          ↓
Still unresolved?
          ↓
Structured escalation
          ↓
Coordinator
          ↓
Choose:
- retry
- delegate
- ask user
- continue with partial result
- human review
```

The coordinator acts as the decision point for workflow-level escalation.

---

## Avoiding Workflow Termination

Escalation does **not** automatically mean stopping the entire workflow.

Suppose four research agents are working:

```text
Agent A → Complete
Agent B → Complete
Agent C → Partial result
Agent D → Failed
```

The coordinator can:

```text
Synthesize A + B
Use C's partial evidence
Record D's failure
Decide whether D's missing information is critical
```

Only the affected part needs escalation.

This is different from the anti-pattern of **workflow termination**, where one failure stops everything unnecessarily.

---

## Error Propagation vs Ambiguity Escalation

These concepts are related but different.

### Error propagation

Something failed:

```text
Database timeout
```

The agent reports:

- Failure type
- What was attempted
- Partial results
- Alternative approaches

### Ambiguity escalation

The task completed but the answer is unclear:

```text
Two valid sources disagree.
```

The agent reports:

- The ambiguity
- Conflicting evidence
- What was checked
- Why a reliable decision cannot be made
- What clarification or review is needed

---

## Partial Results During Escalation

Partial results should be preserved whenever they are useful.

Example:

```json
{
  "status": "partial",
  "completed_sections": [
    "customer_profile",
    "order_history"
  ],
  "missing_sections": [
    "payment_history"
  ],
  "reason": "payment_service_unavailable",
  "recommended_action": "retry_payment_history"
}
```

The coordinator does not need to throw away successful work simply because one component failed.

---

## Escalation Anti-Patterns

### 1. Silent suppression

The agent encounters a problem but returns:

```text
success: true
result: []
```

even though it never successfully checked the source.

This hides the problem.

### 2. Workflow termination

One unresolved issue causes the entire workflow to stop even when other work can continue.

### 3. Guessing through ambiguity

The agent chooses an answer without sufficient evidence.

### 4. Over-escalation

The agent asks for human help when it could have resolved the issue locally.

### 5. Unstructured escalation

The agent says:

```text
Something went wrong.
```

without giving useful context.

---

## The Right Balance

A reliable system aims for:

```text
Local recovery
      ↓
Local validation
      ↓
Clarification if needed
      ↓
Coordinator escalation
      ↓
Human review when necessary
```

Not:

```text
Every problem → Human
```

and not:

```text
Every problem → Ignore
```

---

## Practice Scenario 1

A document agent finds two conflicting payment terms:

```text
Section 4 → Net 30
Section 9 → Net 60
```

It searches the document for amendments but finds none.

What should it do?

**A.** Randomly choose Net 30 because it appears first.

**B.** Return Net 60 because it appears later.

**C.** Mark the result as ambiguous, preserve both pieces of evidence, and escalate for clarification or review.

**D.** Return an empty result and mark the task successful.

### Correct Answer

**C**

The evidence does not establish a single reliable answer. The conflict should be surfaced rather than hidden.

---

## Practice Scenario 2

An agent is asked to deploy an application. The project has two production environments, and the user's request does not specify which one.

What is the best action?

**A.** Choose the environment with the most recent deployment.

**B.** Ask a focused clarification question.

**C.** Deploy to both.

**D.** Cancel the entire workflow permanently.

### Correct Answer

**B**

The ambiguity materially affects a consequential action, so the agent should clarify rather than guess.

---

## Practice Scenario 3

Four sub-agents perform research:

```text
A → Complete
B → Complete
C → Partial
D → Timeout
```

What should the coordinator do?

**A.** Terminate the entire workflow.

**B.** Treat all four as failures.

**C.** Preserve A and B, include C's partial results, record D's structured failure, and determine whether D requires recovery.

**D.** Pretend D returned no results.

### Correct Answer

**C**

This preserves useful work while making the failure visible.

---

## Practice Scenario 4

An agent is uncertain about a low-risk formatting decision, but project conventions clearly specify the format.

What should it do?

**A.** Escalate to a human.

**B.** Ignore the project convention.

**C.** Follow the established convention and continue.

**D.** Stop the workflow.

### Correct Answer

**C**

Not every uncertainty requires escalation. Existing authoritative rules can resolve minor ambiguity locally.

---

## Practice Scenario 5

A financial workflow requires manager approval for refunds above $500. The model is 99% confident that a requested refund is $1,000.

What should happen?

**A.** Automatically approve because confidence is high.

**B.** Retry until confidence reaches 100%.

**C.** Follow the approval policy and escalate for manager approval.

**D.** Ignore the amount and process the refund.

### Correct Answer

**C**

A mandatory business control cannot be overridden by model confidence.

---

## Build Pattern

A practical escalation and ambiguity-resolution workflow:

### Step 1 — Detect the issue

Identify:

- Error
- Missing information
- Conflicting evidence
- Permission limitation
- Policy boundary
- Unclear user intent

### Step 2 — Try local resolution

Use:

- Established rules
- Available evidence
- Retry for recoverable transient failures
- Additional validation
- Targeted investigation

### Step 3 — Determine whether the ambiguity is material

Ask:

> **Would choosing incorrectly materially affect the result or action?**

If no → resolve using established conventions.

If yes → clarify or escalate.

### Step 4 — Preserve evidence

Record:

- What was attempted
- Relevant evidence
- Partial results
- Conflicting values
- What remains unresolved

### Step 5 — Escalate structurally

Provide a concise, machine-readable escalation context.

### Step 6 — Coordinator decides

Possible outcomes:

```text
Retry
Delegate
Ask user
Continue with partial result
Human review
Terminate only when necessary
```

### Step 7 — Learn from recurring ambiguity

If the same ambiguity repeatedly occurs, improve:

- System prompts
- Tool descriptions
- Schemas
- Project documentation
- Validation rules
- Workflow design

---

## Exam Memory Aid

Remember:

> **Don't guess when ambiguity matters.**
>
> **Resolve locally when reliable rules exist.**
>
> **Escalate material uncertainty.**
>
> **Preserve evidence and partial results.**
>
> **Never hide failures as successful empty results.**
>
> **Escalation does not automatically mean workflow termination.**

### Quick Decision Rules

| Situation | Best action |
|---|---|
| Clear project convention | Resolve locally |
| Transient timeout | Retry locally |
| Missing information that cannot be recovered | Escalate / `null` |
| Conflicting source values | Surface ambiguity |
| Material user-intent ambiguity | Ask clarification |
| Mandatory approval threshold | Escalate according to policy |
| Partial sub-agent failure | Preserve successful/partial work |
| Repeated unresolved high-risk issue | Human review |
| One failure in independent workflow branch | Do not automatically terminate everything |

### Final Takeaway

**Escalation is not failure.**

A well-designed agent knows when it can safely continue and when it should stop pretending it knows the answer.

The ideal pattern is:

```text
Can I resolve this reliably?
        │
   ┌────┴────┐
  Yes        No
   │          │
Continue   Is clarification/
           escalation needed?
              │
          ┌───┴───┐
         Yes      No
          │        │
      Escalate   Continue
```

The goal is **safe autonomy**: let the agent solve what it can, make uncertainty visible when it matters, and involve the right decision-maker only when necessary.
