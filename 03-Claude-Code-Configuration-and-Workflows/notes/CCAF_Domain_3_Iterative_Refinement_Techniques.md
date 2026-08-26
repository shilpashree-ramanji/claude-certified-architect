# CCAF Domain 3 — Iterative Refinement Techniques

## What You Need to Know

Iterative refinement means improving an agent's output through repeated cycles instead of expecting the first attempt to be perfect.

The basic pattern is:

```text
Initial attempt
      ↓
Evaluate result
      ↓
Identify problems
      ↓
Refine
      ↓
Evaluate again
      ↓
Repeat until the result meets the goal
```

The key idea is:

> **Do not blindly repeat the same attempt. Use feedback from the previous iteration to improve the next one.**

---

# 1. Why Iterative Refinement Matters

Agentic tasks often involve uncertainty.

The first attempt may:

- Miss an edge case
- Produce incomplete results
- Make an incorrect assumption
- Fail a test
- Produce output in the wrong format
- Discover new information that changes the approach

Instead of treating the first attempt as final, the agent evaluates the result and improves it.

---

# 2. The Refinement Loop

A useful mental model is:

```text
┌───────────────┐
│ Initial task  │
└───────┬───────┘
        ↓
┌───────────────┐
│ First attempt │
└───────┬───────┘
        ↓
┌───────────────┐
│ Evaluate      │
└───────┬───────┘
        ↓
┌───────────────┐
│ Find gaps      │
└───────┬───────┘
        ↓
┌───────────────┐
│ Refine        │
└───────┬───────┘
        ↓
   Good enough?
      /       No     Yes
    ↓       ↓
 Refine   Finish
```

The evaluation step is critical.

Without evaluation:

```text
Attempt → Attempt → Attempt
```

can simply repeat the same mistake.

With evaluation:

```text
Attempt → Feedback → Correction → Better result
```

the iterations have a purpose.

---

# 3. Refinement vs Repetition

These are not the same.

### Repetition

```text
Try again
Try again
Try again
```

The agent does not learn anything from the previous attempt.

### Refinement

```text
Try
 ↓
Evaluate
 ↓
Identify specific problem
 ↓
Change approach
 ↓
Try again
```

### Memory aid

> **Repetition repeats. Refinement learns from feedback.**

---

# 4. Where Feedback Comes From

The feedback used for refinement can come from several places.

## Tests

For code:

```text
Implement
   ↓
Run tests
   ↓
Test failure
   ↓
Analyze failure
   ↓
Fix
   ↓
Run tests again
```

Tests provide an objective signal.

---

## Validation

A structured validator can check whether the output meets requirements.

Example:

```text
Agent generates JSON
       ↓
Schema validation
       ↓
Invalid
       ↓
Agent corrects JSON
```

---

## Human Review

For high-risk or subjective work:

```text
Agent creates draft
       ↓
Human reviews
       ↓
Feedback
       ↓
Agent revises
```

Human feedback can be especially useful when correctness cannot be determined purely by automated tests.

---

## Another Agent

A separate evaluator can inspect the first agent's output.

```text
Worker agent
     ↓
Draft
     ↓
Evaluator
     ↓
Feedback
     ↓
Worker refines
```

This creates an evaluator/critic loop.

---

# 5. Evaluator-Optimizer Pattern

One common architecture separates generation from evaluation.

```text
        ┌─────────────┐
        │   Worker    │
        └──────┬──────┘
               ↓
            Output
               ↓
        ┌─────────────┐
        │  Evaluator  │
        └──────┬──────┘
               ↓
            Feedback
               │
               └──────────→ Worker
```

The worker produces the result.

The evaluator checks it against explicit criteria.

The worker then uses the feedback to improve the result.

### Why separate them?

The evaluator has a specific job:

> **Find what is wrong or missing.**

The worker has a different job:

> **Fix the problems and produce the result.**

---

# 6. Define the Evaluation Criteria

Refinement works best when "good" is clearly defined.

Instead of:

```text
Make this better.
```

use criteria such as:

```text
- All tests must pass.
- No security vulnerabilities should remain.
- All required fields must be present.
- API responses must follow the schema.
- The report must answer all five requested questions.
```

