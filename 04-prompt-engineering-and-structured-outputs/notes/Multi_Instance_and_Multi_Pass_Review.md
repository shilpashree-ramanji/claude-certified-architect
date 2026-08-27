# Multi-Instance and Multi-Pass Review

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

Multi-instance and multi-pass review are techniques for improving the reliability of AI-generated analysis.

The basic idea is:

> **Do not always rely on a single model pass when the task benefits from independent review or a second verification pass.**

There are two related approaches:

- **Multi-instance review** — use multiple independent model instances to review the same input.
- **Multi-pass review** — have the workflow review or refine an output through multiple sequential passes.

These approaches are useful when a single pass may miss issues, produce false positives, or make inconsistent judgments.

---

## Multi-Instance Review

In a multi-instance design, several independent agents or model calls examine the same input.

Example:

```text
                  ┌── Reviewer A
                  │
Document ─────────┼── Reviewer B
                  │
                  └── Reviewer C
                         ↓
                    Aggregator
                         ↓
                    Final result
```

Each reviewer independently evaluates the same material.

The results are then compared or aggregated.

### Why Use Multiple Instances?

Different model instances may notice different things.

For example:

```text
Reviewer A → Finds security issue
Reviewer B → Finds security issue
Reviewer C → Finds no issue
```

The aggregator can use the agreement pattern as additional evidence.

---

## Independent Review Reduces Single-Pass Risk

A single review can fail because the model:

- Misses an important finding
- Misinterprets evidence
- Produces a false positive
- Overlooks an edge case

Independent reviewers provide multiple opportunities to detect the issue.

However:

> **Multiple instances do not guarantee correctness.**

If all reviewers receive the same misleading prompt or evidence, they can make the same mistake.

---

## Consensus and Aggregation

After multiple reviewers finish, an aggregator can combine their outputs.

Example:

```json
{
  "finding": "Potential SQL injection",
  "votes": {
    "reviewer_a": true,
    "reviewer_b": true,
    "reviewer_c": false
  },
  "consensus": "2_of_3",
  "confidence": "medium"
}
```

The aggregator can then determine whether the finding should:

- Be accepted
- Be rejected
- Receive another review
- Go to a human

### Important

Do not blindly use majority vote for every task.

Some findings are high-risk even if only one reviewer detects them.

For example:

```text
Security vulnerability
Reviewer A → found
Reviewer B → missed
Reviewer C → missed
```

A simple 1/3 vote should not automatically dismiss a potentially critical security issue.

Risk and severity matter.

---

## Multi-Pass Review

Multi-pass review uses sequential stages.

Example:

```text
Pass 1: Generate findings
        ↓
Pass 2: Validate findings
        ↓
Pass 3: Refine / remove false positives
        ↓
Final result
```

The second pass receives the first pass's output and checks it.

### Example

Pass 1:

```text
Finding:
Function may contain SQL injection.
```

Pass 2:

```text
Review the finding against the actual code.
Confirm whether user-controlled input reaches the SQL query.
```

The second pass may confirm or reject the original finding.

---

## Multi-Instance vs Multi-Pass

The easiest way to remember the distinction:

| Technique | Main idea |
|---|---|
| Multi-instance | Multiple independent reviewers |
| Multi-pass | Multiple sequential review stages |

### Multi-instance

```text
Input
 ↓
A ─┐
B ─┼→ Aggregate
C ─┘
```

### Multi-pass

```text
Input
 ↓
Pass 1
 ↓
Pass 2
 ↓
Pass 3
 ↓
Final
```

They can also be combined.

---

## Combining Both Techniques

For a high-value code review:

```text
                 ┌── Reviewer A ──┐
Code ────────────┼── Reviewer B ──┼──→ Aggregate
                 └── Reviewer C ──┘
                                  ↓
                            Validation pass
                                  ↓
                            Final findings
```

This gives:

1. Independent perspectives
2. Aggregation
3. A final validation stage

The tradeoff is additional model calls, latency, and cost.

---

## When Multi-Instance Review Is Useful

Use independent instances when:

- False negatives are costly
- The task involves subjective judgment
- Different perspectives are valuable
- The system needs stronger review coverage
- A single pass has demonstrated inconsistent results

Examples:

- Security review
- Architecture review
- Complex document analysis
- Risk classification
- High-value code review

---

## When Multi-Pass Review Is Useful

Use sequential passes when the second stage can meaningfully critique or validate the first.

Examples:

### Code review

```text
Pass 1 → Find potential bugs
Pass 2 → Verify each finding against source code
```

### Document extraction

```text
Pass 1 → Extract fields
Pass 2 → Validate extracted values
```

### Report generation

```text
Pass 1 → Draft report
Pass 2 → Check factual consistency
Pass 3 → Format final report
```

---

## Review Diversity Matters

Multiple instances are most useful when reviewers are genuinely independent.

If every reviewer receives identical instructions and follows the same deterministic process, their outputs may be highly correlated.

For example:

```text
Reviewer A
Reviewer B
Reviewer C
```

may all repeat the same mistake.

Better designs can introduce meaningful diversity through:

- Different review criteria
- Different focused tasks
- Different evidence views
- Independent analysis before aggregation

The goal is not simply **more calls**.

The goal is **more useful independent coverage**.

---

## Structured Reviewer Output

Each reviewer should return a consistent structure.

Example:

```json
{
  "finding": "Potential SQL injection",
  "severity": "critical",
  "evidence": "User input reaches SQL query without parameterization.",
  "confidence": 0.92,
  "reviewer_id": "security-reviewer-1"
}
```

This makes aggregation much easier.

Instead of parsing free-form responses, the aggregator can compare fields directly.

---

## Multi-Pass Validation

A strong pattern is:

```text
Generate
   ↓
Validate
   ↓
Correct
   ↓
Validate again
```

For example, an invoice extraction system might do:

### Pass 1

Extract:

```json
{
  "line_items": [
    {"amount": 100},
    {"amount": 200}
  ],
  "stated_total": 350
}
```

### Pass 2

Validate:

```text
Calculated total = 300
Stated total = 350
Discrepancy detected.
```

### Pass 3

Re-examine the document and correct the extraction.

The key is that validation provides **specific feedback**, rather than simply asking the model to “try again.”

---

## Avoiding Redundant Passes

Do not automatically add multiple passes to every workflow.

Each additional pass adds:

- Cost
- Latency
- Context
- Operational complexity

Use additional review only when the expected improvement justifies the expense.

A simple task such as:

```text
Convert 10°C to Fahrenheit.
```

does not normally need three independent reviewers.

A high-risk security analysis may justify additional review.

---

## Human Review as the Final Layer

Multi-instance and multi-pass review do not eliminate human oversight.

A useful escalation pattern is:

```text
Single review
     ↓
Low confidence / disagreement
     ↓
Additional model review
     ↓
Still uncertain / high risk
     ↓
Human review
```

This concentrates human effort on cases where automated review remains uncertain.

---

## Confidence and Reviewer Agreement

Agreement can be used as one signal of confidence.

Example:

```text
3/3 reviewers agree
→ Strong agreement

2/3 agree
→ Moderate agreement

1/3 agree
→ Disagreement
```

But agreement is **not proof of correctness**.

A unanimous group can still be wrong.

Therefore, combine agreement with:

- Evidence quality
- Finding severity
- Validation results
- Source verification
- Historical accuracy

---

## Example: Security Review

Suppose a codebase contains a suspicious query.

### Single pass

```text
Reviewer:
"No issue."
```

Potential problem: a false negative.

### Multi-instance

```text
Reviewer A → No issue
Reviewer B → SQL injection
Reviewer C → SQL injection
```

Aggregator:

```text
2/3 reviewers identified a security issue.
```

The finding receives further validation.

### Multi-pass

```text
Pass 1 → Detect suspicious query
Pass 2 → Verify data flow
Pass 3 → Confirm severity
```

This creates stronger evidence than relying on one unverified judgment.

---

## Example: Document Analysis

Suppose a contract contains conflicting payment terms.

### Reviewer outputs

```text
Reviewer A → Net 30
Reviewer B → Net 60
Reviewer C → Conflict
```

A good synthesis should **not simply choose the majority value**.

Instead, it should identify the contradiction:

```json
{
  "conflict_detected": true,
  "values": ["Net 30", "Net 60"],
  "requires_review": true
}
```

