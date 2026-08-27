# Error Propagation in Multi-Agent Systems

## Reliability & Context Management

### What You Need to Know

In a multi-agent system, one agent's failure can affect downstream agents.

The goal of **error propagation** is to make failures visible, structured, and actionable so that downstream agents do not mistake incomplete or failed work for successful results.

The core principle is:

> **When an agent fails, it should communicate what failed, what it attempted, what it was able to obtain, and what could be tried next.**

This prevents silent failures from becoming incorrect downstream conclusions.

---

## The Four Elements of Structured Error Context

A useful error context contains four elements:

1. **Failure type**
2. **What was attempted**
3. **Partial results gathered before failure**
4. **Potential alternative approaches**

### Example

Suppose a research agent is asked to search a website for product specifications.

The search fails because the website times out.

A weak response is:

```text
No results found.
```

A structured response is:

```text
Failure type:
Transient timeout

What was attempted:
Searched the product documentation page.

Partial results:
Product name and version were found, but specifications were not retrieved.

Potential alternatives:
Retry the request or search an alternative authoritative source.
```

The downstream coordinator now knows this was an **access/execution failure**, not a successful search with zero results.

---

## Why Error Propagation Matters

Consider this workflow:

```text
Coordinator
     ↓
Research Agent
     ↓
Synthesis Agent
     ↓
Final Answer
```

If the research agent fails silently:

```text
Research Agent
     ↓
[]
     ↓
Synthesis Agent
     ↓
"Nothing was found."
```

The synthesis agent may incorrectly conclude that no information exists.

With structured error propagation:

```text
Research Agent
     ↓
ERROR:
timeout
partial results
retry possible
     ↓
Coordinator
     ↓
Retry / alternative source / partial synthesis
```

The workflow can make an informed decision.

---

# Access Failure vs Valid Empty Result

This is one of the most important distinctions.

## Valid Empty Result

The operation succeeded, but there was nothing to return.

Example:

```text
Database query executed successfully.
Rows returned: 0
```

Structured result:

```json
{
  "status": "success",
  "result_count": 0
}
```

This means:

> **The system successfully checked the source and found no matching data.**

It is **not an error**.

---

## Access / Execution Failure

The system could not successfully perform the operation.

Example:

```text
Database connection timed out.
```

Structured result:

```json
{
  "status": "error",
  "failure_type": "timeout",
  "retryable": true
}
```

This means:

> **We do not know whether matching data exists because the source was not successfully checked.**

### Critical Rule

> **Empty result ≠ failed access.**

Never convert:

```text
Could not access the database.
```

into:

```text
No records found.
```

That is silent suppression.

---

# Silent Suppression Anti-Pattern

**Silent suppression** occurs when an agent encounters a failure but hides it by returning an apparently successful result.

### Example

Actual situation:

```text
Search API authentication failed.
```

Agent returns:

```json
{
  "status": "success",
  "results": []
}
```

The coordinator interprets this as:

```text
The search completed successfully and found nothing.
```

That conclusion is false.

---

## Why Silent Suppression Is Dangerous

It creates a false picture of the world.

```text
Actual:
Source unavailable
      ↓
Suppressed
      ↓
"Nothing found"
      ↓
Incorrect synthesis
      ↓
Incorrect final decision
```

The further the workflow progresses, the harder it becomes to identify the original failure.

### Correct approach

```text
Authentication failure
      ↓
Structured error
      ↓
Coordinator
      ↓
Retry / re-authenticate / escalate
```

---

# Workflow Termination Anti-Pattern

**Workflow termination** occurs when one agent failure causes the entire workflow to stop even though other work can continue.

### Example

A coordinator assigns four independent research tasks:

```text
Agent A → Complete
Agent B → Complete
Agent C → Timeout
Agent D → Complete
```

A poor workflow does:

```text
Agent C fails
     ↓
Terminate entire workflow
```

This throws away useful results from A, B, and D.

### Better approach

```text
A → Complete ─┐
B → Complete ─┼→ Coordinator → Continue
C → Timeout  ─┤
D → Complete ─┘
```

The coordinator should determine whether C's missing result is critical.

If it is not critical, synthesis can continue with a coverage annotation.

---

# Partial Results

An agent may fail after gathering useful information.

Do not discard those results.

Example:

```json
{
  "status": "partial",
  "failure_type": "timeout",
  "attempted": "Search product documentation",
  "partial_results": [
    "Product version: 4.2",
    "Release date: June 2026"
  ],
  "missing": [
    "Configuration limits"
  ],
  "alternatives": [
    "Retry documentation search",
    "Check official API reference"
  ]
}
```

The coordinator can use the partial results while deciding how to recover the missing information.

---

