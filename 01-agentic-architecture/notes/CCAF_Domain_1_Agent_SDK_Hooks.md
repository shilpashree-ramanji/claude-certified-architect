# CCAF Domain 1 — Agent SDK Hooks

## What You Need to Know

Agent SDK hooks inject **deterministic behaviour** into an otherwise probabilistic system.

They sit at the boundary between the model's decisions and the real world, intercepting tool calls and results to:

- Enforce business rules
- Block unsafe or unauthorized actions
- Redirect workflows
- Normalize tool results before the model processes them

## Enforcement Spectrum

| Requirement | Mechanism | Guarantee |
|---|---|---|
| Must be followed 100% of the time | Hooks | Deterministic |
| Preferred, but occasional deviation is acceptable | Prompts | Probabilistic |

**Core rule:** If a single failure has serious financial, legal, regulatory, security, or safety consequences, use programmatic enforcement such as hooks.

---

# 1. Two Types of Hooks

```text
Model decides to use a tool
          ↓
   PreToolUse Hook
          ↓
   Tool executes
          ↓
   PostToolUse Hook
          ↓
 Model processes result
```

## PreToolUse

Runs **before** a tool executes.

It can:

- Block the call
- Modify the call
- Redirect it to another workflow

If the hook blocks the call, the tool does not execute.

**Memory aid:**  
> Pre = before execution = policy enforcement

## PostToolUse

Runs **after** a tool executes but before the model processes the result.

It can:

- Transform tool results
- Normalize data
- Clean or standardize output
- Prepare consistent data for the model

**Memory aid:**  
> Post = after execution = result transformation

---

# 2. PreToolUse Hooks — Policy Enforcement

PreToolUse hooks are the practical mechanism for enforcing prerequisite gates and business rules.

### Refund Threshold

Rule: Refunds above $500 require human approval.

```text
Agent requests process_refund
          ↓
    PreToolUse Hook
          ↓
    Check refund amount
       ↙       ↘
   ≤ $500     > $500
      ↓          ↓
   Allow       Block
               / redirect
                  ↓
             Human review
```

The refund tool never executes when the hook blocks the call.

### Compliance Prerequisite

Rule: International transfers require AML verification.

```text
Agent
 ↓
transfer_funds
 ↓
PreToolUse
 ↓
Has aml_check passed?
    ↙          ↘
  YES           NO
   ↓             ↓
Execute        Block
transfer       transfer
                 ↓
          Complete AML check
```

### Manager Approval

Rule: Discounts above 20% require manager approval.

```text
Agent requests approve_discount
          ↓
    PreToolUse Hook
          ↓
    Discount > 20%?
       ↙       ↘
     NO        YES
      ↓          ↓
   Execute    Approval workflow
                  ↓
             Manager approves
                  ↓
               Execute
```

---

# 3. PostToolUse Hooks — Data Normalization

Different tools and backend systems may return information in different formats.

Example:

```text
get_customer
→ Unix timestamp
→ numeric status code

lookup_order
→ ISO 8601 date
→ English status

check_shipping
→ DD/MM/YYYY
→ single-character status
```

Without normalization, the model must interpret different representations repeatedly.

A PostToolUse hook can normalize them before the model sees them.

Examples:

- Unix timestamps → ISO 8601 dates
- Numeric status codes → human-readable strings
- Currency values → consistent decimal format with currency code
- Regional date formats → one standard format

The model receives consistent data regardless of which backend produced it.

---

# 4. Practical Data Normalization Example

Suppose three tools return:

### `get_customer`

```text
date: 1710489600
status: 200
```

### `lookup_order`

```text
date: 2024-03-15T12:00:00Z
status: active
```

### `check_shipping`

```text
date: 15/03/2024
status: P
```

A PostToolUse hook can normalize the results:

```text
All dates
    ↓
ISO 8601

All status codes
    ↓
Human-readable strings
```

Result:

```text
date: 2024-03-15T12:00:00Z
status: pending
```

The model now receives consistent information.

---

# 5. PreToolUse vs PostToolUse

| Hook | When? | Main purpose |
|---|---|---|
| **PreToolUse** | Before tool execution | Enforce policy, block, modify, or redirect |
| **PostToolUse** | After tool execution | Transform / normalize results |

### Critical exam distinction

```text
PRE  → Tool has NOT executed yet
POST → Tool HAS already executed
```

Therefore:

> **Use PreToolUse to prevent an action.**

> **Use PostToolUse to transform the result of an action that already happened.**

