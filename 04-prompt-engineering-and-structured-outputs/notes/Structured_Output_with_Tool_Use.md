# Structured Output with Tool Use

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

Structured output means requiring an agent to return information in a predictable, machine-readable format rather than relying on free-form prose.

When structured output is combined with tool use, the agent can produce results that are:

- Consistent
- Easier to validate
- Easier for downstream agents to consume
- Easier to route into workflows
- Less dependent on interpreting natural-language responses

The core idea is:

> **Use tools for actions and structured schemas for predictable results.**

---

## Why Structured Output Matters

Free-form output is difficult for software to consume reliably.

### Unstructured response

```text
The customer appears to be active. Their risk is relatively low,
and I found two recent orders.
```

A downstream system has to interpret the text to determine:

- Customer status
- Risk level
- Number of orders

### Structured response

```json
{
  "customer_id": "C123",
  "status": "active",
  "risk_level": "low",
  "order_count": 2
}
```

Now the downstream system knows exactly where each value belongs.

---

## Structured Output with Tool Calls

A tool can require a defined input schema.

For example:

```json
{
  "name": "create_customer_review",
  "description": "Create a structured customer review.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {
        "type": "string"
      },
      "risk_level": {
        "type": "string",
        "enum": ["low", "medium", "high"]
      },
      "requires_review": {
        "type": "boolean"
      }
    },
    "required": [
      "customer_id",
      "risk_level",
      "requires_review"
    ]
  }
}
```

The schema constrains the tool call so the receiving system gets predictable fields.

---

## Tool Use vs Normal Text

A useful distinction:

### Normal text

Use when the model is responding conversationally.

```text
The customer has two active orders.
```

### Tool use

Use when the model needs to perform an operation or produce data that another system will consume.

```json
{
  "name": "lookup_order",
  "arguments": {
    "order_id": "12345"
  }
}
```

The model is not merely describing the action. It is producing a structured tool invocation.

---

## Required vs Optional Fields

A good schema explicitly identifies which information is mandatory.

Example:

```json
{
  "type": "object",
  "properties": {
    "customer_id": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "notes": {
      "type": "string"
    }
  },
  "required": [
    "customer_id",
    "status"
  ]
}
```

Here:

- `customer_id` is required.
- `status` is required.
- `notes` is optional.

This prevents downstream systems from having to guess which fields should always be present.

---

## Enums for Controlled Values

When a field has a known set of valid values, use an enum.

### Weak

```json
{
  "status": "string"
}
```

The model could return:

```text
active
Active
ACTIVE
currently active
```

### Better

```json
{
  "status": {
    "type": "string",
    "enum": [
      "active",
      "cancelled",
      "pending"
    ]
  }
}
```

Now the allowed values are explicit.

---

## Structured Output for Multi-Agent Systems

Structured output is particularly useful when one agent passes information to another.

Imagine:

```text
Research Agent
      ↓
Structured Result
      ↓
Synthesis Agent
```

Instead of sending the entire research transcript, the research agent can return:

```json
{
  "key_findings": [
    "Finding 1",
    "Finding 2"
  ],
  "citations": [
    "Source A",
    "Source B"
  ],
  "evidence_score": 0.91,
  "coverage": "partial"
}
```

The synthesis agent receives the information it actually needs.

This reduces context growth and makes the interface between agents predictable.

---

## Tool Results Should Also Be Structured

Tool results should clearly communicate whether an operation succeeded or failed.

### Successful result

```json
{
  "isError": false,
  "resultCount": 0,
  "message": "No customer matched the requested email."
}
```

This is a **valid empty result**, not a failure.

### Failed result

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "Customer database timed out."
}
```

The structured metadata tells the agent how to recover.

---

## Structured Errors vs Successful Empty Results

This distinction is critical.

### Valid empty result

The tool successfully executed the query but found nothing.

```json
{
  "isError": false,
  "resultCount": 0
}
```

**Action:** Accept the result.

### Access or execution failure

The tool could not successfully complete the query.

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true
}
```

**Action:** Retry or recover according to the error category.

Never represent an access failure as an empty successful result.

---

## Tool Use for Deterministic Interfaces

Tool schemas are useful when downstream software requires predictable data.

