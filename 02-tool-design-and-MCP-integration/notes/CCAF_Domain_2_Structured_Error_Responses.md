# CCAF Domain 2 — Structured Error Responses

## What You Need to Know

When an MCP tool fails, its error response determines whether the agent can recover intelligently.

A generic message such as:

```text
"Operation failed"
```

does not tell the agent:

- What went wrong
- Whether the request can be retried
- What kind of recovery is appropriate
- Whether an alternative workflow is required

The MCP protocol provides the **`isError`** flag to communicate tool failures to the agent.

---

# 1. The Four Error Categories

Every tool failure should be classified so the agent can choose the correct recovery strategy.

1. **Transient**
2. **Validation**
3. **Business**
4. **Permission**

---

# 2. Transient Errors

Transient errors are temporary failures.

Examples:

- Timeout
- Service unavailable
- Rate limit
- Temporary backend outage

The request itself is valid.

### Recovery

> **Retry after a suitable delay.**

Example:

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Service temporarily unavailable"
  }],
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "The order database is experiencing high load. The request is valid and should succeed on retry."
}
```

### Memory aid

> **Transient = system problem → retry.**

---

# 3. Validation Errors

Validation errors occur when the request is malformed or contains invalid input.

Examples:

- Invalid format
- Missing required field
- Out-of-range value
- Incorrect identifier

### Recovery

> **Fix the input, then retry.**

Example:

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Invalid order ID format"
  }],
  "errorCategory": "validation",
  "isRetryable": true,
  "description": "Order ID must be in format #NNNNN. Received: 'order-abc'. Please reformat and retry."
}
```

Notice that `isRetryable` is true, but the agent must **correct the input first**.

### Memory aid

> **Validation = bad input → correct input → retry.**

---

# 4. Business Errors

Business errors occur when the request is technically valid but violates a business rule or policy.

Examples:

- Refund exceeds policy limit
- Transaction violates a business rule
- Operation is not allowed under current policy

### Recovery

> **Do not retry the same request. Use an alternative workflow.**

Example:

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Refund exceeds policy limit"
  }],
  "errorCategory": "business",
  "isRetryable": false,
  "description": "Refund amount exceeds the automatic refund limit. Manager approval is required."
}
```

The same request will continue to violate the same rule.

Typical recovery:

```text
Business error
      ↓
Do NOT retry
      ↓
Alternative workflow
      ↓
Escalation / approval
```

### Memory aid

> **Business = policy problem → alternative path.**

---

# 5. Permission Errors

Permission errors occur when the caller does not have the required authorization.

Examples:

- Access denied
- Insufficient credentials
- Authorization failure
- Service account lacks required access

### Recovery

> **Escalate or use a different authorized principal.**

Example:

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Access denied"
  }],
  "errorCategory": "permission",
  "isRetryable": false,
  "description": "The current service account does not have permission to access financial records. Escalate to a principal with the required access."
}
```

Simply retrying with the same credentials will not solve the problem.

### Memory aid

> **Permission = wrong access → authorized principal / escalation.**

---

# 6. What Does `isRetryable` Really Mean?

`isRetryable` answers:

> **Is there a path to success through retrying?**

It does not mean that the exact same request will always succeed unchanged.

Two categories can have:

```text
isRetryable: true
```

but require different recovery:

### Transient

```text
Temporary outage
      ↓
Retry the request
```

### Validation

```text
Invalid input
      ↓
Correct input
      ↓
Retry
```

Business and permission errors normally use:

```text
isRetryable: false
```

because retrying the same request does not resolve the underlying problem.

### Memory aid

> **Read `isRetryable` first for retry possibility; read `errorCategory` to understand how to recover.**

---

# 7. Access Failure vs Valid Empty Result

This is one of the most important concepts in this topic.

## Access Failure

The tool could not successfully access the data source.

Examples:

- Database timeout
- Authentication failure
- Service unavailable
- Connection failure

The data may exist, but the tool could not check.

The agent may need to retry or escalate.

## Valid Empty Result

The tool successfully queried the data source but found no matching data.

Example:

```text
Customer lookup
      ↓
Database query succeeds
      ↓
No matching customer
      ↓
Empty result
```

This is **not an error**.

The correct answer is:

> No matching customer was found.

Do not retry simply because the result is empty.

---

# 8. Correctly Representing an Empty Result

### Valid empty result

```json
{
  "isError": false,
  "content": [{
    "type": "text",
    "text": "No customer found matching the requested criteria. The query executed successfully but returned no matches."
  }],
  "resultCount": 0
}
```

This tells the agent:

```text
Query succeeded
+
No matches found
=
Valid result
```

