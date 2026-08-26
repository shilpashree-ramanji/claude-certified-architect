# CCAF Domain 3 — Path-Specific Rules for Conditional Convention Loading

## What You Need to Know

Path-specific rules let Claude Code apply conventions **only when it is working with files that match specified path patterns**.

They are useful when a convention applies to a particular file type or path pattern, especially when those files are scattered across different directories.

Path-specific rules live in:

```text
.claude/rules/
```

and use YAML frontmatter containing a:

```yaml
paths:
```

field.

---

# 1. Why Path-Specific Rules Exist

Imagine a project has Terraform files in several places:

```text
project/
├── infrastructure/
│   └── main.tf
├── environments/
│   ├── dev/
│   │   └── app.tf
│   └── prod/
│       └── app.tf
└── modules/
    └── network/
        └── network.tf
```

You want the same Terraform conventions for all Terraform files.

Putting those rules into the root `CLAUDE.md` makes them available even when Claude is working on unrelated files.

A path-specific rule lets the convention become conditional:

```text
Working with matching files
        ↓
Load the relevant rule

Working with unrelated files
        ↓
Do not load the path-specific rule
```

This reduces irrelevant context and keeps specialized rules targeted.

---

# 2. Where the Rules Live

Path-specific rules are stored under:

```text
.claude/rules/
```

Example:

```text
project/
└── .claude/
    └── rules/
        └── terraform.md
```

The rule file is ordinary Markdown with YAML frontmatter.

---

# 3. The `paths` Frontmatter

The key configuration is:

```yaml
---
paths:
  - "src/api/**/*.ts"
---
```

The `paths` value contains glob patterns.

The rule applies when Claude works with files matching those patterns.

---

# 4. Basic Example

```yaml
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation.
- Use the standard error response format.
- Include OpenAPI documentation comments.
```

Conceptually:

```text
Claude works with src/api/users.ts
             ↓
       Pattern matches
             ↓
       API rules apply
```

But:

```text
Claude works with src/components/Button.tsx
             ↓
       Pattern does not match
             ↓
       API rules do not apply
```

---

# 5. Glob Patterns

The `paths` field uses glob patterns.

Common examples:

| Pattern | Meaning |
|---|---|
| `**/*.ts` | All TypeScript files in any directory |
| `src/**/*` | All files under `src/` |
| `*.md` | Markdown files in the project root |
| `src/components/*.tsx` | Matching React component files in that directory |
| `**/*.test.ts` | TypeScript test files anywhere |
| `**/*.test.tsx` | TSX test files anywhere |

### Memory aid

> **Glob pattern = describes which files trigger the rule.**

---

# 6. File Type Example

Suppose your project has tests scattered across many directories:

```text
src/
├── api/
│   └── users.test.ts
├── services/
│   └── billing.test.ts
└── components/
    └── Button.test.tsx
```

You want the same testing conventions everywhere.

Use:

```yaml
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---

# Test Conventions

- Use describe/it blocks.
- Use test data factories.
- Follow the project's standard testing setup.
```

Now the rule can apply to tests regardless of where they are located.

---

# 7. Multiple Path Patterns

A rule can target multiple patterns.

Example:

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
---

# API Rules

- Validate all incoming data.
- Use the standard error format.
- Return appropriate HTTP status codes.
```

The rule applies when either pattern matches.

Conceptually:

```text
src/api/.../*.ts
        OR
src/routes/.../*.ts
        ↓
     API rules
```

---

# 8. Path-Specific Rules vs Unscoped Rules

A rule without `paths` is different.

### Unscoped rule

```yaml
---
---

# General Rules
...
```

This is loaded unconditionally.

### Path-scoped rule

```yaml
---
paths:
  - "src/api/**/*.ts"
---
```

This is loaded conditionally when Claude works with matching files.

### Memory aid

> **No `paths` = broadly applicable.**

> **With `paths` = conditionally applicable.**

---

# 9. Path-Specific Rules vs `CLAUDE.md`

This distinction is important for the exam.

## Use `CLAUDE.md` for:

- Project overview
- Technology stack
- Important commands
- Architecture summary
- General conventions
- Instructions needed broadly

Example:

```text
CLAUDE.md
    ↓
Project-wide guidance
```

## Use path-specific rules for:

- API-specific conventions
- Test-file conventions
- Terraform conventions
- Migration-file conventions
- Frontend-specific file conventions
- Other rules tied to matching file paths

Example:

```text
.claude/rules/api.md
    ↓
