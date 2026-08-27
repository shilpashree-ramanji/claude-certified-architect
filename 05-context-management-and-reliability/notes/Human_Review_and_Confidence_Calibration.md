# Human Review & Confidence Calibration

## Reliability & Human-in-the-Loop Systems

### What You Need to Know

**Human review** is the mechanism for involving a person when an AI system cannot safely or reliably make a decision on its own.

**Confidence calibration** is the process of determining whether the model's confidence signals actually correspond to real-world correctness.

The core principle is:

> **Do not treat model confidence as proof of correctness. Validate confidence against observed outcomes and route uncertain or high-risk cases appropriately.**

A strong workflow combines:

```text
Model output
    ↓
Confidence / risk assessment
    ↓
Validation
    ↓
Automatic processing OR human review
    ↓
Reviewer outcome
    ↓
Calibration feedback
```

---

# Why Human Review Matters

AI systems are useful for high-volume processing, but some cases require human judgment.

Human review is especially useful when:

- Evidence is incomplete
- Sources conflict
- The model is uncertain
- The task is high risk
- A business rule requires approval
- The model's confidence is poorly calibrated
- A category has unacceptable false positives or false negatives

The goal is not to send everything to a human.

The goal is:

> **Use human review where human judgment provides the most value.**

---

# Confidence Is Not the Same as Accuracy

A model may report:

```text
Confidence: 95%
```

That does not automatically mean:

```text
95% probability that the answer is correct.
```

Confidence must be evaluated against actual outcomes.

For example:

```text
100 predictions
Model confidence ≈ 90%
Correct predictions = 70
```

The confidence is not well calibrated.

A calibrated system would have confidence that better corresponds to observed correctness.

---

# What Is Confidence Calibration?

Confidence calibration asks:

> **When the model says it is 80% confident, is it actually correct about 80% of the time for comparable cases?**

Imagine grouping predictions:

| Confidence | Actual accuracy |
|---:|---:|
| 50% | 48% |
| 60% | 62% |
| 70% | 69% |
| 80% | 81% |
| 90% | 91% |

This is reasonably calibrated.

But if:

| Confidence | Actual accuracy |
|---:|---:|
| 90% | 65% |

then the model is **overconfident**.

---

# Human Review Thresholds

A system can use confidence or risk thresholds to route work.

Example:

```text
High confidence + low risk
        ↓
Automatic processing

Medium confidence
        ↓
Additional validation

Low confidence or high risk
        ↓
Human review
```

The exact thresholds depend on the application.

There is no universal rule such as:

```text
confidence < 80% = human review
```

The threshold should be determined using actual validation data and business risk.

---

# Confidence Thresholds Must Be Validated

Suppose a team chooses:

```text
Confidence ≥ 90% → automate
Confidence < 90% → human review
```

That threshold should not be accepted simply because 90% sounds reasonable.

Test it against real outcomes.

For example:

```text
90–100% confidence
→ 97% accuracy

80–89% confidence
→ 88% accuracy

70–79% confidence
→ 72% accuracy
```

The team can then determine whether the 90% automation threshold provides acceptable risk.

---

# Risk Matters Alongside Confidence

Confidence alone should not determine whether an action is automated.

Consider:

```text
Case A:
Confidence = 85%
Risk = Low

Case B:
Confidence = 85%
Risk = High
```

The same confidence level may lead to different decisions.

A low-risk classification might be automated.

A high-risk decision may require human review even with high model confidence.

### Principle

> **Automation thresholds should consider both confidence and consequence.**

---

# High-Risk Example

Suppose a financial workflow has this rule:

```text
Refunds ≤ $500 → automatic
Refunds > $500 → manager approval
```

The model is:

```text
99% confident
```

that a refund request is for:

```text
$1,000
```

It still must not automatically approve the refund.

Why?

Because the business rule requires approval.

```text
High confidence
      +
Policy threshold exceeded
      ↓
Human approval
```

