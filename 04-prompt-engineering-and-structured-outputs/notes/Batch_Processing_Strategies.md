# Batch Processing Strategies

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

The **Message Batches API** is a cost-optimization option for workloads that can tolerate delayed results.

The core decision is not simply:

> “Batch is cheaper, so use batch.”

Instead, ask:

> **Does the workflow require the result immediately, or can it consume the result later?**

According to the Claude Certification Guide, the Message Batches API provides **50% cost savings compared with synchronous API calls**, allows an **up-to-24-hour processing window**, has **no guaranteed latency SLA**, does not support **multi-turn tool calling within a single batch request**, and uses **`custom_id`** to correlate requests with responses. citeturn0search0

---

## Synchronous vs Batch Processing

### Synchronous API

Use synchronous calls for **blocking workflows** where a person or system is waiting for the result.

Examples:

- Pre-merge CI/CD checks
- Real-time code review feedback
- Interactive user requests
- Any workflow where the next step cannot proceed until the result arrives

Example:

```typescript
const preMergeReview = await client.messages.create({
  model: "claude-sonnet-5",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content: prDiffContent
    }
  ]
});
```

The developer is waiting for the review, so latency matters.

---

### Batch API

Use batch processing for **latency-tolerant workflows** where results are consumed later.

Examples:

- Overnight technical-debt reports
- Weekly code-audit summaries
- Nightly test generation
- Batch document extraction
- Large-scale analysis where immediate results are unnecessary

Example:

```typescript
const batchRequest = await client.batches.create({
  requests: technicalDebtDocuments.map((doc, i) => ({
    custom_id: `debt-report-${i}`,
    params: {
      model: "claude-sonnet-5",
      max_tokens: 4096,
      messages: [
        {
          role: "user",
          content: doc
        }
      ]
    }
  }))
});
```

The important distinction is **latency requirement**, not simply cost. citeturn0search0

---

## The Most Important Matching Rule

Remember:

| Workflow characteristic | Recommended approach |
|---|---|
| Someone is blocked waiting for the result | Synchronous |
| Real-time response required | Synchronous |
| Pre-merge check | Synchronous |
| Results consumed later | Batch |
| Overnight report | Batch |
| Weekly audit | Batch |
| Nightly document extraction | Batch |

### Exam Rule

> **Blocking workflow → synchronous.**
>
> **Latency-tolerant workflow → batch.**

Do not switch a blocking workflow to batch merely because batch processing is cheaper. citeturn0search0

---

## Batch Processing Constraints

The Message Batches API has several constraints that matter directly in exam scenarios.

### 1. 50% Cost Savings

Batch processing provides approximately **50% cost savings** compared with synchronous API calls.

This makes it attractive for high-volume workloads.

But cost savings do **not** override latency requirements.

---

### 2. Up to 24-Hour Processing Window

A batch can take minutes or as long as **24 hours** to complete.

Therefore:

> Never design a blocking workflow around the assumption that a batch will finish quickly.

A batch may finish quickly in practice, but your architecture must account for the maximum processing window. citeturn0search0

---

### 3. No Guaranteed Latency SLA

The API does not provide a guaranteed completion time within that window.

Therefore, this is a bad design:

```text
Submit batch
↓
Assume it will finish in 10 minutes
↓
Block the user
```

Instead, batch processing should be used where delayed results are acceptable.

---

### 4. No Multi-Turn Tool Calling

A batch request cannot perform a multi-turn tool-calling loop inside the same request.

You cannot rely on:

```text
Model
 ↓
Call external tool
 ↓
Receive tool result
 ↓
Model reasons over result
 ↓
Call another tool
 ↓
Continue
```

inside a single batch item.

If your workflow requires this kind of **agentic multi-turn tool interaction**, use the synchronous API for that step. citeturn0search0

---

## `custom_id`: Request-Response Correlation

Each batch request should have a unique `custom_id`.

Example:

```json
{
  "custom_id": "invoice-001",
  "params": {
    "model": "claude-sonnet-5",
    "max_tokens": 4096,
    "messages": [
      {
        "role": "user",
        "content": "Extract this invoice."
      }
    ]
  }
}
```

Another request:

```json
{
  "custom_id": "invoice-002",
  "params": {
    "model": "claude-sonnet-5",
    "max_tokens": 4096,
    "messages": [
      {
        "role": "user",
        "content": "Extract this invoice."
      }
    ]
  }
}
```

When results return, `custom_id` tells you which result belongs to which original request.

### Why It Matters

Without correlation identifiers, failure handling becomes difficult.