Only relevant API files
```

### Key distinction

> **Directory-level `CLAUDE.md` = rules tied to a directory.**

> **Path-specific rule = rules tied to matching file/path patterns.**

---

# 10. Why Not Put Everything in `CLAUDE.md`?

A large root `CLAUDE.md` can contain rules that are irrelevant to the current task.

For example:

```text
CLAUDE.md

Terraform rules
React rules
API rules
Testing rules
Database rules
Migration rules
Security rules
Deployment rules
```

Claude may need only one or two of these for a particular task.

Path-specific rules allow specialized instructions to remain targeted.

```text
Root CLAUDE.md
       ↓
General instructions

.claude/rules/api.md
       ↓
API files only

.claude/rules/testing.md
       ↓
Test files only

.claude/rules/terraform.md
       ↓
Terraform files only
```

This helps reduce context noise and keeps instructions modular.

---

# 11. Path-Specific Rules vs Directory-Level `CLAUDE.md`

This is one of the most important comparisons.

### Directory-level `CLAUDE.md`

Use when the convention belongs to a specific directory.

Example:

```text
frontend/
└── CLAUDE.md
```

Rules apply to work within that directory.

### Path-specific rule

Use when the convention applies to matching files regardless of where they are located.

Example:

```text
.claude/rules/tests.md
```

with:

```yaml
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---
```

This can cover tests scattered across:

```text
src/api/
src/services/
src/components/
```

### Exam shortcut

> **Directory = where the code lives.**

> **Path pattern = what kind of files you want to target.**

---

# 12. Conditional Loading

The important idea is that path-specific rules are **conditional**.

Conceptually:

```text
                 Claude works with a file
                           |
                           ↓
                 Does the path match?
                    /                              YES             NO
                   |               |
                   ↓               ↓
             Load the rule     Rule stays inactive
```

This is why path-specific rules are useful for specialized conventions.

---

# 13. Practical Example — Terraform

File:

```text
.claude/rules/terraform.md
```

Content:

```yaml
---
paths:
  - "**/*.tf"
---

# Terraform Conventions

- Use snake_case for resource names.
- Tag every resource with environment and team labels.
- Never hardcode AMI IDs.
- Keep modules consistent with project conventions.
```

Now Terraform conventions can apply to `.tf` files wherever they are located.

---

# 14. Practical Example — Database Migrations

Suppose migrations are scattered across several directories.

Create:

```text
.claude/rules/migrations.md
```

with:

```yaml
---
paths:
  - "**/migrations/**/*.sql"
---

# Migration Rules

- Migrations must be append-only.
- Never modify an already-applied migration.
- Include a rollback strategy where required.
```

The rule becomes relevant when Claude works with matching migration files.

---

# 15. Practical Example — API Files

```text
.claude/rules/api.md
```

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
---

# API Development Rules

- Validate every request.
- Use the standard error response format.
- Return correct HTTP status codes.
- Document public endpoints.
```

This keeps API-specific conventions out of unrelated frontend or documentation work.

---

# 16. Rule Organisation

Rules can be organized into multiple files.

Example:

```text
project/
└── .claude/
    ├── CLAUDE.md
    └── rules/
        ├── coding-style.md
        ├── testing.md
        ├── security.md
        ├── api.md
        └── terraform.md
```

The rules directory can also be organized into subdirectories.

Example:

```text
.claude/
└── rules/
    ├── frontend/
    │   ├── react.md
    │   └── styles.md
    └── backend/
        ├── api.md
        └── database.md
```

The main benefit is modular organisation.

---

# 17. Important Loading Detail

Current Claude Code documentation describes path-scoped rules as loading when Claude works with matching files.

In particular, path-scoped rules trigger when Claude **reads matching files**, rather than on every tool use.

So think:

```text
Matching file is touched/read
        ↓
Path rule becomes relevant
```

rather than:

```text
Every tool call
        ↓
Load every path rule
```

This distinction explains why path-scoped rules help reduce irrelevant context.

---

# 18. Sharing Rules Across Projects

The current Claude Code documentation also supports symlinks in `.claude/rules/`.

This can be useful when organizations maintain common standards.

Conceptually:

```text
Shared rules
     ↓
Symlink
     ↓
Project A/.claude/rules/
Project B/.claude/rules/
```