Confidence cannot override a mandatory control.

---

# Document-Type and Field-Segment Validation

This is one of the most important concepts in confidence calibration.

Never make an automation decision based only on **aggregate metrics**.

Instead, validate performance by:

- Document type
- Field
- Field segment
- Relevant category
- Risk level

---

# The Aggregate Metrics Trap

Suppose an extraction system has:

```text
Overall accuracy = 95%
```

That sounds excellent.

But break it down:

| Document type | Accuracy |
|---|---:|
| Invoices | 99% |
| Purchase orders | 97% |
| Legal contracts | 78% |

The aggregate number hides a major weakness.

If the team automates everything based on the 95% overall accuracy, legal contracts may receive inadequate human oversight.

---

# Field-Level Example

Suppose invoice extraction has:

```text
Overall accuracy = 96%
```

Break it down by field:

| Field | Accuracy |
|---|---:|
| Invoice number | 99% |
| Vendor name | 98% |
| Invoice date | 97% |
| Tax amount | 91% |
| Payment terms | 74% |

The average hides the weak `payment_terms` field.

A better automation policy could be:

```text
Invoice number → automate
Vendor name → automate
Invoice date → automate
Tax amount → validate / monitor
Payment terms → human review
```

### Key Principle

> **Validate accuracy by document type and field segment before automating.**

---

# Stratified Random Sampling

**Stratified random sampling** means dividing the population into meaningful subgroups and randomly sampling from each subgroup.

For document processing, strata might include:

```text
Document type
├── Invoices
├── Contracts
├── Purchase orders
└── Receipts
```

Within each group, randomly select documents for evaluation.

You can also stratify by field or other relevant segments.

---

## Why Stratified Sampling Matters

A simple random sample can accidentally overrepresent common categories.

Suppose your dataset contains:

```text
90% invoices
10% contracts
```

A random sample may contain very few contracts.

You could conclude:

```text
System accuracy = excellent
```

while failing to adequately test the smaller but important contract category.

Stratification ensures important subgroups are deliberately represented.

---

# Example of Stratified Evaluation

Suppose you want to review 100 documents.

Instead of:

```text
Randomly choose 100 from the entire population
```

you might create strata:

```text
40 invoices
25 purchase orders
20 receipts
15 contracts
```

Then randomly select documents within each stratum according to the evaluation plan.

The exact allocation depends on the purpose of the study.

The important idea is:

> **Make sure important subgroups are represented rather than allowing the largest subgroup to dominate the evaluation.**

---

# Reviewer Capacity Prioritization

Human reviewers are a limited resource.

If 10,000 cases require processing, you may not have enough reviewer capacity to manually inspect every case.

Therefore, prioritize cases where human review has the highest value.

Possible prioritization signals include:

- Low confidence
- High business risk
- High-value transactions
- Conflicting evidence
- Unusual document types
- Historically poor-performing fields
- Large potential impact

---

# Example: Reviewer Capacity

Suppose a system produces:

```text
10,000 documents
```

Only:

```text
500 reviewer-hours
```

are available.

A naive strategy might randomly send 5% of everything to reviewers.

A better strategy could prioritize:

```text
High-risk + low-confidence
        ↓
Human review first

Low-risk + high-confidence
        ↓
Automatic processing
```

This concentrates limited human capacity where it can reduce the most risk.

---

# Reviewer Prioritization Is Not the Same as Random Sampling

These serve different purposes.

### Stratified random sampling

Used to **evaluate system performance** across important subgroups.

### Reviewer prioritization

Used to **allocate limited human-review capacity** to cases that need attention most.

You can use both:

```text
Stratified sample
     ↓
Measure/calibrate system
     ↓
Set review thresholds

Operational cases
     ↓
Risk + confidence prioritization
     ↓
Reviewer queue
```

---

# Human Review Feedback Loop

Human reviewers create valuable feedback data.

Example:

