# CCAF Domain 3.2 — Custom Slash Commands and Skills

> Study notes based on Claude Certification Guide, Domain 3 Task 3.2.

## 1. Core Concept

Custom commands and Skills are presented as a **unified Skills system**.

Both of these can create the same slash command:

```text
.claude/commands/deploy.md
        ↓
     /deploy
```

and:

```text
.claude/skills/deploy/SKILL.md
        ↓
     /deploy
```

The important difference is their **file structure**.

### Command

A command is a flat Markdown file:

```text
.claude/commands/deploy.md
```

### Skill

A Skill is a directory containing a required `SKILL.md` entry point:

```text
.claude/skills/deploy/SKILL.md
```

### Critical distinction

A flat file placed directly inside `.claude/skills/` does **not** create a command:

```text
.claude/skills/deploy.md   ❌
```

The Skill structure must be:

```text
.claude/skills/deploy/SKILL.md   ✅
```

---

# 2. Canonical Location

The canonical location for Skills is:

```text
.claude/skills/
```

The older/alias location is:

```text
.claude/commands/
```

The commands location continues to work for backward compatibility.

### Why prefer `.claude/skills/`?

The Skills path supports additional capabilities, including:

- Supporting files alongside `SKILL.md`
- Automatic discovery based on intent
- Skill precedence when a Skill and command share a name

Both paths support the same YAML frontmatter options discussed in this topic.

---

# 3. Command vs Skill Structure

| Type | Location | Structure | Example |
|---|---|---|---|
| Command | `.claude/commands/` | Flat `.md` file | `commands/deploy.md` |
| Skill | `.claude/skills/` | Directory + `SKILL.md` | `skills/deploy/SKILL.md` |

Both can expose:

```text
/deploy
```

### Memory aid

> **Command = flat file.**

> **Skill = directory + `SKILL.md`.**

---

# 4. Project-Scoped Commands and Skills

For team-wide workflows, use the project-scoped locations:

```text
.claude/skills/
```

or:

```text
.claude/commands/
```

These live inside the repository and are shared through Git.

Example:

```text
my-project/
└── .claude/
    └── skills/
        └── review/
            └── SKILL.md
```

Every developer who clones or pulls the repository receives the project-scoped workflow.

### Good use cases

- `/review`
- `/deploy-check`
- `/lint`
- `/migration-guide`

### Mental model

```text
.claude/
   ↓
Project scope
   ↓
Shared via Git
   ↓
Team members get the workflow
```

---

# 5. User-Scoped Commands and Skills

Personal workflows belong in:

```text
~/.claude/skills/
```

or:

```text
~/.claude/commands/
```

These are personal and are not version-controlled or shared with teammates.

Example:

```text
~/.claude/
└── skills/
    └── brainstorm/
        └── SKILL.md
```

### Good use cases

Personal productivity workflows that other team members do not need.

### Mental model

```text
~/.claude/
   ↓
User scope
   ↓
Personal
   ↓
Not shared with teammates
```

---

# 6. Project vs User Scoping

This pattern is important throughout Domain 3.

| Requirement | Location | Scope |
|---|---|---|
| Team-wide Skill | `.claude/skills/<name>/SKILL.md` | Project / shared via Git |
| Team-wide command | `.claude/commands/<name>.md` | Project / shared via Git |
| Personal Skill | `~/.claude/skills/<name>/SKILL.md` | User / personal |
| Personal command | `~/.claude/commands/<name>.md` | User / personal |

### Exam memory aid

> **`.claude/` = project and shared.**

> **`~/.claude/` = user and personal.**

---

# 7. Skills Frontmatter

A Skill's `SKILL.md` can contain optional YAML frontmatter at the top.

Example:

```yaml
---
description: "Analyse a feature area of the codebase and report structure, patterns and risks"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
argument-hint: "Provide a feature description or area of the codebase to analyse"
---
```

The frontmatter configures how the Skill behaves.

The guide specifically highlights:

- `context: fork`
- `allowed-tools`
- `argument-hint`

It also explains the role of `description`.

---

# 8. `description`

Example:

```yaml
description: "Analyse a feature area of the codebase and report structure, patterns and risks"
```

The description tells Claude **what the Skill is for**.

It is important for Skill discovery and matching a user's request to the appropriate Skill.

Without a useful description, automatic matching is weakened; the Skill can still be explicitly invoked by its slash command.

### Memory aid

> **`description` = what the Skill does.**

---

# 9. `context: fork`

Example:

```yaml
context: fork
```

This runs the Skill in an **isolated sub-agent context**.

This is especially useful for verbose, exploratory work.

Examples:

- Codebase analysis
- Brainstorming
- Tasks producing extensive file listings
- Tasks producing many code excerpts
- Noisy exploratory analysis

### Without `context: fork`

```text
Main conversation
      ↓
Skill runs
      ↓
Lots of intermediate output
      ↓
Main context becomes crowded
```

### With `context: fork`

```text
Main conversation
      |
      | invoke Skill
      ↓
┌──────────────────┐
│ Isolated Skill   │
│                  │
│ File listings    │
│ Code excerpts    │
│ Exploration      │
│ Analysis         │
└──────────────────┘
      |
      ↓
Concise result
```

### Exam memory aid

> **`context: fork` = isolate verbose Skill output from the main conversation.**

---

# 10. `allowed-tools`

Example:

```yaml
allowed-tools:
  - Read
  - Grep
  - Glob
```

The guide describes `allowed-tools` as pre-approving the listed tools so that the Skill can use them without a permission prompt while the Skill is active.

In the example, the Skill can use:

- `Read`
- `Grep`
- `Glob`

This is appropriate for a read-only analysis workflow.

### Important current behavior

The guide notes an important distinction:

`allowed-tools` does **not** itself remove other tools from Claude's available tool pool.

The actual mechanism for preventing tools from being used is:

```text
disallowed-tools
```

or permission deny rules.

So remember the distinction:

```text
allowed-tools
→ pre-approves listed tools

disallowed-tools / deny rules
→ actual restriction/security boundary
```

For the exam guide's expected framing, `allowed-tools` is the key configuration for a trusted workflow's tool access.

---

# 11. `argument-hint`

Example:

```yaml
argument-hint: "Specify the module path to analyse (e.g., src/api/auth)"
```

This tells the developer what input the Skill expects.

If the Skill is invoked without the required argument, the hint helps prompt the developer for it.

Example:

```text
/analyse-feature
```

The user can then be prompted to provide something such as:

```text
src/api/auth
```

### Memory aid

> **`argument-hint` = tell the developer what input the Skill needs.**

---

# 12. Skills vs `CLAUDE.md`

This distinction is directly tested.

## Skills

Skills are:

- Task-specific
- Workflow-oriented
- Invoked on demand
- Explicitly invokable with `/skill-name`
- Potentially automatically discoverable when the description matches the user's intent

The full Skill body loads when the Skill is invoked.

## `CLAUDE.md`

`CLAUDE.md` is for:

- Always-loaded guidance
- Universal project standards
- Instructions that should apply broadly

### Rule

> **Do not put task-specific procedures in `CLAUDE.md`.**

> **Do not put always-on reference material in Skills.**

---

# 13. Example: What Belongs Where?

### API naming convention

Suppose:

> "All APIs must use kebab-case."

This should apply to every relevant code-generation task.

Use:

```text
CLAUDE.md
```

or an appropriate path-scoped rule.

### Codebase analysis workflow

Suppose:

> "Inspect the authentication module, trace dependencies, identify risks, and produce a report."

This is an occasional multi-step workflow.

Use a:

```text
Skill
```

Example:

```text
/analyse-feature
```

### Mental model

```text
Always-on convention
        ↓
    CLAUDE.md

Specific workflow
        ↓
      Skill
```

---

# 14. Personal Skill Customisation

Suppose the team has:

```text
/analyse
```

but you want a more detailed personal version.

Create a differently named personal Skill:

```text
~/.claude/skills/deep-analyse/SKILL.md
```

Then use:

```text
/deep-analyse
```

This keeps the team workflow unchanged.

### Key idea

> **Personal variants should use different names to avoid affecting or conflicting with the team workflow.**

---

# 15. Example: Team Review + Personal Brainstorm

A team wants:

```text
/review
```

available to everyone.

Create:

```text
.claude/commands/review.md
```

or the canonical:

```text
.claude/skills/review/SKILL.md
```

A developer wants a personal brainstorming workflow.

Create:

```text
~/.claude/skills/brainstorm/SKILL.md
```

with:

```yaml
---
context: fork
---
```

The result:

```text
/review
→ Team-wide
→ Shared through Git

/brainstorm
→ Personal
→ Isolated execution
→ Not shared with teammates
```

---

# 16. Common Exam Traps

## Trap 1 — Flat file directly inside `.claude/skills/`

Wrong:

```text
.claude/skills/review.md
```

A Skill requires:

```text
.claude/skills/review/SKILL.md
```

A flat Markdown command belongs under:

```text
.claude/commands/review.md
```

---

## Trap 2 — Putting team commands in `~/.claude/`

Wrong for team-wide workflows:

```text
~/.claude/commands/review.md
```

That is user-scoped and personal.

For team sharing, use:

```text
.claude/commands/review.md
```

or:

```text
.claude/skills/review/SKILL.md
```

---

## Trap 3 — Treating Skills like `CLAUDE.md`

Skills are not general always-on guidance.

Use:

```text
CLAUDE.md
```

for universal standards.

Use:

```text
Skill
```

for task-specific workflows.

---

## Trap 4 — Forgetting `context: fork`

If a Skill produces very verbose exploratory output, use:

```yaml
context: fork
```

to isolate that output.

---

## Trap 5 — Misunderstanding `allowed-tools`

For the current behavior described by the guide:

```text
allowed-tools
→ pre-approves listed tools

disallowed-tools
→ actually removes/restricts tools
```

Do not confuse pre-approval with the actual security boundary.

---

## Trap 6 — Putting task-specific procedures in `CLAUDE.md`

If a workflow is something developers run occasionally, such as:

```text
analyse codebase
review a feature
brainstorm an architecture
```

a Skill is generally the appropriate mechanism.

---

# 17. Practice Scenario

### Question

A team wants a `/review` command available to everyone who clones the repository.

A developer also wants a personal `/brainstorm` Skill that produces verbose codebase-analysis output without cluttering the main conversation.

### Correct setup

Team command:

```text
.claude/commands/review.md
```

or canonical:

```text
.claude/skills/review/SKILL.md
```

Personal Skill:

```text
~/.claude/skills/brainstorm/SKILL.md
```

with:

```yaml
---
context: fork
---
```

### Why?

- `.claude/` is project-scoped and shared through Git.
- `~/.claude/` is user-scoped and personal.
- `context: fork` isolates verbose Skill output.

---

# 18. Build Pattern

A read-only personal brainstorming Skill can look like:

```text
~/.claude/skills/brainstorm/SKILL.md
```

with:

```yaml
---
description: "Explore a feature or codebase area and generate alternatives"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
argument-hint: "Provide a feature description or codebase area to explore"
---
```

The important concepts are:

```text
~/.claude/
    ↓
Personal scope

skills/brainstorm/SKILL.md
    ↓
Skill structure

context: fork
    ↓
Isolated verbose execution

allowed-tools
    ↓
Pre-approved tools

argument-hint
    ↓
Expected user input
```

---

# 19. Quick Reference

| Requirement | Canonical location | Alternative | Scope |
|---|---|---|---|
| Team-wide Skill | `.claude/skills/<name>/SKILL.md` | `.claude/commands/<name>.md` | Project / shared |
| Team-wide command | `.claude/skills/<name>/SKILL.md` | `.claude/commands/<name>.md` | Project / shared |
| Personal Skill | `~/.claude/skills/<name>/SKILL.md` | `~/.claude/commands/<name>.md` | User / personal |
| Personal command | `~/.claude/commands/<name>.md` | `~/.claude/skills/<name>/SKILL.md` | User / personal |
| Universal standards | `CLAUDE.md` | `.claude/rules/` where appropriate | Always-on |

---

# 20. Exam Checklist

- [ ] Know the unified Skills system concept.
- [ ] Know `.claude/skills/` is canonical.
- [ ] Know `.claude/commands/` is the backward-compatible alias.
- [ ] Know a Skill is a directory containing `SKILL.md`.
- [ ] Know a command is a flat `.md` file.
- [ ] Know both can expose the same `/command`.
- [ ] Know a flat `.md` directly inside `.claude/skills/` is not a Skill.
- [ ] Know `.claude/` is project-scoped and shared through Git.
- [ ] Know `~/.claude/` is user-scoped and personal.
- [ ] Understand `description`.
- [ ] Understand `context: fork`.
- [ ] Understand `allowed-tools`.
- [ ] Understand `disallowed-tools` as the actual restriction mechanism.
- [ ] Understand `argument-hint`.
- [ ] Distinguish Skills from always-loaded `CLAUDE.md`.
- [ ] Know when to use a Skill versus `CLAUDE.md`.
- [ ] Know how to create a personal Skill without affecting the team's workflow.

---

# One-Line Memory Aids

> **Command = flat `.md` file.**

> **Skill = directory + `SKILL.md`.**

> **`.claude/skills/` = canonical Skill location.**

> **`.claude/commands/` = backward-compatible command location.**

> **`.claude/` = project/shared.**

> **`~/.claude/` = personal/user.**

> **`context: fork` = isolate verbose Skill output.**

> **`allowed-tools` = pre-approve listed tools.**

> **`disallowed-tools` = actual tool restriction.**

> **`argument-hint` = tell the developer what input is needed.**

> **`CLAUDE.md` = always-on standards.**

> **Skill = on-demand task-specific workflow.**
