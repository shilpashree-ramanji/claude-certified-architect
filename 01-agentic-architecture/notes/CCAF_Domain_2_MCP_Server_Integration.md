# CCAF Domain 2 — MCP Server Integration

## What You Need to Know

MCP (Model Context Protocol) servers extend Claude's capabilities by connecting agents to external systems such as:

- Databases
- APIs
- Development tools
- Issue trackers
- Internal systems

The key exam areas are:

1. Project-level vs user-level configuration
2. Environment variable expansion
3. MCP resources
4. Community server vs custom server decisions
5. MCP tool descriptions
6. Common configuration and integration traps

---

# 1. MCP Configuration Scoping

The supplied material describes two configuration levels:

```text
Project-level
.mcp.json
    ↓
Shared with team

User-level
~/.claude.json
    ↓
Personal to the user
```

These serve different purposes.

---

# 2. Project-Level — `.mcp.json`

`.mcp.json` lives in the project repository root.

It is:

- Version-controlled
- Shared with the team
- Appropriate for team-wide integrations

Examples:

- Jira
- GitHub
- Internal APIs
- Shared development tools

### Example

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "jira": {
      "command": "npx",
      "args": [
        "-y",
        "@community/mcp-server-jira"
      ],
      "env": {
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

### Use `.mcp.json` when

The entire team needs the same MCP server configuration.

### Memory aid

> **Project = `.mcp.json` = shared configuration.**

---

# 3. User-Level — `~/.claude.json`

`~/.claude.json` lives in the user's home directory.

It is:

- Personal
- Not version-controlled
- Not automatically shared with teammates

### Use it for

- Experimental servers
- Personal integrations
- Servers being tested before proposing them to the team

### Memory aid

> **User = `~/.claude.json` = personal configuration.**

---

# 4. Project vs User Configuration

| Configuration | Location | Shared? | Typical use |
|---|---|---|---|
| Project | `.mcp.json` | Yes | Team-wide servers |
| User | `~/.claude.json` | No | Personal / experimental servers |

### Exam trap

Do not put a team-wide MCP integration only in:

```text
~/.claude.json
```

because teammates will not automatically receive that configuration.

---

# 5. Tool Availability

The supplied material states that tools from configured project-level and user-level servers are discovered at connection time and are available simultaneously when the servers are configured and reachable.

Conceptually:

```text
Project MCP servers
        ↓
      Tools
        ↘
          Claude
        ↗
      Tools
        ↑
User MCP servers
```

There is no separate manual activation step described in the source.

### Important distinction

Configuration scope controls **where configuration lives and who it is shared with**.

It does not mean the model manually activates one server at a time.

---

# 6. Environment Variable Expansion

`.mcp.json` supports:

```text
${VARIABLE_NAME}
```

This allows configuration to be shared without committing credentials.

Example:

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}",
    "DATABASE_URL": "${DATABASE_URL}"
  }
}
```

The configuration contains the **variable names**, not the secret values.

Each developer provides their own credentials locally.

---

# 7. Why Environment Variables Matter

Using environment variable expansion provides several benefits:

- Configuration can safely be committed
- Each developer can authenticate with their own credentials
- Token rotation does not require changing the configuration
- Secrets are not placed directly in repository history

### Bad

```json
{
  "env": {
    "GITHUB_TOKEN": "actual-secret-value"
  }
}
```

### Better

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```

### Memory aid

> **Commit configuration, not credentials.**

---

# 8. MCP Resources

MCP resources expose information to agents without requiring exploratory tool calls.

Examples:

- Issue summaries
- Documentation hierarchies
- Database schemas

### Database example

Without a resource:

```text
list_tables
    ↓
describe_table
    ↓
describe_table
    ↓
...
```

This can consume unnecessary tool calls.

With a schema resource:

```text
Database schema resource
        ↓
Agent immediately understands
tables, columns, relationships
```

### Key distinction

> **Resources show agents what data is available.**

> **Tools let agents act on that data.**

---

# 9. Why MCP Resources Matter

Resources reduce unnecessary discovery work.

For example, a database schema resource can provide:

```text
Tables
Columns
Types
Relationships
```

before the agent starts querying.

This gives the agent useful context upfront.

### Memory aid

> **Resource = information/context. Tool = action.**

---

# 10. Build vs Use an Existing MCP Server

This is a recurring architectural decision.

The supplied material recommends:

> **Evaluate community servers first for standard integrations.**

Examples of standard integrations include:

- Jira
- GitHub
- Slack
- Linear
- Notion

If an existing community server covers the team's requirements, using it avoids unnecessary development and maintenance.

---

# 11. When to Build a Custom MCP Server

Build a custom server when the scenario explicitly requires capabilities that existing community servers cannot provide.

Examples:

### Team-specific workflow

```text
Standard Jira integration
       ↓
Does not support required custom workflow
       ↓
Custom MCP server may be justified
```

### Proprietary internal system

```text
Internal proprietary API
       ↓
No suitable community server
       ↓
Custom MCP server
```

### Custom business logic

If the tool layer needs specialized business logic that existing servers cannot provide, custom development may be appropriate.

### Exam rule

> **Standard integration → evaluate community server first.**