```text
AI prediction
     ↓
Human review
     ↓
Correct / incorrect
     ↓
Store outcome
     ↓
Measure calibration
     ↓
Adjust thresholds / prompts
     ↓
Evaluate again
```

This is a feedback loop.

The reviewer outcome provides real evidence about whether the model's confidence and decisions are reliable.

---

# Example: Calibration Feedback

Suppose the system automatically processes cases with:

```text
confidence ≥ 90%
```

Human audits reveal:

```text
90–94% confidence → 82% accurate
95–100% confidence → 98% accurate
```

The team may determine that:

```text
90–94% → human review
95–100% → automation
```

The important point is that the threshold was informed by observed performance rather than chosen arbitrarily.

---

# False Positives and Reviewer Trust

A high false-positive rate can damage trust in the entire system.

Suppose a code-review system reports:

```text
Security findings → 98% accurate
Documentation mismatch → 60% accurate
```

The documentation category generates many incorrect findings.

Developers may start ignoring the entire review output, including the accurate security findings.

This creates a **trust problem**, not merely a category-level accuracy problem.

---

# The Correct Response to a Noisy Category

If one category produces an unacceptable false-positive rate:

```text
Identify noisy category
        ↓
Temporarily disable category
        ↓
Refine prompt / criteria
        ↓
Test against representative data
        ↓
Measure false-positive rate
        ↓
Re-enable when acceptable
```

This protects trust in the rest of the system while the problematic category is improved.

---

# Why “Just Increase the Confidence Threshold” Is Not Always Enough

Suppose a documentation detector has many false positives.

A tempting solution is:

```text
Only report confidence > 95%
```

But if the model is poorly calibrated, it may still produce:

```text
False positive
Confidence = 98%
```

Therefore, simply raising the confidence threshold does not necessarily solve the underlying problem.

The category's:

- Criteria
- Prompt
- Examples
- Validation
- Evaluation data

may need improvement.

---

# Human Review for Ambiguity

Human review is particularly valuable when the evidence is ambiguous.

Example:

```text
Contract:
Section 3 → Net 30
Section 8 → Net 60
```

The model should not simply choose one.

A structured result can say:

```json
{
  "conflict_detected": true,
  "values": ["Net 30", "Net 60"],
  "requires_review": true
}
```

The human reviewer can resolve the ambiguity.

---

# Human Review and Error Propagation

Human review should receive enough context to make a useful decision.

A good review package can include:

- Original source
- AI output
- Confidence
- Validation errors
- Relevant evidence
- Reason for escalation

Example:

```text
Document:
Invoice #123

AI extraction:
Total = $1,250

Validation:
Line items sum to $1,150

Confidence:
0.84

Reason for review:
Total discrepancy could not be resolved automatically.
```

The reviewer can investigate the exact issue rather than starting from scratch.

---

# Confidence Calibration by Segment

A model may be well calibrated overall but poorly calibrated for a particular segment.

Example:

```text
Overall:
90% confidence → 89% accuracy
```

But:

```text
Invoices:
90% confidence → 92% accuracy

Contracts:
90% confidence → 65% accuracy
```

A single global threshold would be inappropriate.

Use segment-specific evaluation when the performance characteristics differ materially.

---

# Calibration Metrics

Useful measurements include:

### Accuracy

```text
Correct predictions / Total predictions
```

### Precision

```text
True positives / All predicted positives
```

### Recall

```text
True positives / All actual positives
```

### False-positive rate

How often the system flags something incorrectly.

### Calibration

How closely confidence corresponds to actual correctness.

These metrics answer different questions.

Do not treat one aggregate number as a complete picture of system quality.

---

# Calibration Curves

A calibration curve compares:

```text
Predicted confidence
        vs
Observed accuracy
```

Conceptually:

```text
Observed
accuracy
  1.0 |                  *
      |              *
      |          *
      |      *
      |   *
  0.0 +-------------------------
       0.0              1.0
          Predicted confidence
```

