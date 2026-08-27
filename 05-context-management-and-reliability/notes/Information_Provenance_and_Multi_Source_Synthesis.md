# Information Provenance & Multi-Source Synthesis

## Reliability & Context Management

### What You Need to Know

**Information provenance** is the ability to track where a piece of information came from, how it was obtained, and what evidence supports it.

**Multi-source synthesis** is the process of combining information from multiple sources into a coherent result while preserving the distinction between:

- What each source actually says
- Which claims are supported by multiple sources
- Where sources disagree
- What remains uncertain or unverified

The core principle is:

> **A synthesis should preserve the provenance and confidence of important claims rather than presenting every source-derived statement as equally certain.**

---

# What Is Information Provenance?

Think of provenance as the **history of a piece of information**.

For every important claim, you want to be able to answer:

```text
Where did this come from?
        ↓
Which source?
        ↓
Which document / section / record?
        ↓
What evidence supports the claim?
        ↓
Was it directly observed, inferred, or synthesized?
```

### Example

Suppose an agent reports:

```text
"The payment terms are Net 30."
```

A provenance-aware result might contain:

```json
{
  "claim": "Payment terms are Net 30",
  "source": "contract.pdf",
  "location": "Section 4",
  "evidence": "Payment due within 30 days",
  "confidence": 0.96
}
```

Now the claim can be traced back to its source.

---

# Why Provenance Matters

Without provenance:

```text
Source A
Source B
Source C
   ↓
AI synthesis
   ↓
Final answer
```

The reader may not know which source supports which statement.

With provenance:

```text
Source A ──→ Claim 1
Source B ──→ Claim 2
Source A+B ─→ Claim 3
Source C ──→ Conflicting Claim
```

This makes the final answer:

- Easier to verify
- Easier to audit
- Easier to correct
- More transparent
- Safer for high-impact decisions

---

# Provenance vs Citation

These concepts are related but not identical.

### Citation

A citation tells the reader:

```text
"Where can I find this information?"
```

### Provenance

Provenance provides a broader record:

```text
Where did the information originate?
How was it obtained?
What transformation occurred?
Which evidence supports it?
```

For example:

```text
Original document
      ↓
Extracted field
      ↓
Validation
      ↓
Normalized value
      ↓
Final synthesis
```

Provenance can track this entire chain.

---

# Direct Evidence vs Inference

A critical provenance distinction is whether a claim was:

1. **Directly observed**
2. **Inferred**
3. **Synthesized from multiple sources**

### Directly observed

Source says:

```text
"Payment due within 30 days."
```

Claim:

```text
Payment terms are Net 30.
```

This is directly supported.

### Inferred

Source says:

```text
Invoice issued on June 1.
Payment received on July 1.
```

An agent might infer:

```text
Payment occurred approximately 30 days after issuance.
```

That is an inference, not a directly stated payment term.

### Synthesized

Source A:

```text
Product version = 4.2
```

Source B:

```text
Version 4.2 introduced feature X.
```

Synthesis:

```text
Feature X is available in product version 4.2.
```

This claim is supported by combining evidence.

---

# Provenance Should Survive Transformation

Information often passes through multiple agents.

Example:

```text
Source document
      ↓
Extraction agent
      ↓
Validation agent
      ↓
Synthesis agent
      ↓
Final answer
```

A common reliability problem occurs when the provenance is lost during these transformations.

### Poor design

```json
{
  "result": "Net 30"
}
```

The downstream agent no longer knows where the value came from.

### Better design

```json
{
  "result": "Net 30",
  "provenance": {
    "source": "contract.pdf",
    "location": "section_4",
    "evidence": "Payment due within 30 days"
  }
}
```

The downstream agent can preserve the evidence.

---

# Multi-Source Synthesis

Suppose an agent needs to answer a question using three sources:

```text
Source A → Official documentation
Source B → Technical paper
Source C → Internal project notes
```

The synthesis process should not simply concatenate them.

Instead:

```text
Collect sources
      ↓
Extract relevant claims
      ↓
Track provenance
      ↓
Compare claims
      ↓
Identify agreement
      ↓
Identify conflicts
      ↓
Assess source authority
      ↓
Synthesize
      ↓
Preserve uncertainty
```

---

# Source Authority

Not all sources should automatically have equal weight.

For example:

```text
Official documentation
        ↓
Primary technical paper
        ↓
Internal notes
        ↓
Secondary article
        ↓
Unverified forum comment
```

