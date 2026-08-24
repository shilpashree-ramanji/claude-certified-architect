# CCAF Domain 1 — Workflow Enforcement & Handoffs

## 1. Workflow Enforcement

**Enforcement** means making sure a required workflow rule is actually followed.

Example:

```text
Verify Customer
      ↓
Issue Refund
```

If verification is mandatory, the system should prevent the refund from being issued before verification.

### Prompt Guidance vs Programmatic Enforcement

**Prompt guidance:**
> “Always verify the customer before issuing a refund.”

This tells the agent what it should do, but does not guarantee compliance.

**Programmatic enforcement:**

```text
Agent wants to issue refund
        ↓
Workflow gate
        ↓
Was customer verified?
     ↙       ↘
   YES        NO
    ↓          ↓
 Allow       Block
 refund      refund
```

**Core principle:** Mandatory workflow steps should be enforced programmatically rather than relying only on model instructions.

---

## 2. Why Programmatic Enforcement Matters

Agents are model-driven. A prompt can guide behavior, but mandatory business rules may require deterministic control.

Example:

```text
Production deployment
        ↓
Approval check
     ↙      ↘
 Approved   Not approved
    ↓           ↓
 Execute      Block / escalate
```

---

## 3. What Is a Handoff?

A **handoff** occurs when one agent transfers responsibility for part of a task to another agent or human.

```text
Support Agent
      ↓
Billing issue discovered
      ↓
Billing Specialist
      ↓
Investigates
      ↓
Returns result
```

**Core principle:** The right agent or human should take responsibility for the next part of the task.

---

## 4. What Should a Handoff Contain?

A useful handoff provides the receiving agent or human with relevant context.

Example:

```text
Customer ID: C123
Issue: Customer was charged twice.
Evidence: Two transactions found for the same order.
User request: Determine whether a refund is required.
Required action: Review the duplicate transactions.
```

**Memory aid:**

> Handoff = responsibility + relevant context + expected outcome

Do not simply say: “Take over.”

---

## 5. Context in Handoffs

```text
Too little context
        ↓
Receiver cannot work effectively

Too much irrelevant context
        ↓
More complexity / distraction

Relevant context
        ↓
Focused specialist
```

Pass the context required to continue the task correctly.

---

## 6. Workflow Enforcement + Handoff

These concepts often work together.

Example:

```text
Support Agent
      ↓
Check refund amount
      ↓
Above authorization limit
      ↓
Workflow rule
      ↓
Cannot process automatically
      ↓
Handoff to authorized agent / human
```

The handoff should include the relevant evidence and reason for escalation.

---

## 7. Programmatic Gates

A **gate** is a condition that must be satisfied before a protected action can proceed.

```text
Customer Verification
        ↓
       GATE
        ↓
Verified?
   ↙       ↘
 YES       NO
  ↓         ↓
Refund     Block
```

Other examples:

```text
Code Review
     ↓
Tests Passed
     ↓
Approval Gate
     ↓
Production Deployment
```

---

## 8. When Should a Handoff Happen?

### Specialization
The current agent reaches a task outside its expertise.

### Authorization Boundary
The current agent does not have permission to perform the action.

### Safety / Risk Boundary
The action is too sensitive for automatic execution.

### Workflow Boundary
The next stage belongs to another specialist or process.

---

## 9. Handoff vs Continuing

Do not hand off merely because another agent exists.

Consider:

- Who owns the next task?
- Does the current agent have the required capability?
- Does another specialist have better expertise?
- Is the current agent authorized?
- Is human approval required?

The goal is controlled delegation.

---

## 10. Prompt vs Programmatic Enforcement

| Approach | Meaning |
|---|---|
| Prompt instruction | Guides the model |
| Programmatic gate | Enforces a required condition |
| Handoff | Transfers responsibility |

### Memory aid

> **Prompt = guide**  
> **Gate = enforce**  
> **Handoff = transfer responsibility**

---

## 11. CCAF Exam Traps

### Trap 1 — Prompt-only enforcement

A system prompt saying “always verify” does not by itself guarantee that the verification step cannot be skipped.

### Trap 2 — Handoff without context

Do not simply send a task to another agent and expect it to know everything.

### Trap 3 — Handoff to an unauthorized agent

The receiving agent must have the required capability and authorization.

### Trap 4 — Unnecessary handoffs

Use handoffs when specialization, authorization, safety, or workflow boundaries justify them.

---

## 12. End-to-End Example

```text
User
 ↓
Support Agent
 ↓
Check customer
 ↓
Check refund amount
 ↓
Authorization check
 ↓
 ┌─────────────────┐
 ↓                 ↓
Within limit     Exceeds limit
 ↓                 ↓
Process refund    Handoff
                   ↓
             Authorized Agent
                   ↓
                 Review
                   ↓
                Approval
                   ↓
             Process refund
```

---

## 13. CCAF Checklist

- [ ] Workflow enforcement
- [ ] Prompt guidance vs programmatic enforcement
- [ ] Prerequisite gates
- [ ] Deterministic controls
- [ ] What a handoff is
- [ ] When to hand off
- [ ] Context during handoff
- [ ] Expected outcome / next action
- [ ] Specialization boundaries
- [ ] Authorization boundaries
- [ ] Human escalation
- [ ] Handoff vs unnecessary agent switching

## One-Line Memory Aid

> **Mandatory rule → enforce with a gate; new responsibility → hand off with the right context.**