# Coverage Annotations

**Coverage annotations** tell downstream agents how complete the available information is.

Suppose a synthesis agent expects results from four agents:

```text
Agent A → Complete
Agent B → Complete
Agent C → Partial
Agent D → Partial
```

The synthesis agent should know this explicitly.

Example:

```json
{
  "coverage": {
    "total_agents": 4,
    "complete": 2,
    "partial": 2
  },
  "sources": [
    {
      "agent": "A",
      "status": "complete"
    },
    {
      "agent": "B",
      "status": "complete"
    },
    {
      "agent": "C",
      "status": "partial",
      "reason": "document access timeout"
    },
    {
      "agent": "D",
      "status": "partial",
      "reason": "source returned incomplete data"
    }
  ]
}
```

The synthesis agent can then say:

```text
Two agents returned complete results.
Two agents returned partial results due to documented failures.
```

This is much safer than presenting the synthesis as if all four agents succeeded.

---

# Local Recovery for Transient Failures

Some failures are temporary.

Examples:

- Timeout
- Temporary network failure
- Rate limiting
- Temporary service unavailability

For these failures, the agent can often attempt **local recovery** before escalating to the coordinator.

Example:

```text
Agent
 ↓
API timeout
 ↓
Retry locally
 ↓
Success
```

This avoids unnecessary coordination overhead.

---

## When to Retry

Retry when the failure is plausibly transient.

Example:

```text
Failure:
Request timed out.

Retryable:
Yes
```

The agent can retry according to the workflow's retry policy.

---

## When NOT to Retry

Do not blindly retry failures that are unlikely to succeed without a change.

Examples:

### Authentication failure

```text
Invalid credentials
```

Retrying the same request does not fix invalid credentials.

### Authorization / access failure

```text
User is not authorized to access resource.
```

Repeated requests do not grant permission.

### Business-rule failure

```text
Refund exceeds permitted limit.
```

Retrying does not change the business rule.

### Missing information

```text
Required field does not exist in the source document.
```

Retrying cannot create missing evidence.

---

# Local Recovery vs Coordinator Escalation

A useful pattern is:

```text
Agent encounters failure
        ↓
Is it transient?
    ┌───┴───┐
   Yes      No
    ↓        ↓
Retry     Escalate
 locally   / report
    ↓
Success?
 ┌──┴──┐
Yes    No
 ↓      ↓
Continue Escalate
```

The key is to avoid sending every minor transient problem to the coordinator.

---

# Error Context Between Agents

Consider:

```text
Research Agent
      ↓
Validation Agent
      ↓
Synthesis Agent
```

Research Agent returns:

```json
{
  "status": "partial",
  "failure_type": "timeout",
  "attempted": "Search three documentation sources",
  "partial_results": [
    "Source A confirms feature X.",
    "Source B was unavailable."
  ],
  "alternatives": [
    "Retry Source B",
    "Use official API reference"
  ]
}
```

Validation Agent can now distinguish:

```text
Feature X confirmed by Source A.
Source B was not successfully checked.
```

It does not incorrectly treat the missing source as evidence that the feature does not exist.

---

# Error Propagation and Synthesis

A synthesis agent should never blindly aggregate results.

It should understand the **coverage and reliability** of upstream results.

### Poor synthesis

```text
Three agents returned information.
Therefore all three sources were successfully analyzed.
```

### Better synthesis

```text
Two agents returned complete findings.
One agent returned partial findings because its source timed out.
The final synthesis is therefore based on incomplete coverage.
```

This makes uncertainty visible.

---

# Structured Error Example

A standard structure might look like:

```json
{
  "status": "error",
  "failure_type": "transient_timeout",
  "attempted": "Fetch source documentation",
  "partial_results": [
    "Documentation index retrieved."
  ],
  "alternatives": [
    "Retry request",
    "Use alternate documentation endpoint"
  ],
  "retryable": true
}
```

### Field meanings

| Field | Purpose |
|---|---|
| `status` | Success, partial, or error state |
| `failure_type` | Categorizes the failure |
| `attempted` | Explains what the agent tried |
| `partial_results` | Preserves useful work |
| `alternatives` | Suggests recovery paths |
| `retryable` | Helps determine whether local retry is appropriate |

---

# Error Categories

A practical system can classify errors into categories.

## Transient

Examples:

```text
Timeout
Temporary network failure
Rate limit
Temporary service unavailable
```

Typical action:

```text
Local retry
```

---

## Authentication

Examples:

```text
Invalid token
Expired credentials
Authentication failure
```

Typical action:

```text
Do not blindly retry.
Refresh credentials or escalate.
```

---

## Authorization / Access

Examples:

```text
Permission denied
Resource inaccessible
Insufficient privileges
```

Typical action:

```text
Escalate or request access.
```

---

## Business / Policy

Examples:

```text
Refund exceeds limit.
Approval required.
Operation violates business rule.
```

Typical action:

```text
Follow business workflow / human approval.
```

---

## Data / Validation

Examples:

```text
Required field missing.
Totals do not reconcile.
Conflicting values.
```

Typical action:

```text
Validate → retry if evidence can resolve it → otherwise escalate.
```

---

# Error Propagation Is Not Just Logging

Logging says:

```text
ERROR: timeout at 14:03
```

Error propagation provides information to the **next decision-maker**.

For example:

```json
{
  "status": "error",
  "failure_type": "timeout",
  "attempted": "Retrieve customer history",
  "partial_results": [],
  "alternatives": [
    "Retry",
    "Use customer archive"
  ]
}
```

The difference is:

> **Logging records what happened. Error propagation communicates what happened and what the workflow should do about it.**

---

# Designing Error Contracts

In a multi-agent architecture, define an explicit contract for agent results.

For example:

```json
{
  "status": "success | partial | error",
  "data": {},
  "error": {
    "type": "...",
    "message": "...",
    "retryable": false
  },
  "coverage": {},
  "next_actions": []
}
```

This gives the coordinator a predictable interface.

---

# Example: Four-Agent Research Workflow

Suppose the coordinator asks four agents to research a technology.

```text
Agent A → Architecture
Agent B → Security
Agent C → Pricing
Agent D → Documentation
```

Results:

```text
A → Complete
B → Complete
C → Timeout
D → Partial
```

The coordinator should not say:

```text
All research completed successfully.
```

Instead:

```text
Architecture: complete
Security: complete
Pricing: unavailable due to timeout
Documentation: partial due to incomplete source access
```

Then it can decide:

```text
Retry pricing
Continue with architecture/security
Include documentation with coverage warning
```

---

# Error Propagation with Context Degradation

Error propagation and context management are closely connected.

If an error is reported only in a long conversation:

```text
"By the way, the pricing search failed earlier..."
```

the information can be lost during context compression.

A structured state keeps it visible:

```json
{
  "pricing": {
    "status": "failed",
    "failure_type": "timeout",
    "retryable": true
  }
}
```

This makes critical workflow state easier to preserve across:

- Context compaction
- Phase transitions
- Sub-agent handoffs
- Crash recovery

---

# Error Propagation with Summary Injection

When moving between phases, include unresolved errors in the summary.

### Phase 1

```text
Architecture exploration complete.
Validation service could not be inspected because of access failure.
```

### Injected summary

```json
{
  "phase_1_findings": [
    "Master orchestrator invokes validation workflow."
  ],
  "unresolved": [
    {
      "component": "validation service",
      "reason": "access denied"
    }
  ]
}
```

### Phase 2

The next agent knows:

```text
Do not assume the validation service was inspected successfully.
```

This prevents the next phase from converting missing information into false assumptions.

---

# Exam Traps

## Trap 1 — Empty result means success

Not always.

An empty result is a success **only when the operation actually completed successfully and found nothing**.

---

## Trap 2 — Retry every failure

Wrong.

Retry primarily for failures that are plausibly transient or otherwise fixable.

---

## Trap 3 — Authentication failure should be retried repeatedly

Wrong.

Invalid credentials generally require credential recovery, not repeated identical requests.

---

## Trap 4 — One sub-agent failure should terminate the entire workflow

Wrong.

If tasks are independent, preserve successful and partial results and recover only the failed branch.

---

## Trap 5 — Hide failed agents from synthesis

Wrong.

The synthesis agent needs coverage information so it knows what was and was not successfully investigated.

---

## Trap 6 — Error messages only need the error type

Wrong.

A useful error contract should communicate:

1. Failure type
2. What was attempted
3. Partial results
4. Potential alternatives

---

## Trap 7 — Partial results should be discarded

Wrong.

Partial results may still be valuable and should be clearly labelled as partial.

---

## Trap 8 — High-level synthesis can ignore coverage

Wrong.

A synthesis agent should know whether its upstream inputs are complete, partial, or failed.

---

# Practice Scenario 1

A database agent attempts a query.

The database responds successfully:

```text
Rows returned: 0
```

What should the agent report?

**A.** Error: database failure

**B.** Success with a valid empty result

**C.** Authentication failure

**D.** Retry indefinitely

### Correct Answer

**B**

The query successfully executed and returned no matching records.

---

# Practice Scenario 2

A search agent receives:

```text
Connection timeout
```

What should it do first?

**A.** Return an empty successful result.

**B.** Treat it as evidence that no information exists.