For example, instead of asking:

```text
Tell me whether this refund should be approved.
```

Use a structured operation:

```json
{
  "name": "evaluate_refund",
  "arguments": {
    "refund_amount": 750,
    "automatic_limit": 500
  }
}
```

The result can then be represented consistently:

```json
{
  "decision": "escalate",
  "reason": "refund_exceeds_limit",
  "requires_human_approval": true
}
```

The software does not need to parse a paragraph to determine what happened.

---

## Schema Design Principles

### 1. Keep schemas focused

Only include fields that downstream consumers actually need.

### 2. Use clear field names

Prefer:

```text
customer_id
risk_score
requires_review
```

over vague names such as:

```text
data
result
information
```

### 3. Use the correct data types

For example:

```json
{
  "risk_score": {
    "type": "number"
  },
  "requires_review": {
    "type": "boolean"
  }
}
```

### 4. Constrain values when possible

Use enums, ranges, and required fields where appropriate.

### 5. Avoid unnecessary nesting

Do not create a deeply nested schema when a simple structure is sufficient.

---

## Structured Output Does Not Mean “Put Everything in JSON”

A common mistake is assuming that every response should be converted into a huge JSON object.

That creates unnecessary complexity.

The goal is:

> **Structure the information that needs to be consumed reliably.**

If a user simply asks a conversational question, normal text may be appropriate.

If another agent or application needs predictable fields, structured output is much more valuable.

---

## Structured Output + Tool Selection

Tool descriptions and schemas work together.

### Description

Explains:

- What the tool does
- When to use it
- When not to use it
- Expected inputs
- Important limitations

### Schema

Defines:

- Required fields
- Data types
- Valid values
- Input structure

Think of it this way:

> **Description helps the model choose the tool. Schema helps the model call it correctly.**

---

## Exam Traps

### Exam Trap 1 — Using free-form text when downstream software requires predictable fields

If another system must consume the result reliably, structured output is preferred.

### Exam Trap 2 — Making every field optional

If downstream logic depends on a field, make that field required.

### Exam Trap 3 — Using unrestricted strings for categorical values

If only a known set of values is valid, use an enum.

### Exam Trap 4 — Returning an empty result as an error

A successful query returning zero matches is a valid empty result.

### Exam Trap 5 — Returning an access failure as an empty result

This silently hides the failure and can cause the coordinator to believe the data source was successfully checked.

### Exam Trap 6 — Creating enormous schemas

More fields do not automatically mean better structured output. Include the information required by the consumer.

### Exam Trap 7 — Confusing tool descriptions with schemas

Descriptions help **selection**. Schemas constrain **inputs and outputs**.

---

## Practice Scenario

A research subagent searches five sources. Two searches succeed completely, one returns no matching information, and two fail because of timeouts.

The synthesis agent needs to know:

- Which sources succeeded
- Which sources had no results
- Which sources failed
- What partial findings were collected

What is the best design?

**A.** Return one large natural-language paragraph describing the entire research process.

**B.** Return an empty result whenever a source fails so the synthesis agent can continue.

**C.** Return a structured result distinguishing successful results, valid empty results, failures, partial findings, and relevant metadata.

**D.** Hide failed sources because only successful research should reach the synthesis agent.

### Correct Answer

**C**

The synthesis agent needs to distinguish **success, valid empty results, and actual failures**. Structured output makes those states explicit and allows the coordinator to decide whether to retry, continue, or escalate.

---

## Exam Memory Aid

Remember:

> **Description → choose the tool.**
>
> **Schema → call the tool correctly.**
>
> **Structured result → communicate reliably downstream.**
>
> **Structured error → recover intelligently.**

And the most important distinction:

> **No data is not the same as no access.**

A successful empty result should remain a success. An actual failure should carry structured error information.

### Quick Decision Rule

If the question says:

- **“Downstream system needs predictable fields”** → Structured output
- **“Known set of valid values”** → Enum
- **“Field must always be present”** → Required field
- **“Agent must perform an operation”** → Tool use
- **“Tool failed but may recover”** → Structured error metadata
- **“Query succeeded but found nothing”** → Valid empty result, not an error
- **“Another agent needs only selected findings”** → Structured, distilled result