The exact hierarchy depends on the task.

The key principle is:

> **Source authority should be considered when sources disagree.**

Do not resolve conflicting claims merely by counting how many sources support each side.

---

# Majority Vote Is Not Always Correct

Suppose:

```text
Source A → Date = June 10
Source B → Date = June 10
Source C → Date = June 12
```

A simple majority vote says:

```text
June 10
```

But suppose Source C is the **official event organizer** and the other two sources are old articles.

Then the correct synthesis may be:

```text
June 12 according to the latest official source.
```

### Principle

> **Evidence quality and source authority can matter more than raw source count.**

---

# Source Recency

Recency can also matter.

Suppose:

```text
Source A, updated January:
API version = 3

Source B, updated August:
API version = 4
```

The latest authoritative source may supersede the older information.

A synthesis system should consider:

- Publication date
- Last updated date
- Version
- Whether the source explicitly supersedes older information

---

# Conflicting Sources

Conflicting evidence should be surfaced rather than silently averaged away.

Example:

```text
Source A:
Maximum payload = 10 MB

Source B:
Maximum payload = 20 MB
```

A bad synthesis:

```text
Maximum payload is approximately 15 MB.
```

There is no basis for averaging the values.

A better result:

```text
Sources disagree:
- Source A reports 10 MB.
- Source B reports 20 MB.

The latest official documentation reports 20 MB.
```

If authority cannot resolve the conflict:

```text
The available sources conflict, and the correct limit could not be verified.
```

---

# Contradiction Handling

A useful contradiction workflow is:

```text
Detect conflict
      ↓
Preserve both claims
      ↓
Compare source authority
      ↓
Compare recency
      ↓
Check context/version
      ↓
Search for authoritative clarification
      ↓
Resolve OR report unresolved conflict
```

Never silently delete the conflicting claim.

---

# Example: Version-Specific Conflict

Suppose:

```text
Source A:
Feature X available in version 3.

Source B:
Feature X unavailable in version 3.
```

Before declaring a contradiction, check whether the sources refer to different contexts:

```text
Source A → Version 3.2
Source B → Version 3.0
```

The apparent conflict may disappear once version information is included.

This is why provenance should preserve contextual metadata.

---

# Provenance Metadata

Useful provenance fields can include:

```json
{
  "source_id": "doc-123",
  "source_type": "official_documentation",
  "title": "API Reference",
  "location": "section_5.2",
  "retrieved_at": "2026-08-27T10:00:00Z",
  "version": "4.2",
  "claim": "Maximum payload is 20 MB",
  "evidence": "The request body may not exceed 20 MB."
}
```

Not every system needs every field.

The appropriate provenance schema depends on the workflow.

---

# Source IDs

Assigning stable source identifiers makes multi-source synthesis easier.

Example:

```text
SRC-001 → Official documentation
SRC-002 → Technical paper
SRC-003 → Internal design document
```

A synthesized claim can then reference:

```json
{
  "claim": "Feature X is supported in version 4.2",
  "sources": [
    "SRC-001",
    "SRC-002"
  ]
}
```

---

# Claim-Level Provenance

A strong synthesis system tracks provenance at the **claim level**, not merely at the document level.

Suppose a report contains:

```text
Feature X is supported in version 4.2.
The feature has a 20 MB payload limit.
It was introduced in 2025.
```

Different claims may come from different sources:

```text
Claim 1 → SRC-001
Claim 2 → SRC-001 + SRC-002
Claim 3 → SRC-003
```

This allows each claim to be independently verified.

---

# Provenance in Structured Output

A useful structured output might look like:

```json
{
  "claims": [
    {
      "text": "Feature X is supported in version 4.2.",
      "sources": ["SRC-001"],
      "support": "direct"
    },
    {
      "text": "Feature X has a 20 MB payload limit.",
      "sources": ["SRC-001", "SRC-002"],
      "support": "corroborated"
    },
    {
      "text": "Feature X was introduced in 2025.",
      "sources": ["SRC-003"],
      "support": "direct"
    }
  ]
}
```

This is much more reliable than returning a single untraceable paragraph.

---

# Corroboration

When multiple independent sources support the same claim, the claim may have stronger evidentiary support.

Example:

```text
Source A → Feature X exists
Source B → Feature X exists
Source C → Feature X exists
```

This is **corroboration**.

But be careful:

> Three sources repeating the same original source are not necessarily three independent confirmations.