With them:

```text
invoice-001 → success
invoice-002 → failure
invoice-003 → success
invoice-004 → failure
```

You can retry only the failed requests.

---

## Batch Failure Handling

Not every request in a batch will necessarily succeed.

The correct failure-handling pattern is:

### Step 1 — Identify failed requests

Use `custom_id` to determine which requests failed.

```typescript
const results = await client.batches.results(batchId);

const failures = results.filter(
  r => r.result.type === "errored"
);

const failedIds = failures.map(
  f => f.custom_id
);
```

### Step 2 — Retrieve the original inputs

Use the failed `custom_id` values to locate the original documents.

### Step 3 — Modify the failed requests

Do not simply send the exact same request again.

Target the likely cause.

Possible modifications include:

- Chunk oversized documents
- Increase `max_tokens`
- Simplify extraction prompts
- Add format-specific few-shot examples
- Adjust the schema
- Change instructions for unusual document structures

### Step 4 — Resubmit only failures

Do **not** resubmit successful documents.

```text
Original batch:
100 documents

Success:
92

Failures:
8

Retry:
8 documents only
```

This avoids paying to process the successful 92 again. citeturn0search0

---

## Why `custom_id` Is an Exam Favorite

A typical exam scenario asks how to handle a batch containing mixed successes and failures.

The answer is:

> **Use `custom_id` to identify failures and resubmit only those failed requests with targeted modifications.**

Do not rerun the entire batch.

---

## Prompt Optimization Before Batch Submission

One of the most cost-effective strategies is to **refine the prompt before sending the full batch**.

Do not immediately submit:

```text
1,000 documents
↓
Batch API
↓
Discover prompt problems afterward
```

Instead:

```text
Select representative sample
        ↓
Test prompt
        ↓
Find errors
        ↓
Refine prompt/schema/examples
        ↓
Retest
        ↓
Submit full batch
```

The guide recommends a representative sample of approximately **5–10 documents** covering the range of formats, document types, and edge cases. citeturn0search0

---

## Why Sample Testing Saves Money

Suppose you process 1,000 documents.

### Scenario A — 90% first-pass success

```text
1,000 documents
900 succeed
100 require retry
```

Only 100 need additional processing.

### Scenario B — 60% first-pass success

```text
1,000 documents
600 succeed
400 require retry
```

Now 400 documents require additional processing.

That is **four times as many retries** as the 90% success scenario.

### Principle

> **Spend effort refining prompts before the expensive large-scale run.**

---

## SLA Calculation

Batch processing requires working backward from the deadline.

Suppose an organization needs a report within a **30-hour SLA**.

Maximum batch processing window:

```text
24 hours
```

Remaining time:

```text
30 - 24 = 6 hours
```

So there are **6 hours of buffer** for:

- Collecting requests
- Preparing inputs
- Validation
- Operational delays
- Scheduling

The guide recommends thinking backward from the deadline and submitting batches during that buffer so a fresh batch is already in flight. citeturn0search0

### Exam Memory

```text
30-hour SLA
− 24-hour maximum processing
= 6-hour buffer
```

---

## Batch Processing with Tool Requirements

Suppose the workflow says:

```text
For each document:
1. Ask Claude to inspect the document.
2. Call an external database tool.
3. Use the database result.
4. Call another tool.
5. Produce the final answer.
```

This is a multi-turn agentic workflow.

Do **not** assume the Batch API can perform that entire loop inside one batch item.

The batch limitation means this workflow requires synchronous processing for the tool-interactive portion. citeturn0search0

---

## Batch Processing Decision Tree

Use this quick decision process:

```text
Does someone/system need the result immediately?
            │
        ┌───┴───┐
       Yes      No
        │        │
  Synchronous   Batch
```

Then ask:

```text
Does the workflow require multi-turn tool calling?
            │
           Yes
            ↓
       Synchronous
```

For batch workflows:

```text
Before full batch
       ↓
Test representative sample
       ↓
Refine prompt/schema/examples
       ↓
Submit batch
       ↓
Process results
       ↓
Identify failures by custom_id
       ↓
Retry failures only
```

---

## Exam Traps

### Trap 1 — Switching everything to batch for cost savings

**Wrong.**

Blocking workflows must remain synchronous because batch processing can take up to 24 hours and has no guaranteed latency SLA.

---

### Trap 2 — Assuming batches normally finish quickly

**Wrong.**

Even if results often arrive in minutes, architecture must account for the possible 24-hour processing window.

---

### Trap 3 — Using batch for multi-turn tool calling

