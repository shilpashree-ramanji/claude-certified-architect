# CCAF Domain 2 — Tool Interface Design

## Core Idea

Tool descriptions are the **primary mechanism LLMs use for tool selection**.

A good description helps the model understand:

1. What the tool does
2. What inputs it expects
3. Example queries it handles
4. Edge cases and limitations
5. When to use it versus similar tools

### Memory Aid

> **Purpose + Inputs + Examples + Limits + Boundaries**

---

## 1. Minimal vs Production-Grade Descriptions

### Minimal

```text
get_customer:
"Retrieves customer information"

lookup_order:
"Retrieves order details"
```

These descriptions can cause misrouting because they do not clearly distinguish the tools.

### Production-grade

```text
get_customer:
Looks up a customer account by email, phone, or customer ID.
Returns profile, contact details, account status, and loyalty tier.
Use for customer identity verification.
Do NOT use for order-specific queries.

lookup_order:
Retrieves order details by order number or tracking ID.
Returns status, items, shipping details, and refund eligibility.
Use for questions about a specific order.
Do NOT use for customer identity verification.
```

The production-grade descriptions provide clear inputs, outputs, examples, and boundaries.

---

## 2. Tool Misrouting

Example:

> "Check my order #12345."

If the tools have vague descriptions, the model may select `get_customer` instead of `lookup_order`.

### First fix

**Improve the tool descriptions.**

Add:

- Input formats
- Example queries
- Returned information
- Edge cases
- "Use this when..." guidance
- "Do NOT use this when..." boundaries

### Exam Principle

> **When a manageable set of tools is confused because descriptions are weak, improve the descriptions first.**

---

## 3. Tool Splitting

Broad tools can create ambiguity.

### Before

```text
analyze_document:
"Analyses a document and returns results"
```

### After

```text
extract_data_points
→ Extracts dates, amounts, names, and other structured fields.

summarize_content
→ Produces a concise summary of key arguments and conclusions.

verify_claim_against_source
→ Checks whether a specific claim is supported by the source.
```

Each tool now has a narrow, clearly defined responsibility.

### Memory Aid

> **One tool → one clear job.**

---

## 4. Tool Renaming

Confusing names can also cause ambiguity.

Example:

```text
Before:
analyze_content
```

Rename it:

```text
extract_web_results
```

and provide a web-specific description.

The implementation does not necessarily need to change; the interface becomes clearer.

---

## 5. System Prompt Interactions

System prompt wording can create unintended tool associations.

Example:

```text
"Always check customer details before proceeding."
```

This may cause the model to associate many customer-related requests with `get_customer`, even when the user actually wants `lookup_order`.

### Important

After improving descriptions, review the system prompt for conflicting or overly broad instructions.

> **Tool descriptions and system instructions can interact.**

---

## 6. Tool Overload

Better descriptions are not a universal solution.

If an agent has too many tools, selection can degrade because of decision complexity.

The key diagnostic question is:

> **Is the model confused because two tools are poorly differentiated, or because there are too many tools overall?**

### If descriptions are the problem

```text
Weak descriptions
      ↓
Improve descriptions
```

### If the toolkit is overloaded

```text
Too many tools
      ↓
Address tool scope / overload
```

Rewriting dozens of descriptions does not remove excessive choice.

---

## 7. Choosing the Right Fix

| Problem | First consideration |
|---|---|
| Similar tools have vague descriptions | Improve descriptions |
| Generic tool has broad responsibilities | Split the tool |
| Tool names are confusing | Rename tools |
| System prompt creates conflicting associations | Review prompt wording |
| Too many tools overall | Reduce/scope tool access or address overload |

---

## 8. Why Some Common Answers Are Wrong

### Few-shot examples

For description-based misrouting, few-shot examples add token overhead without directly fixing the weak descriptions.

### Routing classifier

A routing classifier may be useful architecturally, but it is over-engineered as the first response to simple description ambiguity.

### Tool consolidation

Consolidating tools can be a valid long-term design, but it costs more effort than improving descriptions.

### Exam Rule

> **Prefer the lowest-effort, highest-leverage fix that addresses the root cause.**

---

## 9. Practice Scenario

Production logs show that an agent frequently calls `get_customer` when users ask:

> "Check my order #12345."

Both tools have minimal descriptions:

```text
get_customer:
"Retrieves customer information"

lookup_order:
"Retrieves order details"
```

### Correct Answer

**Expand each tool description to include:**

- Input formats
- Example queries
- Edge cases
- Explicit boundaries
- When to use it versus similar tools

### Why?

The root cause is weak differentiation.

```text
Minimal descriptions
        ↓
Tool confusion
        ↓
Misrouting
        ↓
Better descriptions
        ↓
Clearer selection
```

---

## 10. CCAF Exam Traps

- **Few-shot examples as the first fix:** wrong for minimal-description ambiguity.
- **Routing classifier as the first fix:** over-engineered for a simple description problem.
- **Immediate tool consolidation:** potentially valid long-term, but not the proportionate first fix.
- **Ignoring system prompt wording:** broad instructions can interfere with tool selection.
- **Assuming descriptions solve every problem:** excessive tool count is a separate issue.

---

## 11. CCAF Checklist

- [ ] Tool descriptions are the primary mechanism for tool selection
- [ ] Know the five elements of a strong description
- [ ] Understand tool misrouting
- [ ] Know when to improve descriptions
- [ ] Understand tool splitting
- [ ] Understand tool renaming
- [ ] Understand system-prompt interactions
- [ ] Distinguish description ambiguity from tool overload
- [ ] Recognize common exam traps
- [ ] Prefer low-effort, high-leverage fixes

## One-Line Memory Aid

> **Clear purpose + clear inputs + examples + limitations + boundaries = reliable tool selection.**
