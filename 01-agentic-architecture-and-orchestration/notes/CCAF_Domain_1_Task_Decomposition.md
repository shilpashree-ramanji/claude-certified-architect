# CCAF Domain 1 — Task Decomposition Strategies

## What You Need to Know

Task decomposition is how you break complex work into pieces an agentic system can handle.

The CCAF exam focuses on two patterns:

1. **Fixed Sequential Pipelines (Prompt Chaining)**
2. **Dynamic Adaptive Decomposition**

It also tests **attention dilution**, a failure mode that occurs when decomposition is too shallow and an agent processes too many items in a single pass.

---

# 1. Fixed Sequential Pipelines — Prompt Chaining

A fixed sequential pipeline breaks work into predetermined steps that execute in order.

```text
Step 1
  ↓
Step 2
  ↓
Step 3
  ↓
Final Result
```

Each step takes the output of the previous step as input.

### How it works

The workflow is defined in advance.

```text
Step 1 → output → Step 2 → output → Step 3
```

The sequence does **not** change based on intermediate results.

### Example — Code Review Pipeline

1. For each file, run a local analysis pass:
   - Style
   - Bugs
   - Complexity
2. After all local passes, run a cross-file integration pass:
   - Data flow
   - API consistency
   - Import chains
3. Compile results into a unified review report.

### Best for

Predictable, structured tasks where the steps are known in advance:

- Code reviews
- Document processing
- Data extraction
- Compliance checks

### Advantages

- Consistent
- Reliable
- Easy to debug
- Easy to monitor
- Same input follows the same path

### Limitation

A fixed pipeline cannot adapt when intermediate findings should change the later workflow.

Example:

```text
Step 1 → Step 2 discovers something unexpected
                 ↓
        Step 3 should change
                 ↓
        Fixed pipeline cannot adapt
```

### Memory Aid

> **Fixed pipeline = known steps, predictable work, consistent execution.**

---

# 2. Dynamic Adaptive Decomposition

Dynamic adaptive decomposition generates subtasks based on what is discovered during execution.

The plan evolves as the agent learns more.

```text
High-level goal
      ↓
Initial investigation
      ↓
Discover information
      ↓
Create / modify plan
      ↓
Execute
      ↓
Discover more
      ↓
Adapt again
```

### Example — Adding Tests to a Legacy Codebase

1. Map the codebase structure.
2. Identify high-impact areas.
3. Create a prioritized test plan.
4. Start writing tests.
5. Discover that Module A depends on Module B, which has no tests.
6. Reprioritize and test Module B first.
7. Continue adapting as new dependencies and issues emerge.

### Best for

Open-ended tasks where the full scope is not known at the beginning:

- Legacy system exploration
- Security audits
- Research projects
- Debugging unfamiliar codebases

### Advantages

- Adapts to unexpected complexity
- Can discover new issues
- Can change priorities
- Better suited to open-ended investigations

### Limitations

- Less predictable
- Execution time can vary
- Resource usage is harder to estimate
- More difficult to debug

### Memory Aid

> **Dynamic decomposition = discover → adapt → continue.**

---

# 3. Choosing the Right Pattern

| Task characteristic | Pattern | Reason |
|---|---|---|
| Steps known in advance | Fixed pipeline | Consistency and reliability |
| Open-ended / unknown scope | Dynamic decomposition | Adaptability |
| Multi-file code review | Fixed pipeline | Per-file analysis + integration is predictable |
| Legacy codebase exploration | Dynamic decomposition | Dependencies and issues emerge |
| Document extraction | Fixed pipeline | Fields and format are predetermined |
| Debugging unfamiliar system | Dynamic decomposition | Root cause is unknown |

## Exam Rule

> **Match the decomposition strategy to the characteristics of the task, not to which approach sounds more sophisticated.**

### Exam Trap

A question may present:

- A fixed pipeline for an open-ended investigation, or
- Dynamic decomposition for a structured processing task.

The correct choice depends on whether the work is **predictable and predefined** or **unknown and adaptive**.

---

# 4. Attention Dilution

**Attention dilution** occurs when an agent processes too many items in a single pass.

The result can be inconsistent depth.

```text
Too many items
      ↓
Attention spread across items
      ↓
Some items receive less analysis
      ↓
Inconsistent results
```

### Symptoms

- Detailed feedback for early files but shallow feedback for later files
- A pattern is flagged in one file but approved in another identical case
- Obvious bugs are missed in some files
- Minor style issues receive attention while important issues are overlooked