This allows a common rule set to be reused across projects.

---

# 19. User-Level Path Rules

Personal rules can also exist under:

```text
~/.claude/rules/
```

These can apply across projects on the user's machine.

Example:

```text
~/.claude/rules/
├── preferences.md
└── workflows.md
```

Use this for personal preferences rather than project-specific team conventions.

---

# 20. Exam Decision Framework

When you see a scenario, ask:

### Question 1

Does the rule apply broadly to the whole project?

```text
→ CLAUDE.md
```

### Question 2

Does the rule belong to one particular directory?

```text
→ Directory-level CLAUDE.md
```

### Question 3

Does the same rule need to apply to a file type/path pattern scattered across directories?

```text
→ .claude/rules/<name>.md
   with paths:
```

### Question 4

Is it a task-specific workflow rather than a convention?

```text
→ Skill
```

---

# 21. Exam Traps

## Trap 1 — Putting every specialized convention in root `CLAUDE.md`

This increases irrelevant context.

If a rule is only relevant to specific files, path-scope it.

---

## Trap 2 — Using a directory-level `CLAUDE.md` for files scattered across the repository

A directory-level file is appropriate when the rule belongs to that directory.

For a file type scattered across many directories, use:

```text
.claude/rules/
```

with:

```yaml
paths:
```

---

## Trap 3 — Forgetting the `paths` field

This:

```text
.claude/rules/testing.md
```

without path frontmatter is an unscoped rule.

This:

```yaml
---
paths:
  - "**/*.test.ts"
---
```

makes it conditional.

---

## Trap 4 — Using the wrong glob pattern

For example:

```yaml
paths:
  - "*.test.ts"
```

matches test files in the project root, not every nested directory.

For tests anywhere in the project, use:

```yaml
paths:
  - "**/*.test.ts"
```

---

## Trap 5 — Thinking path-specific rules are Skills

Path-specific rules are **conventions/instructions**.

Skills are **task-specific workflows**.

```text
Path rule
→ "All API files must validate inputs."

Skill
→ "/analyse-api"
```

---

# 22. Practice Scenario

### Scenario

A project has hundreds of test files spread across many directories.

The team wants every test file to follow the same conventions, but they do not want those testing instructions loaded when Claude is working only on application code.

### Best approach

Create:

```text
.claude/rules/testing.md
```

with:

```yaml
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---
```

Then add the testing conventions below the frontmatter.

### Why?

The convention is:

- File-type specific
- Spread across many directories
- Not relevant to unrelated files
- A rule rather than a workflow

Therefore:

```text
.claude/rules/
        +
paths
```

is the appropriate solution.

---

# 23. Quick Comparison Table

| Requirement | Best mechanism |
|---|---|
| Project-wide instruction | `CLAUDE.md` |
| Directory-specific instruction | Directory-level `CLAUDE.md` |
| File-type/path-specific instruction | `.claude/rules/*.md` + `paths` |
| Task-specific workflow | Skill |
| Personal cross-project rule | `~/.claude/rules/` |

---

# 24. CCAF Checklist

- [ ] Know that path-specific rules live in `.claude/rules/`.
- [ ] Know the `paths` YAML frontmatter field.
- [ ] Understand glob patterns.
- [ ] Understand conditional loading.
- [ ] Know rules without `paths` are unscoped.
- [ ] Distinguish path-specific rules from directory-level `CLAUDE.md`.
- [ ] Know when a file type is scattered across directories, path rules are a strong fit.
- [ ] Know path-specific rules reduce irrelevant context.
- [ ] Know Skills are for task-specific workflows.
- [ ] Know personal rules can live under `~/.claude/rules/`.
- [ ] Recognize common glob patterns such as `**/*.ts` and `**/*.test.ts`.
- [ ] Know multiple path patterns can be specified.

---

# One-Line Memory Aids

> **`.claude/rules/` = modular rules.**

> **`paths:` = condition for when the rule applies.**

> **Glob = pattern describing matching files.**

> **No `paths` = unscoped rule.**

> **Directory-level `CLAUDE.md` = directory-based scope.**

> **Path-specific rule = file/path-based scope.**

> **Scattered file type + same convention = `.claude/rules/` with `paths`.**

> **Skill = workflow; Rule = convention.**

> **Path-specific rules help keep irrelevant instructions out of context.**