**Wrong.**

The Message Batches API does not support multi-turn tool calling within a single batch request.

---

### Trap 4 — Retrying the entire batch

**Wrong.**

Identify failed requests with `custom_id` and resubmit only those failures.

---

### Trap 5 — Using the same request unchanged for every retry

**Wrong.**

Modify the failed request based on the failure.

Examples:

- Chunk oversized documents
- Increase output capacity
- Simplify prompts
- Add format-specific examples

---

### Trap 6 — Skipping sample testing

**Wrong.**

Refine prompts against a representative sample before processing the full batch.

---

### Trap 7 — Designing around best-case latency

**Wrong.**

The correct architecture uses the maximum processing window rather than assuming the batch will finish quickly.

---

## Practice Scenario 1

A team has two workflows:

1. A pre-merge security review that blocks developers from merging.
2. An overnight technical-debt report reviewed the next morning.

The manager wants to move both workflows to batch processing because it is cheaper.

What is the best approach?

**A.** Move both workflows to batch.

**B.** Keep the pre-merge review synchronous and move the overnight report to batch.

**C.** Keep both synchronous because batch processing is unreliable.

**D.** Move both to batch and assume results will arrive within minutes.

### Correct Answer

**B**

The pre-merge review is blocking, while the overnight technical-debt report is latency-tolerant. citeturn0search0

---

## Practice Scenario 2

A batch contains 500 documents. After processing:

```text
470 succeeded
30 failed
```

What should the system do?

**A.** Resubmit all 500.

**B.** Ignore the 30 failures.

**C.** Identify the 30 failures using `custom_id`, modify their requests as appropriate, and resubmit only those.

**D.** Switch the entire workflow to synchronous processing.

### Correct Answer

**C**

`custom_id` provides request-response correlation, allowing targeted retries. citeturn0search0

---

## Practice Scenario 3

A team has a batch of 1,000 invoices but has never tested its extraction prompt.

What is the best first step?

**A.** Submit all 1,000 immediately.

**B.** Test the prompt on a representative sample of roughly 5–10 invoices covering different formats and edge cases.

**C.** Use synchronous processing for all 1,000.

**D.** Add more output tokens without testing.

### Correct Answer

**B**

Sample testing lets the team refine the prompt before incurring large-scale processing and retry costs. citeturn0search0

---

## Build Pattern

A production-oriented batch workflow looks like this:

### Step 1 — Classify the workflow

Determine whether the workflow is:

```text
Blocking → synchronous
Latency-tolerant → batch
```

### Step 2 — Check tool requirements

If multi-turn tool calling is required, do not put that workflow inside a single batch request.

### Step 3 — Build a representative sample

Use approximately 5–10 documents covering:

- Different formats
- Different document types
- Edge cases
- Expected difficult cases

### Step 4 — Refine

Improve:

- Prompt wording
- Few-shot examples
- Schema
- Chunking
- Output capacity

### Step 5 — Submit the batch

Give every request a unique:

```text
custom_id
```

### Step 6 — Process results

Separate:

```text
Successes
Failures
```

### Step 7 — Retry failures only

Use `custom_id` to locate failed inputs and apply targeted modifications.

### Step 8 — Track SLA

Work backward from the deadline and account for the maximum 24-hour processing window.

---

## Exam Memory Aid

Remember these five facts:

> **50% cheaper**
>
> **Up to 24 hours**
>
> **No guaranteed latency SLA**
>
> **No multi-turn tool calling**
>
> **`custom_id` correlates requests and results**

And the central rule:

> **Blocking → synchronous.**
>
> **Latency-tolerant → batch.**

### Quick Decision Rules

| Scenario | Best choice |
|---|---|
| Developer waiting for pre-merge result | Synchronous |
| Real-time user interaction | Synchronous |
| Multi-turn tool-calling workflow | Synchronous |
| Overnight report | Batch |
| Weekly audit | Batch |
| Nightly document extraction | Batch |
| Mixed batch results | Use `custom_id` |
| Failed documents | Retry failures only |
| Large new workload | Test 5–10 representative documents first |
| 30-hour SLA | Account for 24-hour maximum + 6-hour buffer |

### Final Takeaway

**Batch processing is a cost optimization, not a universal replacement for synchronous calls.**

Use it when the workflow can tolerate delayed results.

Before a large batch, **refine the prompt on a representative sample**.

After processing, **use `custom_id` to identify failures and retry only those failures with targeted modifications**.

And always design around the **24-hour processing window**, not the best-case completion time.
