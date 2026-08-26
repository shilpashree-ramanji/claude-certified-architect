# CCAF Domain 2 — Tool Distribution & Tool Choice

## What You Need to Know

The number of tools given to an agent directly affects how reliably it selects the correct tool.

This is an **architectural decision**, not merely an implementation detail.

### Core principle

> **Give each agent only the tools required for its defined role.**

The supplied study material identifies an optimal range of approximately **4–5 tools per agent**.

Too many tools increase decision complexity and can reduce selection reliability.

---

# 1. The Tool Overload Problem

Giving one agent a large toolkit can degrade tool-selection reliability.

Example:

```text
18 tools
   ↓
More choices
   ↓
Higher decision complexity
   ↓
Less reliable selection
```

The recommended range in this material is:

> **4–5 tools per agent**

But quantity is not the only factor.

**Relevance matters too.**

A synthesis agent should not automatically have web-search tools.

A web-search agent should not automatically have document-analysis tools.

---

# 2. Role-Specific Tool Distribution

Each agent should receive tools appropriate to its role.

### Example

| Agent | Example tools |
|---|---|
| Web Search | `search_web`, `fetch_page`, `extract_links`, `save_snippet` |
| Document Analysis | `extract_metadata`, `extract_data_points`, `summarize_content`, `verify_claim` |
| Synthesis | `compile_report`, scoped `verify_fact`, `format_citation`, `assess_coverage` |
| Coordinator | `Agent`, `review_output`, `request_revision` |

The goal is:

```text
Agent role
    ↓
Required capabilities
    ↓
Only relevant tools
```

---

# 3. Why Irrelevant Tools Are Dangerous

Suppose a synthesis agent receives:

```text
compile_report
verify_fact
format_citation
web_search
```

If `web_search` is unnecessary for its role, the agent may perform its own searches instead of using the results already provided.

This can cause:

- Duplicated work
- Wasted context
- Additional latency
- Unnecessary tool calls

### Memory aid

> **Right role + right tools = better tool selection.**

---

# 4. Choosing the Correct Tool-Reduction Strategy

There are several different problems, and each has a different solution.

| Situation | Appropriate fix |
|---|---|
| Few tools, but two descriptions look alike | Sharpen descriptions |
| Different jobs such as query, transform, export | Split tools by role |
| Many tools are variations of the same operation | Consolidate into one parameterized tool |
| Tool has capabilities beyond the agent's needs | Constrain the tool |

This distinction is important for the exam.

---

# 5. Description Problem vs Tool Overload

Suppose an agent has:

```text
5 tools
```

and repeatedly confuses:

```text
get_customer
lookup_order
```

This is primarily a **description/interface problem**.

Improve the descriptions.

Now suppose an agent has:

```text
22 tools
```

Even excellent descriptions may not completely solve the decision-complexity problem.

### Key exam principle

> **Count the tools before choosing the remedy.**

The same symptom — incorrect tool selection — can have different causes.

---

# 6. Consolidating Near-Duplicate Tools

Sometimes many tools perform variations of the same kind of operation.

Example:

```text
pivot_table
calculate_percentile
normalise_currency
...
```

If a transformation agent receives 19 such tools, splitting them by role does not solve the problem.

Instead, consolidate them into a parameterized tool.

### Example

```json
{
  "name": "transform_data",
  "description": "Apply a transformation to a dataset. Use transform_type to select the operation.",
  "input_schema": {
    "type": "object",
    "properties": {
      "dataset": {
        "type": "string"
      },
      "transform_type": {
        "type": "string",
        "enum": [
          "pivot",
          "percentile",
          "normalise_currency"
        ]
      },
      "options": {
        "type": "object"
      }
    },
    "required": [
      "dataset",
      "transform_type"
    ]
  }
}
```

Instead of:

```text
19 separate transformation tools
```

the model chooses:

```text
transform_data
      ↓
transform_type
      ↓
pivot / percentile / normalise_currency / ...
```

### Why this helps

The number of top-level tool choices decreases.

> **The capability is preserved; the selection problem becomes smaller.**

---

# 7. Consolidation vs Constraining

These are different architectural ideas.

## Consolidation

Reduce the number of tools by combining similar operations.