For example:

```text
Article A cites official documentation.
Article B copies Article A.
Article C copies Article B.
```

Counting them as three independent sources would exaggerate the evidence.

---

# Source Independence

A good multi-source synthesis should consider whether sources are genuinely independent.

### Independent

```text
Official documentation
+
Independent technical benchmark
```

### Not truly independent

```text
Article A
   ↓
Article B copies A
   ↓
Article C copies B
```

All three may ultimately derive from the same underlying claim.

Provenance helps reveal this dependency chain.

---

# Evidence Coverage

A synthesis should also communicate whether all requested areas were covered.

Suppose the task asks for:

```text
Architecture
Security
Pricing
Performance
```

Research results:

```text
Architecture → complete
Security → complete
Pricing → partial
Performance → unavailable
```

The final synthesis should communicate the coverage.

Example:

```text
Architecture: supported by 3 sources.
Security: supported by 2 sources.
Pricing: partial coverage; latest pricing source unavailable.
Performance: not verified.
```

This prevents the reader from assuming that every section received equal investigation.

---

# Multi-Agent Provenance

Provenance is especially important when multiple agents contribute.

Example:

```text
Coordinator
    ↓
┌─────────────┬─────────────┬─────────────┐
Agent A       Agent B       Agent C
Security      Pricing       Architecture
    ↓             ↓             ↓
          Synthesis Agent
                 ↓
             Final report
```

Each finding should retain:

```text
Agent
Source
Evidence
Status
Confidence
```

Example:

```json
{
  "finding": "Feature X requires authentication",
  "agent": "security-agent",
  "source": "official-api-docs",
  "support": "direct",
  "confidence": 0.94
}
```

---

# Provenance and Coverage Annotations

These concepts complement each other.

### Provenance answers:

> **Where did this claim come from?**

### Coverage answers:

> **How completely was the task investigated?**

Example:

```text
Claim:
API requires authentication.

Provenance:
Official API documentation, section 2.

Coverage:
Authentication research complete.
```

For a multi-agent synthesis:

```text
4 agents expected
2 complete
1 partial
1 failed
```

The synthesis should expose this limitation.

---

# Provenance and Context Management

As context is compressed, provenance can easily be lost.

### Poor summary

```text
The API requires authentication.
```

### Better summary

```text
Finding:
API requires authentication.

Source:
Official API documentation, Authentication section.

Support:
Direct.

Confidence:
High.
```

The second summary preserves the information needed for later verification.

---

# Provenance and Context Degradation

Context degradation can cause a model to remember the conclusion but forget its source.

Earlier:

```text
OrderRepository.findById() uses custom caching.
Source:
src/repos/order.ts
```

Later:

```text
The repository layer probably handles caching.
```

The model has lost:

- Exact reference
- Evidence
- Specificity

This is why important findings should retain provenance during context compression.

---

# Provenance and Scratchpad Files

A scratchpad can preserve research findings together with their sources.

Example:

```markdown
# Research Notes

## Finding 1
Feature X is supported in version 4.2.

Source:
- SRC-001
- API documentation, section 4.1

Support:
Direct

## Finding 2
Payload limit is 20 MB.

Sources:
- SRC-001
- SRC-002

Support:
Corroborated

## Open Question
Pricing information could not be verified.
```

This makes the scratchpad much more useful than a list of unsupported conclusions.

---

# Provenance and Human Review

Human reviewers should be able to inspect the evidence behind an AI decision.

Example:

```text
AI decision:
Approve extraction

Confidence:
0.91

Evidence:
Invoice total found in Section 3.

Source:
invoice_123.pdf

Validation:
Line items reconcile with stated total.
```

If the reviewer disagrees, they can inspect the source rather than evaluating an unexplained model conclusion.

---

# Provenance for Auditable Workflows

For high-impact workflows, provenance can support:

- Auditing
- Debugging
- Error investigation
- Human review
- Compliance processes
- Reproducibility
- Model evaluation

If a final answer is later found to be wrong, provenance helps answer:

```text
Which source caused the error?
Which agent introduced it?
Was the source outdated?
Was the claim inferred incorrectly?
Was a conflict ignored?
```

---

# Common Failure Modes

## 1. Source Loss

The system remembers the conclusion but not the source.

```text
"Feature X exists."
```

No evidence.

### Fix

Preserve claim-level provenance.

---

## 2. Citation Drift

