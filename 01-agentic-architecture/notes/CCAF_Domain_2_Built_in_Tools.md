# CCAF Domain 2 — Built-in Tools

## What You Need to Know

The supplied study material identifies six Claude Code built-in tools:

- **Read**
- **Write**
- **Edit**
- **Bash**
- **Grep**
- **Glob**

Each tool has a specific purpose. Choosing the wrong tool can waste:

- Time
- Context tokens
- Both

---

# 1. Grep vs Glob — The Most Important Distinction

This is the core distinction to remember.

## Grep

> **Grep searches file CONTENTS.**

Use Grep when you need to find something **inside files**.

Examples:

```text
Find all callers of processLegacyOrder()
→ Grep: "processLegacyOrder"

Find error messages containing "timeout"
→ Grep: "timeout"

Find imports of a specific module
→ Grep: "import.*from 'utils/auth'"
```

Use Grep for:

- Function references
- Error messages
- Imports
- Variable assignments
- Text patterns
- Content-based searches

### Memory aid

> **Grep = What's INSIDE the file?**

---

# 2. Glob

> **Glob matches file PATHS.**

Use Glob when you need to find files based on:

- Name
- Extension
- Directory structure
- Path pattern

Examples:

```text
Find all test files
→ Glob: "**/*.test.tsx"

Find configuration files
→ Glob: "**/config.*"

Find MDX files in domains
→ Glob: "content/domains/**/*.mdx"
```

### Memory aid

> **Glob = Which FILES / PATHS?**

---

# 3. Grep vs Glob Quick Table

| Need to find... | Tool |
|---|---|
| Function callers | **Grep** |
| Error messages | **Grep** |
| Import statements | **Grep** |
| Variable assignments | **Grep** |
| Files by extension | **Glob** |
| Files by naming pattern | **Glob** |
| Files by directory pattern | **Glob** |
| Test files by filename pattern | **Glob** |

### One-line rule

> **Grep finds what is INSIDE files. Glob finds files by their NAMES / PATHS.**

---

# 4. Read, Write, and Edit

These tools handle file operations.

## Read

Use **Read** to inspect file contents.

Typical flow:

```text
Discover relevant file
       ↓
Read file
       ↓
Understand code
```

Read should be targeted rather than used to load an entire codebase unnecessarily.

---

## Write

Use **Write** when you need to create or replace file contents.

It writes the specified content to a file.

---

## Edit

Use **Edit** for targeted modifications.

Example:

```text
old_string:
function processOrder(id: string)

new_string:
function processOrder(
  id: string,
  validate: boolean = true
)
```

Edit is preferred for focused changes because it modifies only the targeted text.

### Memory aid

> **Read = inspect. Write = create/replace. Edit = targeted change.**

---

# 5. Edit and Unique Matching

Edit relies on matching the specified text.

If the selected `old_string` occurs multiple times, Edit cannot determine which occurrence should be changed.

That is a safety mechanism.

Example:

```text
File contains:

const status = "active";
...
const status = "active";
```

If the Edit anchor is only:

```text
const status = "active";
```

the match is not unique.

---

# 6. What to Do When Edit Has a Non-Unique Match

Do **not** immediately switch to Read + Write.

The supplied material recommends:

### Option 1 — Widen the anchor

Add surrounding context until the target is unique.

```text
old_string:

function processOrder(id: string) {
    const status = "active";
}
```

Now the match may uniquely identify the intended location.

### Option 2 — Use `replace_all: true`

Use this only when you actually want every occurrence changed.

```text
replace_all: true
```

### Last resort

If neither approach can disambiguate the target:

```text
Read
  ↓
Modify
  ↓
Write
```

### Correct ordering

1. Try Edit with a reasonably unique anchor.
2. If non-unique, widen `old_string` or use `replace_all: true` when appropriate.
3. Only then fall back to Read + Write.

### Exam trap

> **Non-unique Edit ≠ immediately use Read + Write.**

---

# 7. Incremental Codebase Understanding

How you explore a codebase matters.

## Wrong approach

```text
Read every file
      ↓
Load entire codebase
      ↓
Huge context consumption
```

A 200-file codebase can consume a large amount of context before you even know which files matter.

## Better approach

Use **incremental discovery**.

```text
Grep
 ↓
Find entry point
 ↓
Read relevant files
 ↓
Trace imports / flow
 ↓
Grep again if needed
 ↓
Read only newly relevant files
```

### Principle

> **Start narrow and expand only as needed.**

---

# 8. Incremental Exploration Workflow

### Step 1 — Grep for the entry point

Search for:

- Function name
- Class name
- Error message
- Other task-specific anchor

Example:

```text
Grep: "processLegacyOrder"
```

### Step 2 — Read relevant files

Read the files discovered by the initial search.

Use them to understand:

- Imports
- Exports
- Wrappers
- Control flow

### Step 3 — Grep again

If you discover a wrapper or re-export, search for that new name.

### Step 4 — Read only what is needed

Each additional file should be justified by something discovered in the previous step.

---

# 9. Tracing Function Usage Through Wrappers

A common codebase pattern is:

```text
Original function
      ↓
Wrapper
      ↓
Consumer
```

A single search for the original function name can miss consumers that use the wrapper.

## Correct approach

### Step 1

Grep for the function definition.

```text
Grep: "processLegacyOrder"
```

### Step 2

Read the defining file.

Determine:

- What is exported?
- Is it wrapped?
- Is it renamed?

### Step 3

Grep for exported or wrapper names.

```text
Grep: "applyLegacyOrder"
```