A well-calibrated model roughly follows the diagonal.

If observed accuracy is consistently below predicted confidence, the model is overconfident.

---

# Human Review Queue Design

A useful reviewer queue can rank cases by:

```text
Priority =
Risk × Uncertainty × Impact
```

This is a conceptual framework rather than a universal formula.

For example:

```text
Case A:
High risk + low confidence
→ Very high priority

Case B:
Low risk + high confidence
→ Low priority
```

The actual scoring system should be based on the organization's operational requirements.

---

# Exam Traps

## Trap 1 — “Overall accuracy is 95%, so automate everything.”

**Wrong.**

Break performance down by document type, field, and relevant segments.

---

## Trap 2 — “90% confidence means 90% accuracy.”

**Wrong unless the confidence is actually calibrated.**

Confidence must be compared against observed outcomes.

---

## Trap 3 — “Raise the confidence threshold to eliminate false positives.”

**Not necessarily.**

A poorly calibrated model can be confidently wrong.

Fix the underlying criteria and evaluation process.

---

## Trap 4 — “Random sampling is always enough.”

**Wrong for subgroup coverage.**

If important categories are small, use stratified random sampling to ensure they are represented.

---

## Trap 5 — “Send every uncertain case to a human.”

**Not necessarily.**

Human capacity is limited. Prioritize cases based on uncertainty, risk, impact, and known weak segments.

---

## Trap 6 — “High model confidence overrides business policy.”

**Wrong.**

Mandatory approval rules and safety controls take precedence over model confidence.

---

## Trap 7 — “One bad category should remain enabled because other categories are accurate.”

**Wrong when false positives are damaging system-wide trust.**

Temporarily disable and refine the noisy category.

---

## Trap 8 — “Human review means the AI failed.”

**Wrong.**

Human review is an intentional part of a reliable system for cases where automated decisions are uncertain, ambiguous, or high risk.

---

# Practice Scenario 1

An invoice extraction system reports:

```text
Overall accuracy = 95%
```

But evaluation shows:

```text
Invoice number = 99%
Vendor name = 98%
Payment terms = 72%
```

What should the team do before automating all invoice fields?

**A.** Use the 95% aggregate accuracy and automate everything.

**B.** Increase the model's temperature.

**C.** Evaluate and calibrate automation by field segment, keeping payment terms under appropriate review until performance is acceptable.

**D.** Remove payment terms from the evaluation.

### Correct Answer

**C**

Aggregate accuracy hides the weak payment-terms field.

---

# Practice Scenario 2

A model reports 95% confidence on a classification task. Historical evaluation shows that cases in this confidence range are only 70% accurate.

What does this indicate?

**A.** Excellent calibration

**B.** Overconfidence / poor calibration

**C.** Perfect automation readiness

**D.** A successful batch job

### Correct Answer

**B**

The confidence substantially exceeds observed accuracy.

---

# Practice Scenario 3

A dataset contains:

```text
90% invoices
10% contracts
```

A team wants to evaluate accuracy across both document types.

What is a strong approach?

**A.** Randomly sample without tracking document type.

**B.** Evaluate only invoices because they dominate the dataset.

**C.** Use stratified random sampling so both document types are adequately represented.

**D.** Evaluate only contracts.

### Correct Answer

**C**

Stratification prevents the large invoice category from hiding contract performance.

---

# Practice Scenario 4

A code-review category produces a 40% false-positive rate and developers have started ignoring all review findings.

What is the best response?

**A.** Keep the category enabled and tell developers to be more careful.

**B.** Temporarily disable the noisy category while refining its criteria and validating the improved version.

**C.** Increase the temperature.

**D.** Ignore the false positives because other categories are accurate.

### Correct Answer

**B**

A noisy category can damage trust in the entire system.

---

# Practice Scenario 5

A human-review team can inspect only 500 cases per day. The system produces 10,000 cases.

Which cases should receive the highest priority?

