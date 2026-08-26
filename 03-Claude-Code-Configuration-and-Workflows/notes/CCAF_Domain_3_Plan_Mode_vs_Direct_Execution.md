# CCAF Domain 3 — Plan Mode vs Direct Execution

## What You Need to Know

Claude Code provides two different ways to approach work:

- **Plan Mode** — investigate and design a plan before making changes.
- **Direct execution** — Claude proceeds directly with the requested work.

The key decision is based on **task complexity, risk, uncertainty, and whether implementation should begin immediately**.

---

# 1. Plan Mode

Plan Mode is used when you want Claude to **understand the problem and design an approach before changing the codebase**.

Think:

```text
Understand
    ↓
Investigate
    ↓
Design a plan
    ↓
Review the plan
    ↓
Implement
```

The important idea is:

> **Plan Mode separates investigation and planning from implementation.**

This is useful when the task is complex or when making the wrong change would be costly.

---

# 2. When to Use Plan Mode

Plan Mode is especially useful for:

- Large refactoring
- Architectural changes
- Changes affecting many files
- Unfamiliar codebases
- Tasks with unclear implementation paths
- High-risk changes
- Changes involving multiple dependencies
- Work where you want to review the approach before implementation

### Example

User asks:

> "Refactor the authentication system to support OAuth without breaking the existing login flow."

This is not a simple one-file edit.

A sensible workflow is:

```text
Explore authentication code
        ↓
Trace dependencies
        ↓
Identify affected files
        ↓
Design migration approach
        ↓
Present plan
        ↓
Implement after approval
```

---

# 3. Direct Execution

Direct execution is appropriate when the task is:

- Small
- Clear
- Low risk
- Well understood
- Easy to verify

Examples:

```text
Fix a typo
Rename a variable
Add a missing import
Change a simple configuration value
Update a single obvious function
```

For a simple task:

```text
User request
    ↓
Understand
    ↓
Make change
    ↓
Verify
```

There is little value in spending a long time planning a one-line change.

---

# 4. The Core Difference

| Plan Mode | Direct Execution |
|---|---|
| Investigate first | Act immediately |
| Produces an implementation plan | Proceeds with implementation |
| Best for complex work | Best for simple work |
| Useful when requirements or architecture are uncertain | Useful when requirements are clear |
| Reduces risk before changes | Faster for straightforward changes |
| Allows plan review before implementation | Changes begin immediately |

### Memory aid

> **Complex or risky → Plan Mode.**

> **Simple and obvious → Direct execution.**

---

# 5. Plan Mode Does Not Mean No Tools

A common misunderstanding is:

> "Plan Mode means Claude does nothing."

That is not the idea.

Claude can investigate the codebase while planning.

For example:

```text
Plan Mode
   ↓
Read relevant files
   ↓
Search for usages
   ↓
Trace dependencies
   ↓
Understand architecture
   ↓
Create implementation plan
```

The distinction is that the investigation is used to **prepare the plan rather than immediately modifying the code**.

---

# 6. Why Planning Helps

Consider a change that appears simple:

> "Replace the old authentication library."

Claude might discover:

```text
auth.ts
   ↓
session.ts
   ↓
middleware.ts
   ↓
API routes
   ↓
tests
   ↓
deployment configuration
```

Changing only `auth.ts` could break the rest of the system.

Plan Mode allows Claude to discover those dependencies before implementation.

---

# 7. Plan Mode for Unfamiliar Codebases

Plan Mode is particularly valuable when Claude does not yet understand the architecture.

Example:

```text
Unknown repository
      ↓
Find entry points
      ↓
Trace relevant modules
      ↓
Understand dependencies
      ↓
Identify risks
      ↓
Create plan
```

This follows the incremental codebase-understanding approach:

> **Discover first, then decide what to change.**

---

# 8. Plan Mode and Multi-File Changes

The more files a task can affect, the more useful planning becomes.

### Small change

```text
Change one function
      ↓
Direct execution
```

### Large change

```text
Change API
   ↓
Database
   ↓
Services
   ↓
Frontend
   ↓
Tests
   ↓
Deployment
```

For the second scenario, planning first reduces the chance of missing dependencies.

---

# 9. Plan Review Before Implementation

One of the main advantages of Plan Mode is that the implementation strategy can be reviewed before changes are made.

Conceptually:

```text
Investigation
      ↓
Plan
      ↓
Human reviews plan
      ↓
Approve / revise
      ↓
Implementation
```

This is especially valuable for:

- Architectural changes
- Security-sensitive changes
- Large refactors
- Database migrations
- Changes with significant operational risk

---

# 10. Direct Execution for Simple Tasks

