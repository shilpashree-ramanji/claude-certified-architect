# Context Window Management

## Domain 4 — Prompt Engineering & Structured Output

### What You Need to Know

A model's **context window** is its working space for the information available during a task.

As a conversation or agent workflow grows, the context can become crowded with:

- Previous conversation turns
- Tool results
- File contents
- Intermediate reasoning
- Research findings
- Repeated information
- Outdated information

The goal of context management is:

> **Keep the context focused on the information that is relevant, current, and important to the task.**

Good context management improves reliability, reduces unnecessary token usage, and helps prevent the model from losing important details.

---

## Context Compression Through Summarization

When a long conversation grows too large, older information can be compressed into a summary.

Conceptually:

```text
Large conversation
       ↓
   Summarization
       ↓
Compact useful context
       ↓
Continue task
```

The summary should preserve important:

- Decisions
- Findings
- Requirements
- Constraints
- References
- Current task state

The objective is **compression without losing information that is necessary for future work**.

---

## Structured Facts Blocks

Some information is too important to leave entirely to normal summarization.

Examples include:

- Customer IDs
- Dates
- Amounts
- Statuses
- Configuration values
- Important decisions
- Explicit requirements

A structured facts block can preserve these values in a stable format.

Example:

```text
FACTS
customer_id: C12345
approved_amount: 500
deadline: 2026-09-01
status: approved
```

The key idea is:

> **Ordinary conversation can be summarized, while critical facts are explicitly preserved.**

This reduces the risk of important numbers, dates, or statuses being altered or dropped during compression.

---

## Lost in the Middle

A long context can produce a specific observable problem:

> The model may pay more attention to information near the beginning or end while important information in the middle becomes harder to use reliably.

This is commonly called the **lost-in-the-middle effect**.

### Example

Suppose an agent discovers:

```text
Beginning:
Architecture uses a master orchestrator.

Middle:
OrderRepository implements BaseRepository
and adds custom caching in findById().

End:
Tests are located under the integration directory.
```

Later, the model might say:

```text
"This follows a typical repository pattern."
```

instead of recalling:

```text
"OrderRepository at src/repos/order.ts implements
BaseRepository and adds custom caching in findById()."
```

That shift from **specific discovered facts to generic patterns** is a sign of context degradation.

---

## Fixing Lost-in-the-Middle Problems

Important findings should be surfaced explicitly rather than relying on their position in a long transcript.

A useful report structure is:

```text
KEY FINDINGS
1. Master orchestrator calls child notebooks.
2. Child notebooks use shared common modules.
3. OrderRepository adds custom caching in findById().

DETAILED EVIDENCE
...
```

Putting the important findings in a compact, clearly labelled section makes them easier to preserve and consume downstream.

---

## Codebase Exploration and Context Management

Codebase exploration means progressively investigating a repository to understand:

- Architecture
- Modules
- Classes
- Functions
- Dependencies
- Data flows
- Conventions
- Patterns

The important principle is:

> **Explore incrementally rather than reading the entire repository upfront.**

### Poor approach

```text
Read 500 files
↓
Load everything into context
↓
Try to understand the repository
```

This wastes context on files that may have nothing to do with the task.

### Better approach

```text
Search
  ↓
Identify relevant files
  ↓
Read relevant files
  ↓
Trace dependencies
  ↓
Search again when needed
  ↓
Build understanding incrementally
```

---

## Codebase Exploration Example

Suppose a new developer joins a project containing:

```text
Ingestion Orchestrator
Validation Orchestrator
File Validation Orchestrator
Data Quality Orchestrator
```

The agent can discover a recurring pattern:

```text
Master orchestrator
        ↓
Child notebook / component
        ↓
Common library / shared module
        ↓
Task execution
```

The useful knowledge is not merely a list of every filename.

The agent can capture the architectural pattern and preserve important references where exact details matter.

### Important distinction

Context management should **not** mean throwing away all specific information.

If an exact path, class, method, or dependency is required for the next task, preserve it.

The goal is to remove unnecessary detail, not useful evidence.

---

## Scratchpad Files

A scratchpad file is a working area for intermediate findings during a complex task.

For example:

```text
research_notes.md
```

might contain:

```markdown
# Research Findings

## Architecture
- Master orchestrator invokes child notebooks.

## Validation
- Validation runs after ingestion.

## Shared modules
- Child workflows rely on common libraries.

## Open Questions
- Confirm whether file validation has a separate retry path.
```

