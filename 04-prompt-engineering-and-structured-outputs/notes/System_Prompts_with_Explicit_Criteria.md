# System Prompts with Explicit Criteria

## Domain 4 — Prompt Engineering & Structured Output

### Core Principle

**Explicit categorical criteria outperform vague instructions.**

Avoid vague instructions such as:

- “Be conservative.”
- “Only report high-confidence findings.”
- “Use your best judgement.”

Instead explicitly define:
- What to flag
- What to skip
- What conditions trigger a finding
- How findings should be classified

### Vague vs Explicit

**Weak:**
```text
Review this code. Be conservative. Only report high-confidence findings.
```

**Better:**
```text
Flag comments only when claimed behaviour contradicts actual code behaviour.
Report bugs and security vulnerabilities.
Skip minor style preferences and local patterns.
```

The second version provides concrete categories and decision boundaries.

---

## False Positive Trust Problem

A high false-positive rate in one category can damage trust in **all** categories.

Example:
- Documentation mismatch: 40% false positives
- Security findings: 98% accuracy

Developers may stop trusting even accurate security findings because the overall review output is noisy.

### Recommended Strategy

Temporarily **disable the high-false-positive category** while refining its prompt and criteria.

Then:
1. Define clearer criteria.
2. Add concrete examples.
3. Test again.
4. Re-enable when precision improves.

---

## Severity Calibration with Code Examples

Severity should be calibrated with **concrete code examples**, not only prose.

### Weak

```text
Critical: Issues that could cause system failures or data loss.
Minor: Issues that affect readability but not functionality.
```

### Better

```text
Critical — Unsanitised user input in SQL query:

query = f"SELECT * FROM users WHERE id = {user_input}"
```

```text
Minor — Inconsistent variable naming:

userName vs user_name in the same module
```

Concrete examples remove ambiguity and improve classification consistency.

---

## Why Confidence-Based Filtering Fails

A tempting instruction is:

```text
Only report high-confidence findings.
```

This is not a substitute for explicit criteria because LLM self-reported confidence can be poorly calibrated. A model may be confident about an incorrect finding or hesitant about a correct one.

**Hierarchy:**

> Explicit criteria first → confidence-based routing second.

Confidence is useful for deciding when to involve a human reviewer, but it should not define what qualifies as a valid finding.

---

## Decision Framework

| Requirement | Preferred approach |
|---|---|
| Define valid findings | Explicit categorical criteria |
| Define exclusions | Explicit skip criteria |
| Calibrate severity | Concrete code examples |
| Route uncertain findings | Confidence-based routing |
| High false positives | Temporarily disable and refine |
| Style-only preferences | Explicitly exclude when appropriate |

---

## Exam Traps

### 1. “Be conservative”

Wrong. It is vague and provides no actionable decision boundary.

### 2. “Only report high-confidence findings”

Wrong as the primary precision fix. Confidence does not replace explicit criteria.

### 3. Keep every review category enabled

Wrong when one category has a high false-positive rate. A noisy category can damage trust in the entire system.

### 4. Describe severity only in prose

Wrong. Use concrete code examples for severity calibration.

---

## Practice Scenario

A CI/CD code review pipeline has a **40% false-positive rate** for “documentation mismatch” findings. Developers have started ignoring all review categories, including highly accurate security findings.

What is the most effective fix?

**A.** Increase model temperature and filter findings that appear only once.

**B.** Temporarily disable the documentation-mismatch category while refining its prompts with explicit criteria and code examples.

**C.** Add “only report high-confidence documentation issues” to the system prompt.

**D.** Keep the category enabled and add more general instructions about being conservative.

### Correct Answer

**B**

The high-false-positive category is damaging trust across the entire review system. Temporarily disabling it while refining its criteria protects trust in the categories that already work.

---

## Build Pattern

### Step 1 — Establish a baseline

Test a vague prompt against known examples containing:
- Bugs
- Security vulnerabilities
- Style issues
- Documentation mismatches

### Step 2 — Replace vague instructions

Define explicit categories:

```text
Report:
- Bugs
- Security vulnerabilities

Skip:
- Minor style preferences
- Local patterns
```

### Step 3 — Add severity examples

Provide actual code snippets showing patterns for each severity level.

### Step 4 — Measure results

Compare vague and explicit versions using the same test set.

Measure:
- False-positive rate
- Classification consistency
- Missed findings
- Repeatability across runs

### Step 5 — Disable problematic categories

If a category produces unacceptable false positives, temporarily disable it and refine its criteria before re-enabling it.

---

## Exam Memory Aid

**Explicit criteria → concrete examples → measure → disable noisy categories → confidence routing**

> **Confidence is for routing uncertainty; explicit criteria define correctness.**

### Source

Based on the Claude Certification Guide lesson **“System Prompts with Explicit Criteria” (Task Statement 4.1).
