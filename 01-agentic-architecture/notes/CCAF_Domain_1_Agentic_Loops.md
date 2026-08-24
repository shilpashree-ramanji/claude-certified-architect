# CCAF Domain 1 — Agentic Loops

**Domain:** Agentic Architecture & Orchestration  
**Domain Weight:** 27%

## 1. What Is an Agentic Loop?

An agentic loop is the cycle an AI agent follows to accomplish a task:

> **Goal → Decide → Act → Observe → Decide again → Finish**

```text
User gives a goal
      ↓
Send request to Claude
      ↓
Claude decides what to do
      ↓
Inspect stop_reason
      ↓
 ┌───────────────┐
 ↓               ↓
tool_use      end_turn
 ↓               ↓
Execute        STOP
tool(s)
 ↓
Get tool result
 ↓
Add result to conversation
 ↓
Send conversation back to Claude
 ↓
Claude decides again
 ↓
Repeat if more work is needed
```

## 2. The Five-Step Lifecycle

### Step 1 — User Gives a Goal
Example: “Find the status of order 123.”

### Step 2 — Claude Decides What to Do
If a tool is needed:

`stop_reason = "tool_use"`

The application executes the requested tool.

### Step 3 — Application Executes the Tool

```text
lookup_order(order_id="123")
```

Example result:

```text
status = SHIPPED
delivery_date = Friday
```

### Step 4 — Tool Result Goes Back to Claude
The application adds the tool result to the conversation and sends the updated conversation back to Claude.

Claude needs the new observation to decide what to do next.

### Step 5 — Claude Decides Whether the Task Is Complete
If Claude has enough information:

`stop_reason = "end_turn"`

The application stops the loop and returns the final response.

## 3. `stop_reason`

For the CCAF agentic-loop objective, focus on these two values:

| `stop_reason` | Meaning | Application action |
|---|---|---|
| `tool_use` | Claude wants the application to execute tool(s) | Execute tools, append results, continue |
| `end_turn` | Claude has finished its turn | Stop the loop |

**Memory trick:**  
`tool_use` = **CONTINUE**  
`end_turn` = **STOP**

## 4. Model-Driven Decision Making

Model-driven decision-making means Claude determines what action to take next based on the user's goal, conversation context, available tools, and previous tool results.

```text
Goal
 ↓
Claude decides
 ↓
Tool
 ↓
Observe result
 ↓
Claude decides again
 ↓
Next tool or finish
```

### Pre-configured vs model-driven

**Pre-configured:**

```text
1. lookup_order()
2. get_payment_status()
3. get_shipping_status()
4. answer
```

The developer determines the sequence.

**Model-driven:**

```text
Goal → Claude decides → Tool → Observe → Claude decides again
```

The model adapts based on intermediate results.

## 5. Anti-Pattern 1 — Parsing Natural-Language Signals

Wrong approach:

```python
if "I'm done" in response.text:
    stop_loop()
```

Or looking for words such as:

- “done”
- “complete”
- “finished”
- “no further action”

### Why it is wrong
Natural language is not a reliable program-control signal. Claude may use completion-like language while still needing another tool.

**Correct:** use `stop_reason`.

## 6. Anti-Pattern 2 — Using Assistant Text as a Completion Indicator

Wrong assumption:

> “Claude produced text, therefore the task is complete.”

An assistant response can contain text while also requesting tools.

```text
Claude response
├── Assistant text
├── Tool request(s)
└── stop_reason
```

**Key distinction:**

> **Text is content. `stop_reason` is the loop-control signal.**

## 7. Anti-Pattern 3 — Arbitrary Iteration Caps as the Primary Stop Mechanism

Wrong approach:

```python
for i in range(5):
    run_agent()
```

or:

`MAX_ITERATIONS = 10`

Different tasks require different numbers of actions, so an arbitrary limit can stop the agent before the goal is achieved.

### Important nuance
An iteration limit can still be a **safety guardrail**.

Good:

```text
Primary termination: stop_reason = end_turn
Safety boundary: maximum allowed iterations
```

Bad:

```text
Primary termination: stop after N iterations
```

## 8. Three Anti-Patterns — Quick Comparison

| Anti-pattern | Wrong assumption |
|---|---|
| Natural-language signals | “Claude said done, so stop.” |
| Assistant text as completion indicator | “Claude produced text, so it must be finished.” |
| Arbitrary iteration cap | “After N iterations, the task must be finished.” |

**Correct principle:** Use `stop_reason` as the primary loop-control mechanism.

## 9. Complete Example

```text
User
 ↓
Claude
 ↓
stop_reason = tool_use
 ↓
lookup_order(123)
 ↓
Tool result: Payment failed
 ↓
Add result to conversation
 ↓
Claude again
 ↓
stop_reason = tool_use
 ↓
get_payment_status()
 ↓
Tool result: Payment declined
 ↓
Add result to conversation
 ↓
Claude again
 ↓
stop_reason = end_turn
 ↓
Final answer
```

Core pattern:

> **Reason → Act → Observe → Reason again → Finish**

## 10. CCAF Exam Checklist

- [ ] What an agentic loop is
- [ ] Why the loop repeats
- [ ] `stop_reason`
- [ ] `tool_use`
- [ ] `end_turn`
- [ ] Tool execution
- [ ] Returning tool results to the conversation
- [ ] Model-driven decision-making
- [ ] Model-driven vs pre-configured workflows
- [ ] Natural-language termination anti-pattern
- [ ] Assistant-text termination anti-pattern
- [ ] Arbitrary iteration-cap anti-pattern
- [ ] Iteration cap as a safety guardrail
- [ ] Application control of the loop
- [ ] Claude's role in choosing the next action

## One-Line Memory Aid

> **Goal → Decide → Tool → Observe → Decide again → `end_turn` → Finish**