The benefit is that important intermediate knowledge can exist **outside the immediate conversational history**.

This can help a long-running workflow avoid putting every intermediate detail into the main context.

### Important distinction

A scratchpad is a **working artifact**.

It should not automatically be treated as a permanent source of truth. Important finalized information should be moved into the appropriate durable project documentation or state representation when required.

---

## Sub-Agent Delegation and Context Isolation

A coordinator can divide a large task among specialized sub-agents.

Example:

```text
                 Coordinator
                 /    |                    /     |            Architecture  Data   Security
          Agent      Agent    Agent
```

Each sub-agent can work within its own focused context.

Instead of passing every detail between agents, the coordinator can pass only the information needed by the next stage.

This reduces unnecessary context growth.

---

## Summary Injection Between Phases

A multi-phase workflow can use summary injection to transfer important findings from one phase to another.

Example:

```text
Phase 1
Architecture exploration
        ↓
Structured summary
        ↓
Phase 2
Component investigation
```

Phase 1 might produce:

```text
PHASE 1 SUMMARY

Architecture:
- Master orchestrator calls child workflows.
- Child workflows use common libraries.

Important references:
- src/orchestration/master.ts
- src/common/

Open question:
- Determine how validation errors are propagated.
```

Phase 2 receives this summary as part of its initial context.

It does **not** need the entire Phase 1 conversation.

---

## Why Summary Injection Helps

Without summary injection:

```text
Phase 1 raw conversation
        ↓
Phase 2 receives everything
        ↓
Large context
```

With summary injection:

```text
Phase 1
  ↓
Key findings + references + open questions
  ↓
Phase 2
```

Benefits include:

- Smaller context
- Clearer handoff
- Less repeated exploration
- Lower risk of irrelevant information dominating the task
- Better continuity between phases

---

## Upstream / Downstream Context Optimization

In a multi-agent workflow, an upstream agent may produce much more information than the downstream agent needs.

### Poor design

```text
Upstream agent
      ↓
20-page raw analysis
      ↓
Downstream agent
```

### Better design

```text
Upstream agent
      ↓
Structured result
      ↓
Downstream agent
```

For example:

```json
{
  "key_findings": [
    "Finding A",
    "Finding B"
  ],
  "citations": [
    "Source A",
    "Source B"
  ],
  "evidence_score": 0.91,
  "coverage": "partial"
}
```

This is a form of **context distillation**.

The downstream agent gets the useful information without carrying the entire upstream working context.

---

## Tool Result Distillation

Tools can return much more information than the next step requires.

Suppose a database tool returns 100 fields.

The downstream agent needs only:

```text
customer_id
status
amount
risk_score
```

Distill the result:

```json
{
  "customer_id": "C123",
  "status": "active",
  "amount": 500,
  "risk_score": 0.12
}
```

This reduces context usage and gives downstream processing a clearer interface.

---

## `/compact`

The `/compact` command is used in Claude Code to compact the current conversation context.

Conceptually:

```text
Large conversation
       ↓
 /compact
       ↓
Condensed context
       ↓
Continue working
```

The important distinction is:

> **Compaction is not the same thing as a manually maintained structured facts block.**

A compacted conversation is a condensed representation of the previous context.

A structured facts block is an intentionally designed representation of critical facts that should be preserved explicitly.

For important information, do not rely solely on a generic summary if exact values or state must remain stable.

---

## Prompt Caching

Prompt caching is different from summarization.

### Summarization

```text
Large context
↓
Compressed representation
```

### Prompt caching

```text
Unchanged prompt content
↓
Reuse efficiently across requests
```

Prompt caching helps reduce repeated processing of unchanged content.

It does **not** solve context degradation by itself.

### Easy memory trick

> **Summarization changes the amount of context.**
>
> **Caching reuses context that has not changed.**

---

## Crash Recovery via Structured State Manifest

Long-running workflows can periodically save their important state in a structured manifest.

Example:

```json
{
  "phase": "component_analysis",
  "completed_steps": [
    "architecture_mapping",
    "dependency_trace"
  ],
  "key_findings": [
    "Master orchestrator calls ingestion workflow",
    "Shared libraries handle common operations"
  ],
  "open_questions": [
    "Trace validation error handling"
  ],
  "last_successful_checkpoint": "component-042"
}
```