> **Unique internal requirement → custom server may be justified.**

---

# 12. Exam Trap — Custom Server for Standard Integration

Scenario:

> A team needs Jira integration.

Incorrect first response:

```text
Build a custom Jira MCP server
```

Preferred first step:

```text
Evaluate existing community Jira MCP servers
        ↓
Can one satisfy the requirements?
        ↓
YES → Use it
NO  → Consider custom
```

The exam favors the pragmatic, lower-maintenance solution.

---

# 13. Enhancing MCP Tool Descriptions

An MCP tool with a sparse description may lose tool-selection decisions to better-described built-in tools.

Example:

```text
search_codebase:
"Searches code"
```

This gives the model very little information.

A richer description might explain:

- Semantic search
- AST-aware indexing
- What the tool returns
- When it is more appropriate than text search
- What kinds of queries it handles

Example:

```text
search_codebase:
"Performs semantic code search across the repository
using AST-aware indexing. Returns matching functions,
classes, and methods with file paths, line numbers,
and surrounding code.

Use this for finding code by intent or behavior rather
than exact text matches. Prefer this over Grep when
searching for what code does rather than what it contains."
```

### Why this helps

The model now has enough information to understand the MCP tool's unique capability.

---

# 14. MCP Tool vs Built-In Tool

Suppose the system provides:

```text
Built-in:
Grep

MCP:
search_codebase
```

If the MCP description is only:

```text
"Searches code"
```

the model may prefer Grep because it has a clearer understanding of it.

Improve the MCP description:

```text
Semantic search
+
AST-aware
+
Full context
+
Better for intent-based searches
```

Now the model can make a more informed selection.

---

# 15. Configuration Mental Model

```text
                    MCP
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
    Project config          User config
     .mcp.json             ~/.claude.json
          |                     |
      Team tools            Personal tools
          \                     /
           \                   /
            ↓                 ↓
               Agent
                 ↓
              Tool use
```

Credentials should be injected through environment variables rather than committed directly.

---

# 16. Complete Example

A team wants:

```text
GitHub
Jira
Internal API
```

### Project configuration

Use:

```text
.mcp.json
```

because all team members need the integrations.

### Credentials

Use:

```text
${GITHUB_TOKEN}
${JIRA_TOKEN}
${INTERNAL_API_TOKEN}
```

rather than committing actual secrets.

### Personal experiment

A developer wants to test a new MCP server before sharing it.

Use:

```text
~/.claude.json
```

### Standard Jira integration

First:

```text
Evaluate community server
```

Only build custom if the community server cannot satisfy required team-specific behavior.

---

# 17. CCAF Exam Traps

## Trap 1 — Building custom MCP servers for standard integrations

For standard systems such as Jira or GitHub, evaluate existing community servers first.

## Trap 2 — Putting team-wide configuration in `~/.claude.json`

`~/.claude.json` is personal.

Team-wide configuration belongs in:

```text
.mcp.json
```

## Trap 3 — Committing credentials into `.mcp.json`

Never place secret values directly into version-controlled configuration.

Use:

```text
${ENV_VAR}
```

## Trap 4 — Leaving MCP descriptions too sparse

Poor descriptions can cause the model to prefer better-described built-in tools.

Enhance descriptions with capabilities, outputs, and usage boundaries.

## Trap 5 — Confusing resources and tools

```text
Resource → exposes information/context
Tool     → performs an action
```

## Trap 6 — Assuming separate MCP servers automatically solve tool overload

If tools from connected servers are exposed together, moving them between servers does not necessarily reduce the model's effective choice set.

---

# 18. Practice Scenario

### Scenario

A team needs Jira integration for its Claude Code workflow.

A developer proposes building a custom MCP server.

### Options

**A.** Evaluate existing community MCP servers for Jira and only build custom if they cannot handle team-specific workflows.

**B.** Add Jira only to `~/.claude.json`.

**C.** Use Bash and the Jira REST API instead of MCP.

**D.** Immediately build a custom MCP server exposing exactly the required Jira endpoints.

### Correct Answer

**A. Evaluate existing community MCP servers first.**

Why?

Jira is a standard integration.

The pragmatic decision is:

```text
Community server
      ↓
Does it satisfy requirements?
   ↙          ↘
 YES          NO
  ↓            ↓
Use it      Consider custom
```

---

# 19. CCAF Checklist

- [ ] Understand MCP server integration
- [ ] Know `.mcp.json`
- [ ] Know `~/.claude.json`
- [ ] Understand project vs user scope
- [ ] Understand environment variable expansion
- [ ] Know why credentials should not be committed
- [ ] Understand MCP resources
- [ ] Distinguish resources from tools
- [ ] Know when to use community servers
- [ ] Know when custom servers are justified
- [ ] Understand MCP tool-description quality
- [ ] Understand why sparse descriptions can cause tool-selection problems
- [ ] Recognize common MCP configuration exam traps

## One-Line Memory Aids

> **`.mcp.json` = project/team. `~/.claude.json` = personal.**

> **`${ENV_VAR}` = keep secrets out of configuration.**

> **Resource = information. Tool = action.**

> **Standard integration → community server first.**

> **Custom requirement → custom server when existing servers cannot satisfy it.**