### Step 4

If a barrel file is involved, search for the barrel module name to find consumers importing through it.

---

# 10. Example Wrapper Trace

Suppose:

```text
processLegacyOrder()
        ↓
applyLegacyOrder()
        ↓
index.ts
        ↓
Consumer
```

A single:

```text
Grep: "processLegacyOrder"
```

may not find consumers using:

```text
applyLegacyOrder
```

So the investigation becomes:

```text
Grep original function
        ↓
Read defining file
        ↓
Find wrapper/export
        ↓
Grep wrapper
        ↓
Find consumers
```

---

# 11. Deprecation Scenario

A common exam scenario:

> Find every file that calls a deprecated function and also find the test files for those callers.

The supplied material recommends:

```text
Grep
  ↓
Glob
  ↓
Grep
```

## Step 1 — Grep for the deprecated function

Example:

```text
Grep: "processLegacyOrder"
```

This finds files whose **contents** directly reference the function.

Example results:

```text
OrderProcessor.ts
RefundHandler.ts
```

## Step 2 — Glob for sibling test files

Use path matching:

```text
Glob: "**/OrderProcessor.test.*"
Glob: "**/RefundHandler.test.*"
```

This can find tests that exercise the source module indirectly, even if the test does not mention the deprecated function by name.

## Step 3 — Grep for wrappers

If the source contains:

```text
applyLegacyOrder()
```

which internally calls:

```text
processLegacyOrder()
```

then:

```text
Grep: "applyLegacyOrder"
```

can identify tests and consumers that use the wrapper.

### Memory aid

> **Grep → Glob → Grep**

Content → paths → indirect content.

---

# 12. Why Glob Is Needed in the Deprecation Scenario

Suppose:

```text
OrderProcessor.ts
```

calls the deprecated function.

Its test might be:

```text
OrderProcessor.test.tsx
```

But that test might never contain:

```text
processLegacyOrder
```

because it tests the behavior through the module.

Therefore:

```text
Grep
```

alone is insufficient.

Use:

```text
Glob
```

to find the sibling test by path/name convention.

---

# 13. Six Built-in Tools — Quick Reference

| Tool | Primary purpose |
|---|---|
| **Read** | Inspect file contents |
| **Write** | Create/replace file contents |
| **Edit** | Make targeted text changes |
| **Bash** | Execute shell commands |
| **Grep** | Search file contents |
| **Glob** | Match file paths |

### Core pair

```text
Grep → content
Glob → paths
```

---

# 14. Tool Selection Decision Tree

```text
What are you trying to do?
          ↓
   ┌──────┼──────────┐
   ↓      ↓          ↓
Find     Inspect    Modify
   ↓      ↓          ↓
Content  Read      Edit
   ↓
 Grep
```

For path discovery:

```text
Need files by name/path?
        ↓
       Glob
```

For creating/replacing a file:

```text
Need to create/replace?
        ↓
      Write
```

For shell/system commands:

```text
Need shell command?
        ↓
      Bash
```

---

# 15. CCAF Exam Traps

## Trap 1 — Using Glob to find function callers

**Wrong.**

Glob searches paths.

Use:

```text
Grep
```

to search file contents.

## Trap 2 — Using Grep to find files by extension

Use:

```text
Glob
```

for:

```text
**/*.test.tsx
**/config.*
```

## Trap 3 — Reading all source files upfront

This wastes context.

Use incremental discovery:

```text
Grep → Read → Grep → Read
```

as needed.

## Trap 4 — Defaulting to Read + Write for modifications

Try:

```text
Edit
```

first.

## Trap 5 — Switching directly to Read + Write after non-unique Edit

Instead:

```text
Non-unique Edit
      ↓
Widen old_string
OR
replace_all: true
      ↓
Read + Write only if still necessary
```

## Trap 6 — Using only one Grep for wrapper-based usage

A wrapper or barrel export can hide indirect consumers.

Trace:

```text
Definition
→ Export
→ Wrapper
→ Consumer
```

---

# 16. Practice Scenario

### Scenario

A developer needs to:

1. Find all files that call the deprecated function `processLegacyOrder()`
2. Find all test files for those callers

### Correct approach

```text
Grep
  ↓
Find direct references
  ↓
Glob
  ↓
Find sibling test files
  ↓
Grep
  ↓
Find wrapper-based / indirect usage
```

### Best answer

The correct sequence is:

> **Grep → Glob → Grep**

### Why?

- **Grep** finds content references to the deprecated function.
- **Glob** finds related test files by path/name.
- **Grep again** finds wrapper-based or indirect consumers.

---

# 17. CCAF Checklist

- [ ] Know the six built-in tools
- [ ] Understand Grep
- [ ] Understand Glob
- [ ] Know the Grep vs Glob distinction
- [ ] Understand Read
- [ ] Understand Write
- [ ] Understand Edit
- [ ] Understand Bash
- [ ] Know Edit's unique matching behavior
- [ ] Know how to recover from a non-unique Edit
- [ ] Understand `replace_all: true`
- [ ] Understand incremental codebase exploration
- [ ] Understand wrapper/function tracing
- [ ] Understand the deprecation scenario
- [ ] Remember Grep → Glob → Grep
- [ ] Recognize common exam traps

## One-Line Memory Aids

> **Grep = inside the file.**

> **Glob = file path/name.**

> **Edit = targeted modification.**

> **Read + Write = fallback, not first choice for small edits.**

> **Explore incrementally: Grep → Read → Grep → Read.**

> **Deprecated-function scenario: Grep → Glob → Grep.**