**C.** Attempt local recovery if the failure is transient and retryable.

**D.** Terminate the entire multi-agent workflow.

### Correct Answer

**C**

A timeout is typically a transient failure and may be recoverable locally.

---

# Practice Scenario 3

Four agents are researching a topic:

```text
A → Complete
B → Complete
C → Partial
D → Failed
```

What should the coordinator do?

**A.** Terminate everything.

**B.** Mark the entire research task as successful.

**C.** Preserve the complete and partial results, record D's failure, and determine whether D's missing information is critical.

**D.** Treat C and D as valid empty results.

### Correct Answer

**C**

The coordinator needs accurate coverage information to decide how to proceed.

---

# Practice Scenario 4

An agent cannot access a document because of authorization failure.

What is the best response?

**A.** Retry the same request indefinitely.

**B.** Return an empty result.

**C.** Report a structured access failure and indicate that access/authorization must be resolved.

**D.** Assume the document contains no relevant information.

### Correct Answer

**C**

Authorization failure means the document was not successfully checked.

---

# Practice Scenario 5

A synthesis agent receives results from four sub-agents. Two are complete and two are partial.

What should the synthesis agent communicate?

**A.** "All four agents successfully completed their analysis."

**B.** "No reliable synthesis is possible."

**C.** "Two agents returned complete results; two returned partial results with documented reasons."

**D.** Ignore the partial results.

### Correct Answer

**C**

Coverage annotations make the completeness of the synthesis explicit.

---

# Build Pattern

A robust multi-agent error propagation workflow:

### Step 1 — Define an agent result contract

Use explicit states:

```text
success
partial
error
```

### Step 2 — Classify failures

Distinguish:

```text
transient
authentication
authorization
business/policy
data/validation
```

### Step 3 — Preserve the four error elements

Record:

```text
Failure type
What was attempted
Partial results
Potential alternatives
```

### Step 4 — Recover locally when appropriate

For transient failures:

```text
Retry locally
```

before escalating.

### Step 5 — Propagate failures explicitly

Do not convert failures into empty successful results.

### Step 6 — Preserve partial work

A failed branch may still contain useful information.

### Step 7 — Annotate coverage

Tell downstream agents which inputs were:

```text
Complete
Partial
Failed
```

### Step 8 — Let the coordinator decide

The coordinator determines whether to:

```text
Retry
Delegate
Use an alternative
Continue with partial results
Escalate to human
Terminate the affected branch
```

### Step 9 — Preserve error state across phases

Include unresolved failures in:

- Summary injection
- Scratchpad/state
- Structured manifests
- Context handoffs

---

# Exam Memory Aid

Remember the four elements:

> **Failure type**
>
> **What was attempted**
>
> **Partial results**
>
> **Potential alternatives**

Remember the key distinction:

> **Access failure ≠ valid empty result.**

Remember the retry rule:

> **Transient/fixable → retry.**
>
> **Authentication/access/business limitation → resolve the underlying issue or escalate.**

Remember the multi-agent rule:

> **One branch failing does not automatically mean the whole workflow failed.**

Remember the synthesis rule:

> **Always expose coverage.**

---

## Quick Decision Table

| Situation | Recommended action |
|---|---|
| Query succeeds, zero rows | Valid empty result |
| Timeout | Local retry if retryable |
| Rate limit | Follow retry/backoff policy |
| Invalid authentication | Resolve credentials / escalate |
| Permission denied | Resolve access / escalate |
| Business rule violation | Follow policy / approval flow |
| Missing source information | Do not invent; escalate or return `null` |
| One independent agent fails | Preserve other results |
| Partial agent result | Label as partial |
| Multiple upstream results | Include coverage annotations |
| Repeated unresolved failure | Escalate |
| Failure hidden as empty result | Anti-pattern: silent suppression |

---

# Final Takeaway

**Error propagation is about preserving truth across agent boundaries.**

When something fails, the next agent should be able to answer:

```text
What failed?
What did the agent try?
What did it successfully learn?
What remains missing?
Can it be retried?
What alternative exists?
```

A reliable multi-agent workflow therefore looks like:

```text
             Agent
               ↓
          Attempt task
               ↓
            Validate
               ↓
        ┌──────┴──────┐
      Success       Failure
        ↓              ↓
      Result       Classify error
                       ↓
                 Retryable?
                  /       \
                Yes        No
                 ↓          ↓
            Local retry   Escalate
                 ↓
             Validate
                 ↓
              Result
                 ↓
             Coordinator
                 ↓
        Coverage-aware synthesis
```

The ultimate goal is not to eliminate every failure.

It is to ensure that **failures remain visible, recoverable when possible, and correctly represented to every downstream agent**.