A citation is attached to a paragraph even though it does not actually support every claim in the paragraph.

### Fix

Associate sources with the specific claims they support.

---

## 3. False Corroboration

Several sources appear to agree, but they all copied the same original source.

### Fix

Track source independence and origin.

---

## 4. Silent Conflict Resolution

Two sources disagree, but the synthesis picks one without explaining why.

### Fix

Compare authority, recency, context, and version; report unresolved conflicts.

---

## 5. Majority-Count Fallacy

The system assumes:

```text
3 sources vs 1 source
```

means the three-source position is correct.

### Fix

Consider source authority and evidence quality.

---

## 6. Unsupported Inference

The source does not directly state the conclusion, but the synthesis presents it as fact.

### Fix

Label claims as:

```text
Direct
Inferred
Synthesized
```

---

## 7. Stale Source

An older document conflicts with a newer authoritative source.

### Fix

Track dates and versions.

---

## 8. Coverage Blindness

The final report sounds complete even though some research branches failed.

### Fix

Include coverage annotations.

---

# Practice Scenario 1

Three articles say that an API supports Feature X. All three articles ultimately cite the same official documentation page.

What should the synthesis system conclude?

**A.** Three independent sources confirmed Feature X.

**B.** The evidence should be treated as three independent confirmations.

**C.** The articles may provide corroborating references, but they are not necessarily independent evidence because they share the same underlying source.

**D.** Ignore the official documentation.

### Correct Answer

**C**

Provenance reveals that the apparent three-source agreement may trace back to one source.

---

# Practice Scenario 2

Two sources disagree about an API limit:

```text
Source A → 10 MB
Source B → 20 MB
```

Source B is the latest official documentation.

What is the best synthesis?

**A.** Average the values to 15 MB.

**B.** Pick 10 MB because it appears first.

**C.** Prefer the latest authoritative source and explain the conflict if relevant.

**D.** Pick the value with the highest number.

### Correct Answer

**C**

Source authority and recency matter more than arbitrary averaging.

---

# Practice Scenario 3

A synthesis agent reports:

```text
"Feature X was introduced in 2025."
```

The only supporting source actually says:

```text
"Feature X was available in 2025."
```

What is the issue?

**A.** The claim may be an unsupported inference.

**B.** The source is automatically invalid.

**C.** Provenance is unnecessary.

**D.** The claim is guaranteed to be correct.

### Correct Answer

**A**

“Available in 2025” does not necessarily establish that the feature was introduced in 2025.

---

# Practice Scenario 4

A research task has four areas:

```text
Architecture → complete
Security → complete
Pricing → partial
Performance → unavailable
```

What should the final synthesis do?

**A.** Present all four areas as fully researched.

**B.** Omit the missing areas without explanation.

**C.** Include coverage information and clearly identify partial or unavailable areas.

**D.** Treat unavailable information as negative evidence.

### Correct Answer

**C**

Coverage annotations prevent incomplete research from being mistaken for complete research.

---

# Practice Scenario 5

An agent extracts a value from a document and passes it to a synthesis agent.

Which structured representation is more reliable?

**A.**

```json
{
  "value": "Net 30"
}
```

**B.**

```json
{
  "value": "Net 30",
  "source": "contract.pdf",
  "location": "section_4",
  "support": "direct"
}
```

### Correct Answer

**B**

The second representation preserves provenance.

---

# Practice Scenario 6

An internal document says:

```text
API version 3 supports Feature X.
```

The latest official documentation says:

```text
API version 3 does not support Feature X.
```

What should the system do?

**A.** Choose whichever source appears first.

**B.** Count the number of words in each source.

**C.** Compare source authority, recency, version/context, and report or resolve the conflict.

**D.** Average the two claims.

### Correct Answer

**C**

Conflicts require evidence-based resolution rather than arbitrary selection.

---

# Build Pattern

A practical provenance-aware synthesis workflow:

### Step 1 — Identify sources

Assign stable source IDs.

```text
SRC-001
SRC-002
SRC-003
```

### Step 2 — Extract claims

Separate important claims from raw source content.

### Step 3 — Attach provenance

For each claim, preserve:

```text
Source
Location
Evidence
Version/date
Agent
```

where relevant.

### Step 4 — Classify support

Mark claims as:

```text
Direct
Inferred
Synthesized
Corroborated
Conflicted
```

### Step 5 — Compare sources

Check:

- Agreement
- Contradictions
- Authority
- Recency
- Version
- Source independence

