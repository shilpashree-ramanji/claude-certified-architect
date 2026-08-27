# Few-Shot Prompting

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

**Few-shot prompting** means giving the model a small number of representative examples so it can infer the desired behaviour, output pattern, or classification style.

The basic pattern is:

```text
Instruction
+ Example 1
+ Example 2
+ Example 3
→ New input for the model to handle
```

Instead of explaining every possible case in abstract language, examples demonstrate what the expected behaviour looks like.

---

## When Few-Shot Prompting Is Useful

Few-shot examples are especially useful when the desired behaviour is difficult to express completely through prose.

Good use cases include:

- Classification
- Structured extraction
- Output formatting
- Ambiguous categories
- Demonstrating subtle distinctions
- Showing correct tool-selection patterns

### Example: Classification

**Prompt:**

```text
Classify each issue as Critical, Major, or Minor.

Example 1:
SQL injection vulnerability → Critical

Example 2:
Incorrect error message → Major

Example 3:
Inconsistent variable naming → Minor

Now classify:
Hardcoded API credential in source code.
```

The examples establish the expected classification pattern.

---

## Few-Shot Prompting vs Explicit Criteria

These techniques solve related but different problems.

### Explicit criteria

Use explicit criteria to define **what the model should consider correct**.

```text
Report security vulnerabilities.
Skip formatting preferences.
Flag only documentation claims that contradict actual code behaviour.
```

### Few-shot examples

Use examples to demonstrate **how those criteria should be applied**.

```text
Example:
Claim: "This function validates the user ID."
Code: No validation occurs.
Result: Documentation mismatch.
```

### Key Principle

> **Criteria define the rules; examples demonstrate the rules in action.**

In many production prompts, the strongest design combines both.

---

## Few-Shot Prompting for Tool Selection

Few-shot examples can demonstrate which tool should be selected for particular requests.

Example:

```text
User: "Check the status of order #12345."
Tool: lookup_order

User: "Find the customer associated with john@example.com."
Tool: get_customer

User: "What is the current status of order #67890?"
Tool:
```

Expected tool:

```text
lookup_order
```

This can help when the model needs to learn a recurring distinction between similar tools.

### Important Exam Distinction

If the root problem is that tool descriptions are **too vague**, improve the descriptions first.

Few-shot examples are not automatically the best first fix.

---

## Few-Shot Prompting for Structured Output

Examples can show the exact structure expected from the model.

### Example

```text
Return customer information using this structure:

{
  "customer_id": "...",
  "status": "...",
  "risk_score": 0
}

Example:

Input:
Customer 123 is active with low risk.

Output:
{
  "customer_id": "123",
  "status": "active",
  "risk_score": 10
}
```

The example demonstrates the desired output shape and interpretation.

---

## Choosing Good Examples

Good few-shot examples should be:

### 1. Representative

Examples should reflect the situations the model will actually encounter.

### 2. Clear

Avoid examples where the expected answer is ambiguous.

### 3. Diverse

Include important variations rather than repeating nearly identical cases.

### 4. Boundary-focused

Examples near category boundaries are particularly useful.

For example, if distinguishing between **Major** and **Minor**, include examples showing what separates those two categories.

### 5. Correct

Incorrect demonstrations can teach the model the wrong behaviour.

---

## Example Selection Matters

Not all examples have equal value.

Suppose an agent must classify support tickets:

```text
Billing
Technical
Account Access
Security
```

Providing five examples that are all billing tickets gives little information about the other categories.

A stronger set might include:

```text
"Charged twice" → Billing
"Application crashes on startup" → Technical
"Cannot reset password" → Account Access
"Someone accessed my account" → Security
```

Each example demonstrates a different decision boundary.

---

## The Token-Cost Tradeoff

Few-shot examples consume context.

Adding more examples can improve clarity, but excessive examples can:

- Increase prompt size
- Increase cost
- Reduce available context for the actual task
- Introduce contradictory patterns
- Make prompts harder to maintain

Therefore:

> **Use the smallest set of examples that clearly teaches the desired behaviour.**

Do not automatically add dozens of examples.

---

## Few-Shot Prompting and Edge Cases

Few-shot examples are particularly valuable for edge cases.

For example:

```text
Example:
Input: "Customer has no orders."
Result: Valid empty result, not an error.
```

This teaches the model an important distinction between:

- A successful query with no matches
- An actual access or execution failure

The example complements the explicit rule.

---

## Common Exam Traps

### Exam Trap 1 — Using few-shot examples when the root problem is vague tool descriptions

If two tools are poorly described and the model misroutes requests, the first fix is usually to **improve the tool descriptions**.

Few-shot examples may help, but they add token overhead without directly fixing the unclear tool interface.

---

### Exam Trap 2 — Assuming more examples are always better

More examples are not automatically better.

The goal is **representative coverage with minimal context cost**.

---

### Exam Trap 3 — Using examples instead of explicit rules for deterministic requirements

If a requirement must be enforced 100% of the time—such as:

- Financial approval
- Security controls
- Regulatory checks
- Refund limits

few-shot prompting does not provide a deterministic guarantee.

Use programmatic enforcement such as hooks where required.

---

### Exam Trap 4 — Providing only easy examples

If all examples are obvious, the model may still fail on boundary cases.

Include examples that demonstrate important distinctions and edge cases.

---

## Few-Shot vs Other Prompt Techniques

| Technique | Primary purpose |
|---|---|
| Explicit criteria | Define rules and decision boundaries |
| Few-shot prompting | Demonstrate desired behaviour through examples |
| Structured output | Constrain the response into a defined schema |
| Prompt caching | Efficiently reuse unchanged prompt content |
| Summarization | Compress accumulated context |

These techniques can work together.

---

## Practical Design Pattern

A strong production prompt can combine:

```text
1. Explicit task definition
2. Explicit inclusion/exclusion criteria
3. A small number of representative examples
4. Required output schema
5. Edge-case examples
6. Clear final instruction
```

For example:

```text
Task:
Review code for security vulnerabilities.

Criteria:
- Report exploitable security vulnerabilities.
- Skip formatting and style preferences.
- Do not report theoretical issues without evidence.

Examples:

Example 1:
Unsanitised SQL input → Critical

Example 2:
Unused variable → Skip

Example 3:
Hardcoded credential → Critical

Return:
{
  "severity": "...",
  "finding": "...",
  "evidence": "..."
}
```

This gives the model both **rules** and **demonstrations**.

---

## Exam Memory Aid

Remember:

> **Explicit criteria tell the model WHAT the rule is.**
>
> **Few-shot examples show the model HOW to apply it.**

And:

> **Use enough examples to teach the pattern, but not so many that they consume the context window unnecessarily.**

### Quick Decision Rule

If the question says:

- **“The model doesn't understand the expected pattern.”** → Consider few-shot examples.
- **“The rules are ambiguous.”** → Add explicit criteria.
- **“Two tools are being confused because descriptions are minimal.”** → Improve tool descriptions first.
- **“100% compliance is required.”** → Use deterministic enforcement, not prompting alone.
- **“Prompt is becoming too large.”** → Reduce or optimize examples.