### Why it happens

The model has to distribute its attention across all items in the context.

When there are too many items:

> **Attention per item decreases.**

Early items can receive disproportionate attention while later items may be skimmed.

---

# 5. Fixing Attention Dilution — Multi-Pass Architecture

The structural fix is a **multi-pass architecture**.

## Pass 1 — Per-Item Local Analysis

Analyze each item independently.

```text
File 1 → Analysis
File 2 → Analysis
File 3 → Analysis
...
File 14 → Analysis
```

Each file gets a dedicated analysis pass.

## Pass 2 — Cross-Item Integration

After local analysis is complete:

```text
All local findings
       ↓
Cross-file integration
       ↓
Check relationships
```

Look for:

- Data flow issues
- Cross-file dependencies
- Inconsistent pattern usage
- API consistency
- Import chains

### Why this works

Local passes provide dedicated attention to each item.

The integration pass focuses specifically on relationships between items.

---

# 6. Practical Example — 14-File Code Review

Suppose an agent reviews 14 files in one pass.

### Observed result

Files 1–5:

- Detailed feedback
- Specific line references
- Bug identification
- Improvement suggestions

Files 6–9:

- Moderate feedback
- Some issues identified
- Less thorough analysis

Files 10–14:

- Superficial feedback
- Obvious null-pointer bugs missed
- SQL injection vulnerabilities missed

Another inconsistency:

```text
File 3:
forEach loop → flagged as inefficient

File 11:
Identical forEach loop → no comment
```

This is **attention dilution**.

---

# 7. Structural Fix

Do not solve the problem merely by:

- Using a larger context window
- Writing a longer prompt
- Assuming a better model will fix it

The source's recommended fix is **structural**:

```text
14 files
   ↓
14 per-file analysis passes
   ↓
Local findings
   ↓
Cross-file integration pass
   ↓
Unified review
```

The local passes give each file dedicated attention.

The integration pass catches cross-file issues such as inconsistent patterns and data-flow relationships.

---

# 8. Fixed Pipeline vs Dynamic Decomposition

### Fixed

```text
Known workflow
     ↓
Step 1
     ↓
Step 2
     ↓
Step 3
     ↓
Done
```

Use when the workflow is predictable.

### Dynamic

```text
Goal
 ↓
Investigate
 ↓
Discover
 ↓
Replan
 ↓
Investigate
 ↓
Discover
 ↓
Replan
 ↓
Finish
```

Use when the problem is open-ended.

---

# 9. CCAF Exam Traps

### Trap 1 — Using fixed pipelines for unknown investigations

If the root cause or scope is unknown, a rigid predetermined workflow may not adapt sufficiently.

### Trap 2 — Using dynamic decomposition for structured processing

If fields, steps, and output format are already known, dynamic planning adds unnecessary complexity.

### Trap 3 — Ignoring attention dilution

Putting many files/items into one giant pass can lead to inconsistent analysis.

### Trap 4 — Fixing attention dilution only with a bigger context window

The source's recommended solution is structural: use dedicated local passes plus an integration pass.

### Trap 5 — Assuming dynamic is always better

Dynamic decomposition is not automatically superior.

Choose based on the task.

---

# 10. CCAF Decision Framework

Ask:

### Question 1
**Are the steps known in advance?**

Yes → **Fixed sequential pipeline**

### Question 2
**Is the scope unknown and likely to change as we investigate?**

Yes → **Dynamic adaptive decomposition**

### Question 3
**Are there many independent items that need consistent deep analysis?**

Yes → Consider **multi-pass decomposition** to prevent attention dilution.

---

# 11. CCAF Checklist

- [ ] Understand task decomposition
- [ ] Fixed sequential pipelines
- [ ] Prompt chaining
- [ ] Dynamic adaptive decomposition
- [ ] When to use fixed pipelines
- [ ] When to use dynamic decomposition
- [ ] Advantages and limitations of each
- [ ] Understand attention dilution
- [ ] Recognize symptoms of attention dilution
- [ ] Understand multi-pass architecture
- [ ] Per-item local analysis
- [ ] Cross-item integration
- [ ] Match strategy to task characteristics
- [ ] Recognize common exam traps

## One-Line Memory Aid

> **Known workflow → Fixed pipeline. Unknown problem → Dynamic decomposition. Too many items → Multi-pass to prevent attention dilution.**