```text
19 transformation tools
        ↓
1 transform_data tool
```

## Constraining

Reduce what a tool is capable of doing.

Example:

```text
Generic:
fetch_url
```

becomes:

```text
Constrained:
load_document
```

The constrained tool accepts only appropriate document resources.

### Important

> **Consolidation reduces the number of choices.**

> **Constraining reduces the capabilities exposed to the agent.**

Both can be useful in the same system.

---

# 8. Least Privilege in Tool Design

Instead of giving an agent:

```text
fetch_url
```

which can potentially retrieve arbitrary URLs, give it:

```text
load_document
```

which is restricted to the resources the agent actually needs.

Benefits:

- Prevents misuse
- Makes the purpose clearer
- Reduces unintended side effects
- Limits the agent's capabilities

This is **least privilege applied to tool design**.

### Memory aid

> **Give the agent exactly what it needs — not everything the system can do.**

---

# 9. Server Boundaries Do Not Automatically Solve Tool Overload

A common incorrect assumption is:

> "If we move 22 tools across two MCP servers, the model now has fewer choices."

According to the supplied material, this does **not** solve the problem if the client exposes all those tools together.

Conceptually:

```text
MCP Server A → 11 tools
MCP Server B → 11 tools
                  ↓
             Client/model
                  ↓
             22 available tools
```

The selection problem is still effectively 22 tools.

### Exam trap

> **Moving tools between MCP servers is not, by itself, a solution to model-level tool overload.**

---

# 10. The `tool_choice` Configuration

The `tool_choice` parameter controls how the model interacts with available tools.

The supplied material describes three important modes:

1. `"auto"`
2. `"any"`
3. Forced selection of a specific tool

---

# 11. `tool_choice: "auto"`

```json
{
  "tool_choice": {
    "type": "auto"
  }
}
```

### Meaning

The model decides:

- Whether to call a tool
- Which tool to call

It can also return normal text when no tool call is appropriate.

### Use when

The model needs flexibility.

### Memory aid

> **Auto = model decides whether and what to call.**

---

# 12. `tool_choice: "any"`

```json
{
  "tool_choice": {
    "type": "any"
  }
}
```

### Meaning

The model **must call a tool**, but it chooses which available tool to call.

It cannot simply respond with plain text.

### Useful scenario

Suppose you have several extraction schemas:

```text
invoice schema
receipt schema
contract schema
```

The document type is unknown.

`"any"` ensures the model selects one of the available tools and produces structured output.

### Memory aid

> **Any = must call a tool, model chooses which.**

---

# 13. Forced Tool Selection

A specific tool can be forced:

```json
{
  "tool_choice": {
    "type": "tool",
    "name": "extract_metadata"
  }
}
```

### Meaning

The model must call the named tool.

This is useful when a workflow has a mandatory first step.

Example:

```text
extract_metadata
      ↓
enrichment
      ↓
final processing
```

If metadata extraction must happen first, force:

```text
extract_metadata
```

The model cannot skip directly to enrichment.

After the mandatory operation completes, later turns can return to:

```text
tool_choice: auto
```

### Memory aid

> **Forced = specific tool must run.**

---

# 14. `auto` vs `any` vs Forced

| Mode | Can the model answer without a tool? | Who chooses the tool? |
|---|---:|---|
| `auto` | Yes | Model, if it decides a tool is needed |
| `any` | No | Model |
| Forced tool | No | System/application specifies the tool |

### Exam shortcut

```text
Flexible operation
→ auto

Must call some tool
→ any

Must call THIS tool
→ forced selection
```

---

# 15. Scoped Cross-Role Tools

Sometimes an agent needs occasional access to a capability normally associated with another role.

A naive solution is:

```text
Synthesis Agent
      ↓
Coordinator
      ↓
Search Agent
      ↓
Coordinator
      ↓
Synthesis Agent
```

This creates extra round trips.

The supplied material recommends a **scoped cross-role tool** for frequent simple operations.

Example:

```text
Synthesis Agent
      ↓
scoped verify_fact
```

The synthesis agent can handle simple verification directly.

Complex verification still follows the full workflow:

```text
Simple verification
→ local scoped tool

Complex verification
→ coordinator
→ specialist
```

---

# 16. The 85% / 15% Pattern

The study material gives this conceptual example:

```text
85% → simple lookups
15% → complex verification
```

Instead of routing all 100% through the coordinator:

```text
Simple 85%
   ↓
Scoped verify_fact
```

and:

```text
Complex 15%
   ↓
Coordinator
   ↓
Specialist
```

This reduces unnecessary round trips for common simple operations.

### Key principle

> **Keep the common simple path local; route exceptional complex work through orchestration.**

---

# 17. Scoped Tools and Least Privilege

A scoped cross-role tool should be deliberately limited.

It should not turn the agent into a full version of another specialist.

Example:

```text
Synthesis Agent

Allowed:
verify_fact(simple lookup)

Not allowed:
full web research
multi-source investigation
complex browsing
```

This preserves role boundaries while reducing unnecessary orchestration.

---

# 18. Complete Architecture Example

```text
                    Coordinator
                   /     |      \
                  /      |       \
                 ↓       ↓        ↓
          Web Search   Document   Synthesis
            Agent      Agent       Agent
              |          |           |
          4 tools     4 tools      4 tools
                                   |
                              verify_fact
                                (scoped)
```

The synthesis agent does not receive the complete web-search toolkit.

It receives only the small capability it frequently needs.

---

# 19. CCAF Decision Framework

When a question says tool selection is unreliable, ask:

### Step 1 — How many tools?

```text
Small/manageable
→ investigate descriptions
```

```text
Very large
→ investigate tool overload
```

### Step 2 — Are the tools doing different jobs?

If yes:

```text
Split by role
```

### Step 3 — Are many tools variations of the same operation?

If yes:

```text
Consolidate into a parameterized tool
```

### Step 4 — Does the agent have more capability than it needs?

If yes:

```text
Constrain the tool
```

### Step 5 — Does the workflow require a specific tool?

If yes:

```text
Forced tool choice
```

### Step 6 — Must some tool always be called, but any one is acceptable?

If yes:

```text
tool_choice = any
```

### Step 7 — Does the model need normal flexibility?

If yes:

```text
tool_choice = auto
```

---

# 20. Exam Traps

### Trap 1 — Giving one agent every available tool

More tools increase decision complexity.

Use role-specific tool scoping.

### Trap 2 — Giving a synthesis agent full web-search capabilities

This can duplicate work and waste context.

Use a scoped verification capability when appropriate.

### Trap 3 — Splitting 19 near-identical transformation tools across agents

If they share the same operation shape, consolidation may be better.

### Trap 4 — Moving tools between MCP servers

Server boundaries do not automatically reduce the model's available tool choices when all tools are exposed together.

### Trap 5 — Confusing `auto` and `any`

```text
auto → tool call optional
any  → tool call mandatory
```

### Trap 6 — Confusing `any` and forced selection

```text
any
→ some tool must be called
→ model chooses

forced
→ specific tool must be called
```

### Trap 7 — Routing every simple cross-role request through the coordinator

For frequent simple operations, a scoped cross-role tool can avoid unnecessary round trips.

---

# 21. CCAF Checklist

- [ ] Understand tool overload
- [ ] Know the recommended 4–5 tool range from the supplied material
- [ ] Understand role-specific tool distribution
- [ ] Understand tool relevance
- [ ] Distinguish description problems from tool-count problems
- [ ] Understand consolidation of near-duplicate tools
- [ ] Understand parameterized tools
- [ ] Understand constrained tools
- [ ] Understand least privilege
- [ ] Know that MCP server boundaries do not automatically solve model-level overload
- [ ] Understand `tool_choice: auto`
- [ ] Understand `tool_choice: any`
- [ ] Understand forced tool selection
- [ ] Understand scoped cross-role tools
- [ ] Understand local handling of frequent simple operations
- [ ] Recognize common exam traps

## One-Line Memory Aids

> **4–5 tools per agent → role-scoped selection.**

> **Many similar tools → consolidate.**

> **Too much capability → constrain.**

> **Simple cross-role task → scoped local tool.**

> **`auto` = optional tool call. `any` = some tool required. Forced = specific tool required.**