**A.** Random cases only

**B.** High-risk and uncertain cases, especially from historically weak segments

**C.** Only the easiest cases

**D.** Only cases with the highest model confidence

### Correct Answer

**B**

Limited reviewer capacity should be directed toward cases where human intervention provides the most value.

---

# Practice Scenario 6

A model is 99% confident that a $1,000 refund should be approved, but company policy requires manager approval for refunds above $500.

What should happen?

**A.** Automatically approve because confidence is 99%.

**B.** Retry until confidence reaches 100%.

**C.** Escalate for manager approval.

**D.** Ignore the policy.

### Correct Answer

**C**

A mandatory business control overrides model confidence.

---

# Build Pattern

A practical human-review and confidence-calibration workflow:

### Step 1 — Collect predictions and outcomes

Store:

```text
Prediction
Confidence
Document type
Field / segment
Actual outcome
Reviewer decision
Risk level
```

### Step 2 — Measure performance

Calculate:

- Accuracy
- Precision
- Recall
- False-positive rate
- Calibration by confidence range

### Step 3 — Segment the evaluation

Break results down by:

```text
Document type
Field
Field segment
Risk category
Other meaningful subgroups
```

### Step 4 — Identify weak segments

Find categories where:

```text
Accuracy is low
OR
False positives are high
OR
Confidence is poorly calibrated
```

### Step 5 — Establish routing thresholds

Use observed performance to determine:

```text
Automatic
Additional validation
Human review
```

### Step 6 — Prioritize reviewer capacity

Send the highest-risk and most uncertain cases to humans first.

### Step 7 — Capture reviewer outcomes

Use human decisions as ground-truth or evaluation signals where appropriate.

### Step 8 — Improve the system

Use feedback to refine:

- Prompts
- Examples
- Criteria
- Schemas
- Validation rules
- Thresholds

### Step 9 — Re-evaluate

Run the same evaluation process again to verify improvement.

---

# Exam Memory Aid

Remember:

> **Confidence is a signal, not proof.**
>
> **Validate accuracy by document type and field segment.**
>
> **Never automate based on aggregate metrics alone.**
>
> **Use stratified sampling for subgroup coverage.**
>
> **Prioritize scarce reviewer capacity by risk and uncertainty.**
>
> **Mandatory business rules override model confidence.**
>
> **High false-positive categories can damage system-wide trust.**

---

## Quick Decision Table

| Situation | Recommended action |
|---|---|
| High confidence + validated low-risk performance | Consider automation |
| Low confidence | Human review or additional validation |
| High-risk action | Apply required controls / human approval |
| High aggregate accuracy but weak segment | Review segment separately |
| Small important document category | Stratified sampling |
| Poor confidence calibration | Recalibrate thresholds / improve system |
| High false-positive category | Temporarily disable and refine |
| Limited reviewer capacity | Prioritize high-risk / uncertain cases |
| Conflicting evidence | Human review / ambiguity resolution |
| Model confidence conflicts with policy | Follow policy |

---

# Final Takeaway

Human review and confidence calibration are about **knowing when automation can be trusted**.

The reliable workflow is:

```text
Model prediction
      ↓
Confidence + risk
      ↓
Segment-level validation
      ↓
┌──────────────┴──────────────┐
Reliable                    Uncertain /
low-risk                    high-risk
   ↓                            ↓
Automation                 Human review
   ↓                            ↓
              Reviewer outcome
                    ↓
              Calibration data
                    ↓
              Improve thresholds,
              prompts & rules
                    ↓
                 Re-test
```

The biggest exam lesson is:

> **Never let a good aggregate metric create false confidence.**

A system can be 95% accurate overall while being dangerously weak on one document type, one field, or one high-risk segment.

The right question is not:

```text
"How accurate is the system overall?"
```

It is:

```text
"Where is it accurate, where is it not,
how well calibrated is its confidence,
and where should a human intervene?"
```