Do not over-engineer simple requests.

Example:

> "Rename `getUserData` to `getUserProfile` in this file."

If the scope is obvious and the change is easy to verify:

```text
Direct execution
    ↓
Edit
    ↓
Run relevant verification
```

A full planning phase would add unnecessary overhead.

---

# 11. The Decision Framework

Ask four questions.

### Question 1 — Is the task simple?

If yes:

```text
→ Direct execution
```

### Question 2 — Is the task complex?

If yes:

```text
→ Plan Mode
```

### Question 3 — Is the architecture or scope unclear?

If yes:

```text
→ Plan Mode
```

### Question 4 — Would an incorrect change be expensive or risky?

If yes:

```text
→ Plan Mode
```

---

# 12. Practical Examples

## Example A — Fix a typo

```text
Task:
Fix "recieve" → "receive"

Choice:
Direct execution
```

Why?

The change is obvious and low risk.

---

## Example B — Add a new API endpoint

```text
Task:
Add GET /customers/:id
```

If the project structure and API conventions are already obvious, direct execution may be sufficient.

If the architecture is unfamiliar or the endpoint affects authentication, database access, caching, and multiple services, use Plan Mode first.

---

## Example C — Migrate a database schema

```text
Task:
Replace the existing customer schema with a new normalized schema.
```

Choice:

```text
Plan Mode
```

Why?

The change may affect:

- Database migrations
- Queries
- Services
- APIs
- Tests
- Existing data
- Deployment

---

## Example D — Large refactor

```text
Task:
Move the authentication system into a new service.
```

Choice:

```text
Plan Mode
```

Why?

The architecture and dependency graph must be understood before implementation.

---

# 13. Exam Traps

## Trap 1 — Always using Plan Mode

Plan Mode is not automatically better.

For simple, low-risk tasks, it adds unnecessary overhead.

> **Use the least expensive workflow that safely solves the task.**

---

## Trap 2 — Always executing immediately

Direct execution is not ideal for large or risky changes.

If Claude starts changing files before understanding the architecture, it can create inconsistent or incomplete changes.

---

## Trap 3 — Thinking Plan Mode means Claude cannot investigate

Plan Mode is specifically useful for investigation.

Claude can inspect relevant files and understand the system before proposing implementation steps.

---

## Trap 4 — Planning a trivial change

A one-line typo fix does not normally require a detailed architectural plan.

Match the workflow to the complexity of the task.

---

## Trap 5 — Treating the plan as the implementation

A plan describes **what should be done and how**.

It does not mean the implementation has already happened.

```text
Plan
≠
Implementation
```

---

# 14. Practice Scenario

### Scenario

A developer asks Claude Code:

> "We need to replace our legacy payment service with a new provider. The old service is used by several backend modules and integration tests, but I'm not sure exactly where."

What should Claude do first?

### Best approach

Use:

```text
Plan Mode
```

Then:

```text
1. Explore the payment-service usage.
2. Identify dependent modules.
3. Inspect relevant tests.
4. Identify configuration and deployment dependencies.
5. Design the migration plan.
6. Review the plan.
7. Implement the approved changes.
```

### Why?

The scope is uncertain and the change is potentially high risk.

Directly modifying the first file discovered could miss important dependencies.

---

# 15. Quick Reference

| Situation | Preferred approach |
|---|---|
| Typo fix | Direct execution |
| Simple one-file change | Direct execution |
| Clear variable rename | Direct execution |
| Complex multi-file refactor | Plan Mode |
| Unfamiliar codebase | Plan Mode |
| Architectural change | Plan Mode |
| High-risk migration | Plan Mode |
| Unclear dependencies | Plan Mode |
| Straightforward, low-risk request | Direct execution |

---

# 16. CCAF Checklist

- [ ] Understand what Plan Mode is.
- [ ] Understand what direct execution is.
- [ ] Know Plan Mode is useful for investigation before implementation.
- [ ] Know direct execution is faster for simple, clear tasks.
- [ ] Know complexity and risk are key decision factors.
- [ ] Know unfamiliar codebases favor planning first.
- [ ] Know multi-file architectural changes favor planning first.
- [ ] Know Plan Mode does not mean Claude cannot use investigation tools.
- [ ] Know a plan is not the same thing as implementation.
- [ ] Avoid using Plan Mode unnecessarily for trivial changes.

---

# One-Line Memory Aids

> **Plan Mode = understand first, plan second, implement after.**

> **Direct execution = clear task, act immediately.**

> **Complex + risky + uncertain → Plan Mode.**

> **Simple + clear + low risk → Direct execution.**

> **Plan ≠ implementation.**
