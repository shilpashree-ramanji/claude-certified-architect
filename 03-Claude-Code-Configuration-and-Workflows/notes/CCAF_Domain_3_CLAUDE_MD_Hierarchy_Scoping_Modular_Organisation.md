# CCAF — CLAUDE.md Hierarchy, Scoping, and Modular Organisation

## What You Need to Know

`CLAUDE.md` is the rulebook for Claude Code. It contains instructions that guide Claude while it works on a project.

The key ideas for this topic are:

- **Hierarchy** — instructions can exist at different levels.
- **Scoping** — instructions apply to the relevant scope.
- **Modular organisation** — project-specific instructions can be organised so that rules are kept close to the code they govern.

---

## 1. What Is `CLAUDE.md`?

Think of `CLAUDE.md` as a set of instructions for Claude Code.

Examples:

```md
# Project Rules

- This project uses TypeScript.
- Use npm for package management.
- Run tests after significant changes.
- Do not modify generated files.
```

Typical instructions include:

- Coding conventions
- Testing requirements
- Project-specific practices
- Things Claude should avoid
- Workflow guidance

### Mental Model

> **`CLAUDE.md` = the project's rulebook for Claude Code.**

---

# 2. The Three Levels

The study material uses three important levels:

1. **User level**
2. **Project level**
3. **Directory level**

The simple mental model is:

```text
User
  ↓
Project
  ↓
Directory
```

Think:

> **Broad rules → more specific rules**

---

## 3. User-Level `CLAUDE.md`

The user-level file is:

```text
~/.claude/CLAUDE.md
```

This is for general personal instructions.

Example:

```md
# My General Rules

- Explain changes before making them.
- Prefer simple solutions.
- Always mention tests that should be run.
```

Think of this as:

> **My general preferences when Claude works for me.**

---

# 4. Project-Level `CLAUDE.md`

The project-level file is:

```text
./CLAUDE.md
```

It lives in the project and contains rules that apply to the project as a whole.

Example:

```md
# Project Rules

- This project uses TypeScript.
- Use npm for package management.
- Run `npm test` after significant changes.
- Do not modify generated files.
```

Think:

> **Rules for everyone working on this project.**

---

# 5. Directory-Level `CLAUDE.md`

A directory can have its own `CLAUDE.md`.

Example:

```text
my-project/
│
├── CLAUDE.md
│
├── frontend/
│   ├── CLAUDE.md
│   ├── App.tsx
│   └── Button.tsx
│
└── backend/
    ├── CLAUDE.md
    ├── server.ts
    └── database.ts
```

The frontend can have frontend-specific instructions:

```md
# Frontend Rules

- Use React components.
- Use TypeScript for new files.
- Run frontend tests after frontend changes.
```

The backend can have backend-specific instructions:

```md
# Backend Rules

- Use Node.js APIs.
- Keep database access inside the database layer.
- Run backend integration tests after backend changes.
```

Think:

> **Rules for this particular part of the project.**

---

# 6. Why Hierarchy Matters

Imagine the project has:

```text
frontend/
backend/
```

The project-level `CLAUDE.md` might say:

```md
- Use TypeScript.
- Run tests after changes.
```

The frontend-level `CLAUDE.md` might say:

```md
- Use React components.
- Run frontend tests.
```

The backend-level `CLAUDE.md` might say:

```md
- Keep database access inside the database layer.
- Run backend integration tests.
```

Claude working on frontend code needs the general project guidance plus the relevant frontend-specific guidance.

Claude working on backend code needs the general project guidance plus the relevant backend-specific guidance.

---

# 7. Scoping

**Scoping** means deciding:

> **Which instructions apply to which work?**

For example:

```text
Project
│
├── General project rules
│
├── Frontend
│   └── Frontend-specific rules
│
└── Backend
    └── Backend-specific rules
```

The project rules are broad.

The frontend and backend rules are narrower and more specific.

### Memory Aid

> **Scope = where the rule applies.**

---

# 8. Modular Organisation

Modular organisation means keeping instructions organised according to the part of the project they govern.

Instead of putting every possible rule into one huge root file:

```text
CLAUDE.md
├── frontend rules
├── backend rules
├── database rules
├── testing rules
└── deployment rules
```

you can organise instructions closer to the relevant part of the codebase.