### Step 6 — Resolve conflicts carefully

Use authoritative evidence where possible.

If unresolved:

```text
Preserve both claims
        ↓
Explain conflict
        ↓
Mark unresolved
```

### Step 7 — Track coverage

Record:

```text
Complete
Partial
Unavailable
```

for relevant research branches.

### Step 8 — Synthesize

Combine supported claims into a coherent answer.

### Step 9 — Preserve provenance in the final result

Important claims should remain traceable.

---

# Recommended Structured Schema

A useful general schema is:

```json
{
  "claims": [
    {
      "claim": "Feature X is supported in version 4.2.",
      "sources": [
        {
          "source_id": "SRC-001",
          "location": "section_4.1"
        }
      ],
      "support": "direct",
      "confidence": 0.94
    }
  ],
  "conflicts": [],
  "coverage": {
    "status": "complete"
  }
}
```

For a conflict:

```json
{
  "claims": [
    {
      "claim": "Limit is 10 MB.",
      "sources": ["SRC-001"]
    },
    {
      "claim": "Limit is 20 MB.",
      "sources": ["SRC-002"]
    }
  ],
  "conflicts": [
    {
      "topic": "payload_limit",
      "status": "unresolved"
    }
  ]
}
```

---

# Exam Traps

## Trap 1 — “More sources always means stronger evidence.”

**Wrong.**

Multiple sources may all originate from the same underlying source.

---

## Trap 2 — “Use majority vote to resolve conflicts.”

**Not necessarily.**

Authority, recency, version, and evidence quality can matter more than source count.

---

## Trap 3 — “Citations are optional if the synthesis is confident.”

**Wrong for provenance-sensitive workflows.**

Important claims should remain traceable to supporting evidence.

---

## Trap 4 — “If a source implies something, report it as directly stated.”

**Wrong.**

Distinguish direct evidence from inference.

---

## Trap 5 — “Average conflicting values.”

**Wrong.**

Conflicting evidence requires investigation, not mathematical averaging.

---

## Trap 6 — “If one source is missing, just ignore the gap.”

**Wrong.**

The final result should communicate relevant coverage limitations.

---

## Trap 7 — “Document-level provenance is enough.”

**Not always.**

Claim-level provenance is more useful when different claims within the same document have different evidence.

---

## Trap 8 — “An old authoritative source always beats a newer secondary source.”

**Not necessarily.**

Compare both authority and recency, and consider whether the newer source is authoritative for the specific question.

---

# Exam Memory Aid

Remember:

> **Every important claim should have a traceable source.**
>
> **Distinguish direct evidence from inference.**
>
> **Do not count copied sources as independent corroboration.**
>
> **Resolve conflicts using authority, recency, and context.**
>
> **Never silently average conflicting claims.**
>
> **Expose coverage gaps.**

### Quick Decision Rules

| Situation | Recommended action |
|---|---|
| Important claim | Preserve provenance |
| Direct source statement | Mark as direct |
| Conclusion derived from evidence | Mark as inferred |
| Multiple genuinely independent sources agree | Corroboration |
| Multiple sources copy one source | Not independent corroboration |
| Sources conflict | Investigate authority/recency/context |
| Conflict unresolved | Report the disagreement |
| Research branch incomplete | Add coverage annotation |
| Different versions involved | Preserve version metadata |
| Downstream synthesis | Carry provenance forward |

---

# Final Takeaway

**Information provenance is the chain of evidence behind a claim.**

**Multi-source synthesis is the careful combination of claims from multiple sources without losing that chain.**

A reliable workflow looks like:

```text
Sources
   ↓
Extract claims
   ↓
Attach provenance
   ↓
Check source independence
   ↓
Compare evidence
   ↓
Resolve conflicts
   ↓
Assess coverage
   ↓
Synthesize
   ↓
Preserve provenance
```

The most important mental model is:

```text
Claim
 ↓
Evidence
 ↓
Source
 ↓
Context / version / date
```

And when several sources are involved:

```text
Source A ──┐
Source B ──┼──→ Compare ──→ Synthesize
Source C ──┘       │
                   ↓
              Conflict?
              /       \
            No        Yes
            ↓          ↓
        Synthesize   Investigate
                       ↓
                  Resolve OR
                  expose conflict
```

The goal is **not merely to produce an answer**.

The goal is to produce an answer where important conclusions can be **traced, checked, challenged, and corrected**.