If the workflow crashes, it can recover from the saved state rather than reconstructing progress from a huge conversation.

Think of it as a **checkpoint or save point**.

---

## Why Structured State Helps Recovery

Without a manifest:

```text
Crash
 ↓
Try to reconstruct progress
 ↓
Re-read large conversation
 ↓
Potentially lose details
```

With a manifest:

```text
Crash
 ↓
Load state manifest
 ↓
Know current phase
 ↓
Know completed work
 ↓
Know key findings
 ↓
Resume
```

This improves reliability and reduces unnecessary reprocessing.

---

## Context Degradation

Context degradation is not simply:

> “The context window is full.”

It is a **quality problem** that can appear before the hard token limit is reached.

Observable symptoms include:

- Generic descriptions replacing exact findings
- Forgetting previously discovered dependencies
- Repeating exploration
- Confusing old and new information
- Losing track of requirements
- Ignoring earlier decisions
- Falling back to typical repository patterns instead of observed facts

### Example

Earlier:

```text
OrderRepository at src/repos/order.ts
implements BaseRepository
and adds caching in findById().
```

Later:

```text
This appears to follow a typical repository pattern.
```

The second response has lost important specificity.

---

## Causes of Context Degradation

Common causes include:

### 1. Excessive context

Too many files, tool results, or conversation turns.

### 2. Irrelevant information

Large amounts of information unrelated to the current task.

### 3. Repetition

The same information appearing multiple times.

### 4. Stale information

Files or requirements changing after the agent originally inspected them.

### 5. Poor handoffs

Passing raw outputs between phases or agents instead of distilled summaries.

### 6. Lost-in-the-middle effects

Important findings becoming buried inside long context.

---

## Fixes for Context Degradation

Use a combination of:

### Incremental exploration

Read only what is relevant.

### Summarization

Compress older conversation.

### Structured facts

Protect critical values and decisions.

### Scratchpad / working files

Store intermediate findings externally when appropriate.

### Summary injection

Pass concise phase summaries to later phases.

### Tool result distillation

Send downstream agents only the fields they need.

### Fresh context

When context becomes stale or overloaded, start a focused phase with a concise state summary and re-read current files.

### Structured state manifests

Save checkpoints for long-running or crash-prone workflows.

---

## Context Management Strategy

A strong workflow can look like this:

```text
                 Start
                   ↓
          Define current goal
                   ↓
         Explore incrementally
                   ↓
       Capture important findings
                   ↓
       Distill tool/agent results
                   ↓
        Is context getting noisy?
              /                       No             Yes
            ↓               ↓
       Continue        Summarize /
                       compact / refresh
                              ↓
                     Continue with focused
                          context
```

---

## What to Preserve

When compressing context, prioritize:

1. Current objective
2. Important constraints
3. Key findings
4. Decisions already made
5. Exact references needed for the next step
6. Important identifiers
7. Open questions
8. Validation or error state
9. Completed work
10. Next actions

Do not preserve every intermediate thought simply because it existed.

---

## What to Remove or Compress

Good candidates include:

- Repeated explanations
- Irrelevant tool output
- Exploratory dead ends
- Duplicate findings
- Old intermediate reasoning
- Files discovered but unrelated to the current task
- Repeated copies of the same evidence

The goal is:

> **Maximum useful information per token.**

---

## Exam Traps

### Trap 1 — Read the entire repository first

Wrong.

Explore incrementally to avoid unnecessary context consumption.

### Trap 2 — Context degradation means only token exhaustion

Wrong.

Degradation is also about declining **specificity, accuracy, and focus** even before the hard context limit.

### Trap 3 — Summarize everything indiscriminately

Wrong.

Critical facts such as dates, IDs, amounts, and statuses should be explicitly preserved.

### Trap 4 — Prompt caching solves context degradation

Wrong.

Caching improves reuse of unchanged content. It does not automatically fix an overloaded or degraded context.

### Trap 5 — Pass all raw subagent output downstream

Wrong.

Distill upstream findings into structured information relevant to the downstream task.

### Trap 6 — Use generic summaries when exact references are required

Wrong.

If a future step needs a specific class, method, dependency, or file path, preserve that exact reference.

### Trap 7 — Treat a scratchpad as permanent truth

Wrong.