This is a key reliability principle:

> **When independent reviewers disagree on important evidence, expose the disagreement instead of hiding it.**

---

## Exam Traps

### Exam Trap 1 — “Use multiple agents for everything”

Wrong.

Multiple reviews have additional cost and latency. Use them where the reliability benefit justifies the expense.

### Exam Trap 2 — Majority vote always determines correctness

Wrong.

A majority can be wrong, and a single reviewer may identify a critical issue that others missed.

### Exam Trap 3 — Multi-instance and multi-pass are identical

Wrong.

- Multi-instance = independent reviewers in parallel.
- Multi-pass = sequential stages.

### Exam Trap 4 — More passes automatically mean higher quality

Wrong.

Extra passes can introduce cost, latency, and even additional opportunities for errors.

### Exam Trap 5 — Ignore disagreement

Wrong.

Disagreement can be an important signal that a finding requires deeper validation or human review.

### Exam Trap 6 — Return unstructured reviewer responses

This makes aggregation difficult.

Use structured reviewer outputs.

---

## Practice Scenario 1

A security-review system uses three independent reviewers. Two identify a critical vulnerability, while one reports no issue.

What should the aggregator do?

**A.** Automatically discard the finding because the majority threshold is not 100%.

**B.** Accept the finding as definitely correct because two reviewers agreed.

**C.** Preserve the finding and evidence, then apply appropriate validation or escalation based on its high severity.

**D.** Ignore all reviewer results and run the same review again indefinitely.

### Correct Answer

**C**

Reviewer agreement is useful evidence, but high-severity findings should not be dismissed simply because one reviewer disagreed. The system should validate the evidence and escalate when appropriate.

---

## Practice Scenario 2

A document extraction workflow first extracts fields and then checks whether the extracted values satisfy business rules.

Which pattern is this?

**A.** Multi-instance review

**B.** Multi-pass review

**C.** Prompt caching

**D.** Batch processing

### Correct Answer

**B**

The workflow performs sequential stages: extraction followed by validation.

---

## Practice Scenario 3

A team wants stronger coverage for a high-risk code review. Three independent agents examine the same code, and their structured findings are sent to an aggregator.

Which technique is being used?

**A.** Multi-pass review

**B.** Multi-instance review

**C.** Context compaction

**D.** Scratchpad delegation

### Correct Answer

**B**

Multiple independent reviewers are examining the same input before their results are aggregated.

---

## Build Pattern

A practical high-reliability review workflow can be built as follows:

### Step 1 — Decide whether additional review is justified

Consider:

- Risk
- Cost of false negatives
- Cost of false positives
- Latency requirements
- Review complexity

### Step 2 — Choose the pattern

Use:

```text
Multi-instance
```

when independent perspectives are valuable.

Use:

```text
Multi-pass
```

when later stages can validate or refine earlier output.

### Step 3 — Use structured outputs

Make reviewer results easy to aggregate.

### Step 4 — Aggregate carefully

Consider:

- Agreement
- Evidence
- Severity
- Confidence
- Contradictions

### Step 5 — Validate important findings

Do not assume consensus equals truth.

### Step 6 — Escalate unresolved high-risk cases

Use human review when automated reviewers remain uncertain or disagree on consequential findings.

---

## Exam Memory Aid

Remember:

> **Multi-instance = multiple independent reviewers.**
>
> **Multi-pass = sequential review stages.**
>
> **Agreement is evidence, not proof.**
>
> **High-risk disagreement should trigger deeper validation or human review.**

### Quick Decision Rules

| Scenario | Best approach |
|---|---|
| Need independent perspectives | Multi-instance |
| Need to validate an initial result | Multi-pass |
| Multiple reviewers disagree | Investigate / validate |
| Critical finding with mixed votes | Do not dismiss automatically |
| Simple low-risk task | Usually single pass |
| Structured aggregation | Use consistent reviewer schema |
| Persistent high-risk uncertainty | Human review |

### Final Takeaway

**Use multiple independent instances when you want independent coverage.**

**Use multiple passes when you want iterative validation or refinement.**

**Do not confuse consensus with correctness.**

The strongest systems combine structured reviewer outputs, evidence-based aggregation, targeted validation, and human escalation for cases that remain uncertain or high risk.
