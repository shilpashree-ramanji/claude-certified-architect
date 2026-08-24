# CCAF Domain 1 — Session State and Resumption

## What You Need to Know

Session management determines how an agent maintains continuity across work sessions.

In long-running tasks, context can accumulate:

- Tool results
- File analyses
- Reasoning history
- Prior findings

The key decision is whether to:

1. Continue an existing session
2. Branch into an independent session
3. Start fresh while preserving useful knowledge

---

# 1. Three Session Management Options

| Option | Purpose |
|---|---|
| `--resume <session-name>` | Continue the same session |
| `fork_session` | Explore a divergent approach from a shared baseline |
| Fresh start + summary injection | Start clean when previous tool results are stale |

### Memory aid

> **Resume = Continue**  
> **Fork = Diverge**  
> **Fresh + Summary = Clean baseline**

---

# 2. `--resume <session-name>`

Resume continues an existing named session from where it stopped.

The prior conversation history is restored, including previous tool results and analyses.

## Use `--resume` when

- Prior context is still valid
- Files have not changed significantly
- You want to continue exactly where you stopped

Example:

```text
Day 1
  ↓
Analyze codebase
  ↓
Stop
  ↓
Day 2: --resume
  ↓
Continue same investigation
```

## Do NOT prefer resume when

Important files have changed since the previous session.

Old tool results remain in the restored conversation and may no longer represent the current state.

---

# 3. Session Naming and Resuming

According to the supplied study material:

`--resume` resumes sessions that already exist; it does not create a session.

A session can be named when starting it with:

```text
--name
```

or:

```text
-n
```

A session can also be renamed during a session with:

```text
/rename
```

Later, it can be resumed with:

```text
claude --resume <name>
```

There is also:

```text
-c
```

or:

```text
--continue
```

which resumes the most recent conversation in the current directory without naming it.

---

# 4. `fork_session`

`fork_session` creates an independent branch from a shared analysis baseline.

```text
Shared baseline
       ↓
   ┌───┴───┐
   ↓       ↓
Branch A  Branch B
   ↓       ↓
Approach A Approach B
```

After the fork:

- Each branch operates independently
- Changes in one branch do not affect the other
- Branches cannot see each other's later results

## Use `fork_session` when

You want to explore **different approaches** from the same starting point.

Example:

```text
Initial analysis
      ↓
     Fork
    /    \
   ↓      ↓
Refactor A  Refactor B
```

Both approaches start from the same baseline.

## Do NOT use it when

You simply want to continue the same investigation.

> **Fork is for divergence, not continuation.**

---

# 5. Fresh Start with Summary Injection

A fresh session starts without the old conversation history and stale tool results.

Important knowledge is preserved through a structured summary.

```text
Old session
    ↓
Important findings
    ↓
Structured summary
    ↓
NEW session
    ↓
Fresh analysis
```

## Use this when

- Files have changed
- APIs have changed
- Dependencies have shifted
- Prior tool results are stale
- A long session has too much irrelevant context
- You need a clean baseline while preserving important findings

Example:

```text
Prior findings:
- Authentication issue identified
- Three files were involved
- Issues were fixed

Changed files:
- auth.ts
- session.ts
- middleware.ts

New session:
Re-analyze the changed files.
```

---

# 6. The Stale Context Problem

This is the central concept of this task statement.

Stale context occurs when an agent resumes a session after code or file changes and reasons from old tool results that no longer reflect the current state.

## Example

```text
Day 1
 ↓
Agent reads auth.ts
 ↓
Old version is stored in conversation
 ↓
Developer changes auth.ts
 ↓
Day 2
 ↓
Agent resumes session
 ↓
Old tool result still exists
```

The model may now reason from:

```text
Old file contents
+
New file contents
```

This can produce contradictory advice.

---

# 7. How Stale Context Manifests

A developer modifies:

```text
auth.ts
session.ts
middleware.ts
```

After resuming, the agent may:

- Recommend fixes that were already completed
- Refer to code that no longer exists
- Give contradictory answers
- Mix old and current versions of files

The issue is that outdated tool results remain in the restored conversation history.

---

# 8. Naive Fix — Re-read the Changed Files

A tempting solution is:

> Resume the old session and ask the agent to re-read the modified files.

This can help, but it is not the preferred solution for the stale-context problem described in the supplied material.