Then each iteration can be evaluated against those criteria.

---

# 7. Code Refinement Example

Suppose the task is:

> Add a new authentication feature.

Initial implementation:

```text
Implement authentication
        ↓
Run tests
        ↓
3 tests fail
```

Do not simply restart the entire implementation.

Instead:

```text
Identify failing tests
        ↓
Determine root causes
        ↓
Modify affected code
        ↓
Run tests again
```

Suppose the second run produces:

```text
3 failures
    ↓
1 failure
```

The system has evidence that the refinement improved the result.

Continue until the acceptance criteria are satisfied.

---

# 8. Document Refinement Example

Suppose an agent creates a technical report.

First draft:

```text
Draft report
```

Evaluation finds:

```text
Missing:
- Risk analysis
- Deployment considerations
- Two required references
```

Refinement:

```text
Add risk analysis
Add deployment section
Add references
```

Then evaluate again.

This is much more reliable than asking:

```text
"Write the report again."
```

without explaining what was wrong with the first version.

---

# 9. Refinement With Structured Output

Structured outputs make iterative refinement easier because validation can be automated.

Example:

```text
Agent
 ↓
JSON output
 ↓
Schema validator
 ↓
Invalid
 ↓
Validation errors
 ↓
Agent
 ↓
Corrected JSON
```

For example:

```json
{
  "name": "Alice",
  "age": "thirty"
}
```

A schema may require:

```text
age = integer
```

The validator reports:

```text
age must be an integer
```

The agent can then correct it:

```json
{
  "name": "Alice",
  "age": 30
}
```

---

# 10. Targeted Refinement

A good refinement loop changes only what is necessary.

Suppose:

```text
10 tests
```

are passing and:

```text
2 tests
```

are failing.

Do not automatically rewrite the entire implementation.

Instead:

```text
Identify the 2 failures
        ↓
Trace their causes
        ↓
Make targeted changes
        ↓
Run the relevant tests
        ↓
Run the full suite
```

This reduces unnecessary changes and regression risk.

---

# 11. Refinement and Verification

A refinement loop should normally contain a verification step.

```text
Change
  ↓
Verify
  ↓
Feedback
  ↓
Change
```

Without verification, the agent cannot reliably know whether the refinement worked.

### Important distinction

> **Making a change is not the same as proving the change worked.**

---

# 12. Stop Conditions

An iterative system needs a stopping condition.

Possible stop conditions include:

```text
All tests pass
```

or:

```text
Validation score reaches required threshold
```

or:

```text
All required sections are present
```

or:

```text
Human approves the result
```

Without a stop condition, an agent can continue refining unnecessarily.

---

# 13. Avoiding Unlimited Refinement

A refinement loop should have bounded execution.

Example:

```text
Maximum 3 refinement iterations
```

or:

```text
Stop when all acceptance criteria pass
```

or both:

```text
Stop if:
- criteria pass
OR
- maximum iteration count is reached
```

### Mental model

```text
Refine
  ↓
Evaluate
  ↓
Pass? ── Yes → Finish
  │
  No
  ↓
Iteration limit?
  │
 Yes → Escalate / report failure
  │
 No
  ↓
Refine again
```

---

# 14. Refinement vs Task Decomposition

These concepts are related but different.

## Task decomposition

Break one large task into smaller tasks.

```text
Large task
   ↓
Task A
Task B
Task C
```

## Iterative refinement

Improve the result through feedback.

```text
Initial result
   ↓
Evaluate
   ↓
Improve
   ↓
Evaluate again
```

### Memory aid

> **Decomposition = break the work apart.**

> **Refinement = improve the work repeatedly.**

---

# 15. Refinement vs Planning

Planning happens before implementation.

```text
Understand
   ↓
Plan
   ↓
Implement
```

Refinement happens after an attempt exists.

```text
Attempt
   ↓
Evaluate
   ↓
Improve
```

They can be combined:

```text
Plan
 ↓
Implement
 ↓
Evaluate
 ↓
Refine
 ↓
Verify
```

---

# 16. When Iterative Refinement Is Appropriate

Use iterative refinement when:

- The first result may be incomplete.
- There is a measurable way to evaluate the result.
- Feedback can identify what needs improvement.
- The task benefits from successive improvements.
- The result can be safely tested or validated.

Examples:

- Code generation with tests
- Structured data extraction
- Report generation
- Research synthesis
- Configuration generation
- Refactoring
- Test generation

---

# 17. When Refinement Is Less Useful

If there is no meaningful feedback signal, repeated attempts may not improve quality.

For example:

```text
Try 1 → "Looks okay"
Try 2 → "Looks okay"
Try 3 → "Looks okay"
```

There is no concrete information telling the agent what to change.

A better approach is to define acceptance criteria first.

---

# 18. Common Exam Traps

## Trap 1 — Repeating the same prompt

Simply asking the model to try again does not guarantee improvement.

The next iteration should use feedback from the previous attempt.

> **Refinement requires a feedback signal.**

---

## Trap 2 — Refining without evaluation

If the system does not evaluate the output, it cannot reliably determine what needs to change.

Use:

```text
Generate → Evaluate → Refine
```

not:

```text
Generate → Refine blindly
```

---

## Trap 3 — No stopping condition

An iterative loop without a termination condition can continue indefinitely.

Use:

- Acceptance criteria
- Maximum iterations
- Human approval
- Escalation after repeated failure

---

## Trap 4 — Rewriting everything after a small failure

If only one component fails, targeted refinement is usually better than restarting the entire task.

---

## Trap 5 — Confusing decomposition with refinement

Decomposition creates smaller tasks.

Refinement improves an existing result.

---

# 19. Practice Scenario

### Scenario

An agent generates code for a new feature. The test suite reports four failures.

The team wants the agent to improve the implementation automatically while preventing endless retries.

### Best approach

Use an iterative refinement loop:

```text
1. Generate implementation.
2. Run tests.
3. Analyze failing tests.
4. Make targeted corrections.
5. Run tests again.
6. Repeat while failures remain.
7. Stop when all acceptance criteria pass or the iteration limit is reached.
8. Escalate/report if the limit is reached without success.
```

### Why?

The test suite provides an objective feedback signal, so each iteration can be guided by concrete failures.

---

# 20. Exam Decision Framework

When you see an iterative-refinement question, ask:

### Is there feedback?

```text
No
→ Define evaluation criteria first.

Yes
→ Refinement may be appropriate.
```

### Can the result be measured?

```text
Yes
→ Automate the evaluation where possible.
```

### Can the system stop safely?

```text
No
→ Add an iteration limit or clear stopping condition.
```

### Did the previous iteration reveal a specific problem?

```text
Yes
→ Target the next change at that problem.
```

---

# 21. Quick Reference

| Concept | Meaning |
|---|---|
| Iterative refinement | Repeatedly improve an output using feedback |
| Evaluation | Determine whether the current result meets requirements |
| Feedback | Information about what is wrong, missing, or needs improvement |
| Evaluator | Component that checks the worker's output |
| Stop condition | Rule determining when refinement ends |
| Iteration limit | Maximum number of refinement attempts |
| Targeted refinement | Change only what the evaluation shows is necessary |
| Decomposition | Break a large task into smaller tasks |
| Planning | Decide how to approach the task before implementation |

---

# 22. CCAF Checklist

- [ ] Understand iterative refinement.
- [ ] Know that refinement requires feedback.
- [ ] Know the Generate → Evaluate → Refine loop.
- [ ] Understand evaluator/optimizer architecture.
- [ ] Know why explicit evaluation criteria matter.
- [ ] Know how tests can provide objective feedback.
- [ ] Know how structured validation can drive refinement.
- [ ] Use targeted changes instead of unnecessary rewrites.
- [ ] Define clear stop conditions.
- [ ] Bound the number of iterations.
- [ ] Distinguish refinement from task decomposition.
- [ ] Distinguish refinement from planning.

---

# One-Line Memory Aids

> **Iterative refinement = attempt → evaluate → improve → repeat.**

> **Feedback makes refinement useful.**

> **No evaluation = no reliable refinement.**

> **Target the next change at the previous failure.**

> **Always define a stopping condition.**

> **Decomposition breaks work apart; refinement improves the result.**

> **Planning decides what to do; refinement improves what was done.**