A scratchpad is a working artifact. Finalized knowledge should be placed in an appropriate durable source when needed.

### Trap 8 — Retry or re-read without checking for stale information

Wrong.

If files changed, refresh the relevant files rather than relying on old context.

---

## Practice Scenario 1

An agent has investigated 40 files. Earlier it identified:

```text
OrderRepository
src/repos/order.ts
findById()
custom caching
```

Later, when asked about the repository architecture, it responds:

```text
"The project follows a typical repository pattern."
```

What does this most strongly indicate?

**A.** Prompt caching failure

**B.** Context degradation

**C.** Successful context compression

**D.** Tool failure

### Correct Answer

**B**

The model has shifted from a specific observed fact to a generic pattern, which is an observable sign of context degradation.

---

## Practice Scenario 2

A Phase 1 architecture agent completes its investigation. Phase 2 needs only:

- Architecture findings
- Important file references
- Open questions

What should Phase 1 provide?

**A.** Its complete conversation history

**B.** Every tool result

**C.** A concise structured summary containing those required items

**D.** Nothing; Phase 2 should restart the exploration

### Correct Answer

**C**

This is summary injection between phases.

---

## Practice Scenario 3

A tool returns 10,000 tokens of data, but the downstream agent needs only five fields.

What should happen?

**A.** Pass the entire result downstream

**B.** Use tool result distillation

**C.** Delete the result completely

**D.** Start a new conversation

### Correct Answer

**B**

Distill the tool output into the fields required by the downstream consumer.

---

## Practice Scenario 4

A long-running agent crashes after completing several phases. A structured state manifest records the current phase, completed steps, key findings, and open questions.

What is the primary benefit?

**A.** It increases the model's context window.

**B.** It allows the workflow to resume from a known structured checkpoint.

**C.** It guarantees the model will never make an error.

**D.** It eliminates the need for validation.

### Correct Answer

**B**

The manifest acts as a structured recovery checkpoint.

---

## Practice Scenario 5

A developer asks an agent to analyze a repository. The agent immediately reads hundreds of files before determining which ones are relevant.

What is the main problem?

**A.** The agent is using too many sub-agents.

**B.** The agent is performing inefficient, non-incremental exploration and consuming unnecessary context.

**C.** The agent should use prompt caching.

**D.** The agent should never read files.

### Correct Answer

**B**

Start with targeted discovery and expand only as needed.

---

## Build Pattern

A practical context-management workflow:

### Step 1 — Define the task

Know exactly what information is required.

### Step 2 — Explore incrementally

Search for relevant entry points instead of reading everything.

### Step 3 — Capture important findings

Preserve exact references where they matter.

### Step 4 — Distill large results

Reduce verbose tool and subagent outputs to useful structured fields.

### Step 5 — Protect critical facts

Use structured facts for values that must remain stable.

### Step 6 — Inject summaries between phases

Give the next phase only the context it needs.

### Step 7 — Watch for degradation

Look for generic reasoning replacing specific discovered facts.

### Step 8 — Refresh when necessary

Summarize, compact, or start a focused context and re-read changed files.

### Step 9 — Checkpoint long workflows

Use a structured state manifest when crash recovery or resumability matters.

---

## Exam Memory Aid

Remember:

> **Explore narrowly.**
>
> **Read progressively.**
>
> **Distill aggressively.**
>
> **Protect critical facts.**
>
> **Summarize between phases.**
>
> **Refresh stale context.**
>
> **Checkpoint long workflows.**

### The Key Distinctions

| Concept | Purpose |
|---|---|
| Summarization | Compress conversation/context |
| Structured facts block | Preserve critical facts explicitly |
| Scratchpad | Store intermediate working findings |
| Summary injection | Transfer concise findings between phases |
| Tool result distillation | Reduce verbose tool output |
| Prompt caching | Reuse unchanged prompt content |
| State manifest | Save structured progress for recovery |
| Incremental exploration | Avoid loading unnecessary code |
| Context degradation | Loss of specificity/focus as context becomes noisy |

### Final Takeaway

**Context management is not just about avoiding token limits.**

It is about preserving the **right information** while removing or compressing the information that no longer helps.

A reliable agent should be able to say:

```text
What am I doing?
What have I learned?
What evidence supports it?
What exact references matter?
What remains unresolved?
What should happen next?
```

That is the heart of effective context-window management.