Example:

```text
my-project/
│
├── CLAUDE.md
│
├── frontend/
│   ├── CLAUDE.md
│   └── ...
│
└── backend/
    ├── CLAUDE.md
    └── ...
```

This makes the configuration easier to understand and keeps specialised instructions close to the code they govern.

---

# 9. Simple Example

Suppose you are working on:

```text
frontend/Button.tsx
```

You can think about the instruction scopes like this:

```text
~/.claude/CLAUDE.md
        ↓
General personal instructions

./CLAUDE.md
        ↓
Project-wide instructions

frontend/CLAUDE.md
        ↓
Frontend-specific instructions
```

For backend work:

```text
~/.claude/CLAUDE.md
        ↓
General personal instructions

./CLAUDE.md
        ↓
Project-wide instructions

backend/CLAUDE.md
        ↓
Backend-specific instructions
```

---

# 10. Exam Mental Model

When you see a question about:

### General personal instructions

Think:

```text
~/.claude/CLAUDE.md
```

### Rules for the entire project

Think:

```text
./CLAUDE.md
```

### Rules specific to a part of the project

Think:

```text
./subdirectory/CLAUDE.md
```

---

# 11. Important Distinction: Path-Specific Rules

Do not confuse directory-level `CLAUDE.md` with path-specific rules.

Path-specific rules are useful when the same convention should apply to a particular file type or path pattern even when matching files are scattered across different directories.

Those rules live in:

```text
.claude/rules/
```

and use a `paths` field.

Example:

```yaml
---
paths: ["terraform/**/*"]
---

# Terraform Conventions

- Use snake_case for resource names.
- Tag every resource with environment and team labels.
- Never hardcode AMI IDs.
```

### Mental Model

```text
CLAUDE.md
→ instructions based on configuration scope

.claude/rules/*.md + paths
→ instructions conditionally loaded based on matching file paths
```

---

# 12. When to Use Each

| Situation | Use |
|---|---|
| Personal/general instructions | `~/.claude/CLAUDE.md` |
| Rules for the whole project | `./CLAUDE.md` |
| Rules for a specific directory | `./subdirectory/CLAUDE.md` |
| Same convention for files matching a path pattern across directories | `.claude/rules/*.md` with `paths` |

---

# 13. Common Exam Traps

## Trap 1 — Putting project-wide rules only in the user-level file

User-level configuration is for personal/general instructions.

Project-wide conventions belong at project scope.

---

## Trap 2 — Putting every rule in the root `CLAUDE.md`

This can make the root instruction set unnecessarily broad.

Specialised conventions can be organised closer to the code they govern.

---

## Trap 3 — Confusing directory scope with path-specific rules

A directory-level `CLAUDE.md` is tied to a directory's scope.

A path-specific rule uses glob patterns to conditionally apply a convention when matching files are edited.

---

## Trap 4 — Using a directory-level `CLAUDE.md` for a file type scattered throughout the repository

If Terraform files exist across many directories and the same Terraform rules should apply to all of them, path-specific rules are a better fit:

```text
.claude/rules/terraform.md
```

with:

```yaml
paths: ["terraform/**/*"]
```

---

# 14. Quick Revision

Remember the three levels:

```text
~/.claude/CLAUDE.md
        ↓
       User
        ↓
 ./CLAUDE.md
        ↓
     Project
        ↓
./subdirectory/CLAUDE.md
        ↓
    Directory
```

### One-line memory aids

> **User level = my general instructions.**

> **Project level = project-wide instructions.**

> **Directory level = instructions for a specific part of the project.**

> **Scoping = deciding where instructions apply.**

> **Modular organisation = keeping specialised rules organised near the code they govern.**

> **Path-specific rules = conditional rules based on file/path matching.**

---

# 15. CCAF Checklist

- [ ] Know what `CLAUDE.md` is.
- [ ] Know the user-level path: `~/.claude/CLAUDE.md`.
- [ ] Know the project-level path: `./CLAUDE.md`.
- [ ] Know directory-level `CLAUDE.md`.
- [ ] Understand broad versus specific scope.
- [ ] Understand why modular organisation is useful.
- [ ] Distinguish directory-level rules from path-specific rules.
- [ ] Know that `.claude/rules/` with `paths` supports conditional convention loading.
