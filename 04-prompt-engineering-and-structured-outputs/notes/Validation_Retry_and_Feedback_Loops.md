# Validation, Retry, and Feedback Loops

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

Production extraction systems fail. Documents can contain unexpected formats, inconsistent values, missing information, misplaced fields, or numerical discrepancies.

The goal is not to assume failures will never happen. The goal is to build a **self-correcting validation and retry workflow**.

The core pattern is:

> **Generate → Validate → Identify the specific error → Retry with feedback → Validate again → Escalate when retry cannot help**

The Claude Certification Guide emphasizes that the retry should include three things:

1. The original document
2. The failed extraction
3. The specific validation error

This gives the model the source, its previous attempt, and a precise explanation of what needs to be corrected. citeturn0search0

---

## Retry with Error Feedback

A naive retry looks like:

```text
Try again.
```

This is weak because the model does not know what was wrong with its previous answer.

A better retry contains concrete feedback:

```text
Original document:
[document]

Your extraction:
{
  "line_items": [...],
  "stated_total": 500
}

Validation error:
Line items sum to £450 but stated_total is £500.

Please re-extract the line items and verify the total.
```

The model can now target the specific problem instead of blindly repeating the same extraction. citeturn0search0

### Mental Model

```text
Original source
      ↓
Initial extraction
      ↓
Validation
      ↓
Error?
  ┌───┴───┐
 No      Yes
 ↓        ↓
Success  Error feedback
          ↓
        Retry
          ↓
       Validate
```

---

## The Three Pieces of Retry Context

Every corrective retry should preserve:

### 1. Original document

The model needs access to the source so it can re-examine the evidence.

### 2. Failed extraction

The model needs to see what it previously produced.

### 3. Specific validation error

The model needs to know exactly what failed.

### The Formula

> **Retry context = Original source + Failed output + Specific error**

This is one of the most important exam patterns for this topic. citeturn0search2

---

## The Retry Effectiveness Boundary

Retries are **not universally useful**.

### Retry when the error is fixable

Retries can correct problems such as:

- Format mismatches
- Incorrect date formats
- Currency formatting issues
- Values placed in the wrong fields
- Incorrect nesting
- Mathematical inconsistencies
- Missed line items that are actually present in the source

For example:

```text
Validation:
calculated_total = £450
stated_total = £500

Error:
The line items do not reconcile with the stated total.
```

The model can re-examine the document and potentially discover a missed line item.

### Do NOT retry when the information is genuinely absent

Suppose the document says:

```text
Customer: John Smith
Department: [not provided]
```

The requested `department` value does not exist in the source.

Retrying cannot manufacture reliable information.

The correct response is to:

- Return `null` if the schema allows it, or
- Flag the extraction for human review.

The exam explicitly tests this boundary. citeturn0search0

---

## Fixable vs Unfixable Errors

| Error | Retry? | Reason |
|---|---|---|
| Wrong date format | Yes | Model can reformat it |
| Wrong JSON structure | Yes | Output structure can be corrected |
| Value in wrong field | Yes | Model can relocate the value |
| Incorrect line-item total | Yes | Model can re-check source evidence |
| Missing field that exists in source | Yes | Model can re-examine the document |
| Information absent from source | No | Retry cannot create missing evidence |
| Information only available in an unavailable external source | No | Required evidence is unavailable |
| Knowledge outside the model's available information | No | Retry does not add knowledge |

### Exam Rule

> **Ask: “Can the model fix this by looking again at the available evidence?”**

If yes → retry with targeted feedback.

If no → stop retrying and use an alternative such as human review or `null`.

---

## Schema Validation vs Semantic Validation

This distinction is extremely important.

### Schema/Syntax Errors

These involve the **shape** of the output:

- Invalid JSON
- Missing required fields
- Wrong data types
- Invalid enum values
- Incorrect nesting

Structured tool use / JSON Schema can address these structural requirements. citeturn0search0

### Semantic Validation Errors

These involve the **meaning or correctness** of the values:

- Totals do not reconcile
- Dates are in the wrong order
- A value was placed in the wrong field
- Two extracted values contradict each other
- A calculated value does not match the source

These require additional validation logic.

### Key Distinction

> **Schema validation checks whether the output has the right shape. Semantic validation checks whether the output is actually correct.**

Tool use can enforce schema conformance, but semantic business rules still require validation logic. citeturn0search0turn0search2

---

## Pydantic as a Validation Layer

In Python pipelines, Pydantic can enforce both structural and semantic rules.

For example:

```python
from pydantic import BaseModel, model_validator

class LineItem(BaseModel):
    description: str
    amount: float

class Invoice(BaseModel):
    line_items: list[LineItem]
    stated_total: float

    @model_validator(mode="after")
    def totals_must_match(self):
        calculated = round(
            sum(item.amount for item in self.line_items), 2
        )

        if abs(calculated - self.stated_total) > 0.01:
            raise ValueError(
                f"line items sum to {calculated} "
                f"but stated_total is {self.stated_total}"
            )

        return self
```

The important concept is not the specific library syntax.

The important concept is:

```text
Extraction
   ↓
Validation layer
   ↓
Specific error
   ↓
Retry prompt
```

Pydantic can provide machine-readable validation errors that can be fed directly into the retry loop. citeturn0search0

---

## Self-Correction Fields

The extraction schema itself can contain fields that make validation easier.

### calculated_total vs stated_total

Instead of extracting only:

```json
{
  "total": 500
}
```

extract:

```json
{
  "calculated_total": 450,
  "stated_total": 500,
  "total_discrepancy": true
}
```

Now the system can immediately identify a discrepancy.

### Why this helps

The model is explicitly asked to:

1. Extract the individual values.
2. Calculate the total.
3. Capture the total stated by the document.
4. Compare them.
5. Flag a discrepancy.

This creates a built-in self-check.

---

## conflict_detected

Documents can contain contradictory information.

For example:

```text
Section A:
Payment terms: Net 30

Section B:
Payment terms: Net 60
```

Instead of silently selecting one value:

```json
{
  "payment_terms": "Net 30"
}
```

use a structure that makes the conflict visible:

```json
{
  "payment_terms": "Net 30",
  "conflict_detected": true,
  "conflict_details": [
    "Document also states Net 60 in Section B."
  ]
}
```

The important principle is:

> **Expose uncertainty and conflicts instead of silently choosing an interpretation.**

The certification material specifically highlights `conflict_detected` as a self-correction pattern. citeturn0search0

---

## detected_pattern for Feedback Loops

For code review and analysis systems, add a `detected_pattern` field to findings.

Example:

```json
{
  "finding": "Potential SQL injection vulnerability",
  "severity": "critical",
  "detected_pattern": "string concatenation in SQL query",
  "file": "user_service.py",
  "line": 42
}
```

Now you can track which patterns repeatedly generate findings.

Suppose developers repeatedly dismiss findings caused by:

```text
variable shadowing in nested scope
```

That suggests the corresponding detection rule or prompt may need refinement.

This creates a feedback loop:

```text
Detect
  ↓
Review
  ↓
Dismiss / Accept
  ↓
Analyse detected_pattern
  ↓
Refine prompt or criteria
  ↓
Test again
  ↓
Deploy
```

The goal is not just to retry individual outputs. It is to use production feedback to improve the system over time. citeturn0search0

---

## Validation as a Quality Gate

Validation should happen **before** an output is passed downstream.

Example:

```text
Document
   ↓
Extraction Agent
   ↓
Structured Output
   ↓
Validation
   ↓
┌───────────────┴───────────────┐
Valid                         Invalid
 ↓                               ↓
Downstream                     Diagnose
processing                       ↓
                               Retry?
                            ┌────┴────┐
                           Yes        No
                            ↓          ↓
                          Retry      Human
                            ↓         Review
                       Validation
```

This prevents invalid output from silently propagating through the rest of the workflow.

---

## Retry Limits

Retries should not continue indefinitely.

A practical workflow should define a retry boundary.

For example:

```text
Attempt 1 → validation fails
Attempt 2 → validation fails
Attempt 3 → validation fails
             ↓
        Stop retrying
             ↓
       Escalate / review
```

The certification material identifies repeated failures as a reason to escalate rather than continuing indefinitely. citeturn0search2

### Why?

Infinite retries can:

- Waste tokens
- Increase latency
- Repeat the same mistake
- Create unnecessary costs
- Delay human intervention

---

## Feedback Should Be Specific

Compare these two messages.

### Bad

```text
Validation failed. Try again.
```

### Good

```text
Validation failed:
line_items[2].amount is missing.
The document shows “Widget C — £125” on page 2.
Please re-extract the line items and include Widget C.
```

The second message provides an actionable correction target.

### Principle

> **The feedback should tell the model what is wrong, where it is wrong, and what evidence it should re-check.**

---

## Prompt Chaining and Validation Gates

Validation works naturally inside a multi-step prompt chain.

Example:

```text
Step 1: Extract
      ↓
Step 2: Validate
      ↓
Step 3: Retry if necessary
      ↓
Step 4: Format
      ↓
Step 5: Synthesize
```

The advantage is that intermediate outputs can be validated before the next step consumes them.

The certification guide describes prompt chaining as useful when a task has distinct phases and notes that intermediate results can be validated before proceeding. citeturn0search2

---

## Exam Traps

### Exam Trap 1 — “Just try again”

A retry without specific feedback often reproduces the same mistake.

**Correct:** Include the original source, failed extraction, and specific validation error.

### Exam Trap 2 — Retry every failure

Not every failure is fixable.

If the information is genuinely absent from the source, retrying cannot create it.

### Exam Trap 3 — Rely only on JSON Schema

Schema validation catches structural problems but not semantic problems such as:

```text
calculated_total != stated_total
```

Semantic validation requires additional validation logic.

### Exam Trap 4 — Treat Pydantic as unnecessary once a schema exists

A schema can enforce structure. Pydantic/custom validators can enforce semantic business rules such as arithmetic or date ordering.

### Exam Trap 5 — Retry indefinitely

Repeated failures should eventually lead to escalation or human review.

### Exam Trap 6 — Ignore production feedback

Fields such as `detected_pattern` can reveal which findings are repeatedly dismissed and therefore which prompts need refinement.

---

## Practice Scenario

An invoice extraction pipeline produces this result:

```json
{
  "line_items": [
    {"description": "Widget A", "amount": 150},
    {"description": "Widget B", "amount": 300}
  ],
  "stated_total": 500
}
```

Validation calculates:

```text
150 + 300 = 450
```

The document actually contains a third line item:

```text
Widget C — £50
```

What should the system do?

**A.** Immediately escalate to a human because the extraction failed.

**B.** Retry with the original document, failed extraction, and the specific discrepancy error.

**C.** Ignore the discrepancy because the schema is valid JSON.

**D.** Retry with only the message “Please try again.”

### Correct Answer

**B**

The error is potentially fixable because the missing information exists in the original document. The retry should include the **original document + failed extraction + specific validation error**.

---

## Second Scenario

An extraction requires a `department` field.

The source document contains no department information anywhere.

What should the system do?

**A.** Retry until the model produces a department.

**B.** Ask the model to infer the department from the company name.

**C.** Return `null` if permitted or flag the extraction for human review.

**D.** Generate a likely department based on previous documents.

### Correct Answer

**C**

The required information is absent from the source. Retrying cannot create evidence that does not exist. citeturn0search0

---

## Build Pattern

A strong validation-retry system follows these steps:

### Step 1 — Generate structured output

Use a schema to enforce the expected structure.

### Step 2 — Run semantic validation

Check:

- Field completeness
- Numerical consistency
- Enum validity
- Date ordering
- Cross-field relationships
- Contradictions

### Step 3 — Produce specific errors

Avoid:

```text
Validation failed.
```

Prefer:

```text
line_items sum to £450 but stated_total is £500.
```

### Step 4 — Determine whether retry can help

Ask:

> Is the required information available in the source?

If yes → retry.

If no → stop and escalate or return `null`.

### Step 5 — Retry with complete feedback

Send:

```text
Original document
+
Failed extraction
+
Specific validation error
```

### Step 6 — Revalidate

Do not assume the retry is correct.

Run the same validation rules again.

### Step 7 — Escalate repeated failures

If the output continues to fail, stop retrying and route to human review or another recovery path.

### Step 8 — Analyse feedback

Track fields such as:

```text
detected_pattern
```

and use acceptance/dismissal data to improve prompts and detection logic.

---

## Exam Memory Aid

Remember this sequence:

> **Extract → Validate → Diagnose → Retry with feedback → Revalidate → Escalate if necessary**

And the most important formula:

> **Retry = Original document + Failed extraction + Specific validation error**

### Quick Decision Rules

| Scenario | Best action |
|---|---|
| Wrong format | Retry with specific error |
| Wrong field placement | Retry with specific error |
| Mathematical discrepancy | Retry with specific error |
| Missing information that exists in source | Retry |
| Information genuinely absent from source | Do not retry indefinitely |
| Invalid schema structure | Fix/retry |
| Valid JSON but wrong business value | Semantic validation + retry |
| Repeated validation failures | Escalate |
| Repeatedly dismissed finding pattern | Refine prompt/rules |

### Final Takeaway

**Validation makes failures visible.**

**Specific feedback makes retries useful.**

**Retry boundaries prevent wasted effort.**

**Feedback data improves the system over time.**