Why?

The old tool results are still present:

```text
Old tool result
      ↓
Still in history
      ↓
New tool result
      ↓
Both may influence reasoning
```

---

# 9. Correct Fix — Fresh Session + Structured Summary

For stale context, the recommended approach is:

```text
Old session
    ↓
Create structured summary
    ↓
Start fresh session
    ↓
Identify changed files
    ↓
Targeted re-analysis
```

Example:

> Prior analysis identified three authentication issues in `auth.ts`, `session.ts`, and `middleware.ts`. All three have been fixed. Re-analyze these three files to verify the fixes and check for new issues.

The new session has:

- Curated prior knowledge
- Current file contents
- No stale tool results from the old session

---

# 10. Targeted Re-Analysis

When only a few files changed, do not unnecessarily re-analyze the entire codebase.

Example:

```text
50-file codebase
      ↓
3 files changed
      ↓
Re-analyze 3 files
      +
Use summary for unchanged findings
```

## Steps

1. Start a fresh session.
2. Inject a structured summary of prior findings.
3. Identify the files that changed.
4. Re-read and re-analyze only those files.
5. Combine fresh analysis with the preserved summary.

This is more efficient than full re-exploration.

---

# 11. Decision Matrix

| Scenario | Best option | Reason |
|---|---|---|
| Continue work from yesterday, no files changed | `--resume` | Prior context is valid |
| Compare two refactoring approaches | `fork_session` | Divergent exploration |
| 3 of 50 files changed | Fresh + summary | Avoid stale results; targeted re-analysis |
| Long session with cluttered history | Fresh + summary | Clean baseline |
| Compare testing vs documentation strategy | `fork_session` | Independent approaches |
| Dependency updates occurred | Fresh + summary | Multiple files may have changed indirectly |

---

# 12. Practical Example — Contradictory Advice

A developer analyzes a 50-file codebase over two days.

### Day 1

The agent identifies three authentication issues in:

```text
auth.ts
session.ts
middleware.ts
```

### Overnight

The developer fixes all three.

### Day 2

The developer resumes the session.

The old tool results are still in the conversation.

The agent recommends fixing the same issues again.

It may also give contradictory advice because it sees:

```text
Old tool result → old code
New tool result → current code
```

### Better approach

Start a fresh session and inject:

> Prior analysis identified three authentication issues in `auth.ts`, `session.ts`, and `middleware.ts`. All three have been fixed. Re-analyze these files to verify the fixes and check for new issues.

Now the agent works from a clean baseline.

---

# 13. Exam Traps

## Trap 1 — Full re-exploration

> “Three files changed, so re-analyze all 50 files.”

**Wrong / inefficient.**

Use targeted re-analysis of the changed files while preserving prior findings in a summary.

## Trap 2 — Resume after important file changes

> “Just use `--resume`; the agent can read the new files.”

**Not the preferred answer when stale tool results are a concern.**

The old results remain in the restored history.

## Trap 3 — Confusing `fork_session` and `--resume`

```text
resume → continue the same line of investigation
fork   → explore a different approach
```

## Trap 4 — Fork for stale context

A fork starts from the existing session baseline, so it can inherit stale tool results.

For stale data:

> **Fresh session + structured summary**

---

# 14. Master Decision Tree

```text
Need to continue previous work?
          ↓
         YES
          ↓
Did important files / dependencies change?
       ↙        ↘
      NO        YES
      ↓           ↓
  --resume    Fresh session
                  ↓
          Structured summary
                  ↓
          Targeted re-analysis
```

If the question is instead:

```text
Do I want to explore different approaches?
          ↓
         YES
          ↓
    fork_session
```

---

# 15. CCAF Checklist

- [ ] Understand session state
- [ ] Understand `--resume`
- [ ] Know when resume is appropriate
- [ ] Know when resume can create stale-context problems
- [ ] Understand `fork_session`
- [ ] Know that fork is for divergent exploration
- [ ] Understand fresh session + summary injection
- [ ] Understand stale tool results
- [ ] Understand targeted re-analysis
- [ ] Distinguish targeted re-analysis from full re-exploration
- [ ] Know the session decision matrix
- [ ] Recognize session-management exam traps

## One-Line Memory Aid

> **Resume = same work. Fork = different path. Fresh + summary = changed/stale world.**