---

# 6. Major Exam Trap

### Wrong approach

Use a PostToolUse hook to block a policy-violating action.

Why wrong?

```text
Tool call
   ↓
Tool executes
   ↓
PostToolUse
   ↓
Too late to prevent the action
```

If the requirement is:

> “Do not allow this action unless condition X is satisfied.”

Use **PreToolUse** because the check must happen before execution.

---

# 7. Hooks vs Prompts

## Scenario: International transfers must pass AML checks

### Prompt approach

> “Always complete AML verification before processing an international transfer.”

This provides guidance, but compliance remains probabilistic.

### Hook approach

```text
transfer_funds
      ↓
PreToolUse
      ↓
AML verified?
      ↓
NO → Block
YES → Execute
```

This provides deterministic programmatic enforcement.

## Scenario: Markdown formatting

Requirement:

> “Responses should use Markdown headers and bullet points.”

A prompt is normally enough. A hook would be unnecessary overhead because occasional formatting deviation is not a serious business risk.

## Scenario: Large refund approval

Requirement:

> “Refunds above $500 require human approval.”

Use a PreToolUse hook:

```text
process_refund
      ↓
PreToolUse
      ↓
amount > $500?
      ↓
YES → Block / human escalation
```

---

# 8. Decision Framework

Ask:

1. **Must this rule be followed 100% of the time?**
2. **Would one failure cause serious financial, legal, regulatory, security, or safety consequences?**
3. **Is it only a preference or formatting guideline?**

If the rule is mandatory and high-consequence, use deterministic programmatic enforcement.

If it is a low-risk preference, prompt guidance is generally sufficient.

**Memory aid:**

> High consequence + mandatory rule → deterministic enforcement.

---

# 9. Connection to Workflow Enforcement

Workflow enforcement and hooks are closely related.

Earlier concept:

```text
Mandatory prerequisite
        ↓
Workflow Gate
        ↓
Action allowed?
```

A hook provides a practical implementation point:

```text
Agent requests tool
        ↓
PreToolUse Hook
        ↓
Check prerequisite
        ↓
Allow / Block / Redirect
```

So:

> **Workflow enforcement = the requirement.**

> **PreToolUse hook = one mechanism for implementing the requirement.**

---

# 10. CCAF Exam Traps

### Trap 1 — Using PostToolUse to block an action

**Wrong.** PostToolUse runs after execution.

### Trap 2 — Using a prompt for a 100% compliance requirement

**Wrong.** Prompts provide probabilistic guidance.

### Trap 3 — Suggesting model-side data transformation instead of PostToolUse

When consistent normalization of heterogeneous tool data is required, PostToolUse provides a deterministic place to normalize results before the model processes them.

### Trap 4 — Confusing hook direction

```text
PreToolUse
→ before execution
→ policy enforcement

PostToolUse
→ after execution
→ result transformation
```

---

# 11. Practice Scenario

An agent occasionally processes international transfers without required compliance checks.

The compliance team requires **100% enforcement** of anti-money laundering checks before any international transfer is executed.

The current system uses prompt instructions that work approximately 95% of the time.

### Options

**A.** Implement a PreToolUse hook that blocks `transfer_funds` until `aml_check` returns a verified pass.

**B.** Add more detailed AML instructions to the system prompt.

**C.** Add a PostToolUse hook that flags completed transfers that skipped AML checks.

**D.** Train the agent with more few-shot examples.

### Correct answer

**A. PreToolUse hook**

Why?

```text
transfer_funds
      ↓
PreToolUse
      ↓
AML check passed?
   ↙        ↘
 YES        NO
  ↓          ↓
Execute     Block
```

The business requires **100% enforcement**, and the transfer must be prevented before it happens.

---

# 12. CCAF Checklist

- [ ] Understand what an Agent SDK hook does
- [ ] Understand deterministic enforcement
- [ ] Know PreToolUse
- [ ] Know PostToolUse
- [ ] Know the direction of each hook
- [ ] Use PreToolUse for policy enforcement
- [ ] Use PostToolUse for result transformation / normalization
- [ ] Understand prerequisite gates
- [ ] Understand hooks vs prompts
- [ ] Know when deterministic enforcement is required
- [ ] Recognize the PostToolUse blocking-action exam trap
- [ ] Recognize the prompt-only 100%-compliance exam trap
- [ ] Understand data normalization with PostToolUse

## One-Line Memory Aid

> **PRE = Prevent / Policy. POST = Process the result.**