### Access failure

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Could not reach customer database"
  }],
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "Connection to the customer database timed out. The query did not execute."
}
```

This tells the agent:

```text
Query did NOT successfully complete
=
Failure
```

---

# 9. Why the Distinction Matters

Consider:

> A customer lookup returns an empty array.

If the system does not distinguish:

```text
No matches
```

from:

```text
Database unavailable
```

the agent may:

```text
Empty result
   ↓
Retry
   ↓
Retry
   ↓
Retry
   ↓
Escalate
```

That wastes resources and creates unnecessary human escalation.

The correct design is:

```text
Successful query + no matches
        ↓
Valid empty result
        ↓
No retry
        ↓
Tell user no results were found
```

---

# 10. Error Propagation in Multi-Agent Systems

In a multi-agent architecture, use **local recovery with selective propagation**.

## Step 1 — Local recovery

A subagent should attempt to recover from failures it can handle locally.

Example:

```text
Search Agent
     ↓
Search timeout
     ↓
Retry locally
```

## Step 2 — Propagate unresolved errors

If local recovery fails:

```text
Search Agent
     ↓
Retries exhausted
     ↓
Report failure
     ↓
Coordinator
```

## Step 3 — Include useful context

Do not simply report:

```text
"Search failed."
```

Instead provide:

```text
"I searched 3 of 5 sources successfully.
Sources 4 and 5 timed out.
Here are the partial results from the
3 successful sources."
```

The coordinator can then make an informed decision.

---

# 11. Two Multi-Agent Anti-Patterns

## Anti-pattern 1 — Silently suppressing errors

Bad:

```text
Subagent fails
     ↓
Returns empty result
     ↓
Coordinator thinks no data exists
```

The coordinator cannot distinguish:

```text
Found nothing
```

from:

```text
Could not search
```

## Anti-pattern 2 — Terminating everything on one failure

A single failed subtask should not automatically terminate an entire workflow if useful partial results are available.

Better:

```text
Agent A → Success
Agent B → Failure
Agent C → Success
       ↓
Coordinator
       ↓
Use partial results
+
Decide how to handle B
```

---

# 12. Complete Error-Recovery Mental Model

```text
Tool call
   ↓
Did it succeed?
   ↓
 ┌───────────────┐
 │               │
YES              NO
 │                │
 ↓                ↓
Result       errorCategory
                ↓
      ┌─────────┼──────────┐
      ↓         ↓          ↓
Transient  Validation  Business/Permission
      ↓         ↓          ↓
   Retry    Fix input   Alternative /
                         escalation
```

And remember:

```text
Successful query + no matches
        ↓
Valid empty result
        ↓
NO retry
```

---

# 13. CCAF Exam Traps

### Trap 1 — Retrying an empty result

**Wrong.**

A successful query with no matches is a valid result.

### Trap 2 — Generic error messages

**Wrong.**

Messages such as `"Operation failed"` do not provide enough information for intelligent recovery.

### Trap 3 — Treating business errors as retryable

**Wrong.**

A policy violation will not disappear through repeated retries.

### Trap 4 — Silently converting subagent failures into empty results

**Wrong.**

The coordinator must know whether the subagent found nothing or failed to obtain the data.

### Trap 5 — Confusing access failure with empty data

Remember:

```text
Access failure → couldn't check
Empty result   → checked successfully, found nothing
```

---

# 14. Practice Scenario

### Scenario

A tool returns an empty array after a customer lookup.

The agent retries three times and then escalates to a human.

Analysis shows that the customer's account simply does not exist.

### What is the root cause?

**Correct answer:**

> The tool does not distinguish between access failures and valid empty results, so the agent treats no matches as a retriable failure.

The query succeeded.

```text
Database query
      ↓
Successful
      ↓
No matching account
      ↓
Valid empty result
      ↓
No retry
      ↓
No escalation
```

---

# 15. CCAF Checklist

- [ ] Understand `isError`
- [ ] Understand structured error metadata
- [ ] Know the four error categories
- [ ] Transient errors
- [ ] Validation errors
- [ ] Business errors
- [ ] Permission errors
- [ ] Understand `isRetryable`
- [ ] Distinguish retryable from non-retryable failures
- [ ] Distinguish access failures from valid empty results
- [ ] Know that empty results are not automatically errors
- [ ] Understand local recovery
- [ ] Understand selective error propagation
- [ ] Preserve partial results
- [ ] Recognize common exam traps

## One-Line Memory Aid

> **Transient → retry. Validation → fix then retry. Business → alternative path. Permission → authorized access. Empty result → success with no matches.**
