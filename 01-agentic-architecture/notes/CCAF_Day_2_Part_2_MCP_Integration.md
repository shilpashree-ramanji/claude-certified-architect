# CCAF Day 2 — Part 2
# MCP Integration

**Domain:** Tool Design & MCP Integration  
**Part:** 2 — Model Context Protocol (MCP)

---

# 🎯 Part 2 Goal

Understand **MCP**, why it exists, its architecture, and how AI applications can discover and use external capabilities through MCP.

The goal is to understand the architecture first — not memorize implementation details.

---

# 1. What is MCP?

**MCP = Model Context Protocol.**

At a high level, MCP is a standardized protocol for connecting AI applications to external systems and capabilities.

Mental model:

```text
AI Application
      ↓
     MCP
      ↓
External capabilities / context
```

Instead of every AI application creating a completely different integration for every external system, MCP provides a common protocol for these interactions.

---

# 2. Why Do We Need MCP?

Imagine an AI application needs access to:

```text
GitHub
Jira
Database
File System
Internal APIs
Cloud Services
```

Without a common protocol, integrations can become application-specific.

Conceptually:

```text
AI App
 ├── Custom GitHub integration
 ├── Custom Jira integration
 ├── Custom DB integration
 ├── Custom File integration
 └── Custom API integration
```

With MCP, the architecture can use a standardized protocol:

```text
                    AI Application
                          ↓
                     MCP Client
                          ↓
                     MCP Protocol
                          ↓
                    MCP Servers
                  /      |                       ↓       ↓       ↓
             GitHub    Jira     Database
```

### Core idea

> **MCP provides a standardized way for AI applications to connect to external capabilities and context.**

---

# 3. MCP Is a Protocol

This is important.

MCP is not:

```text
❌ An AI model
❌ An agent
❌ A database
❌ A programming language
❌ A replacement for APIs
```

MCP is a **protocol** that defines how an AI application can communicate with MCP servers and interact with capabilities they expose.

Think of it like a common communication contract.

---

# 4. High-Level MCP Architecture

The basic architecture is:

```text
                AI Application
                     │
                     ↓
                 MCP Client
                     │
                     ↓
                MCP Protocol
                     │
                     ↓
                 MCP Server
                     │
                     ↓
              External System
```

This is the most important diagram to understand first.

---

# 5. MCP Host

The **MCP host** is the AI application/environment in which the user interacts with the model.

Conceptually:

```text
Host
 ├── AI Model
 ├── MCP Client(s)
 └── User interaction
```

Examples of host environments can include AI applications or coding environments that support MCP.

### Mental model

> **Host = the AI application that manages the overall interaction.**

---

# 6. MCP Client

The **MCP client** is the component that communicates with an MCP server.

Conceptually:

```text
MCP Host
   │
   └── MCP Client
          │
          ↓
      MCP Server
```

The client handles the MCP communication with the server.

### Important distinction

```text
Host
 ↓
Contains/manages MCP client
 ↓
Client communicates with server
```

Do not think of Host and Client as identical concepts.

---

# 7. MCP Server

An **MCP server** exposes capabilities and context to an MCP client.

For example:

```text
MCP Server
   │
   ├── Tool: search_jira()
   ├── Tool: get_issue()
   ├── Resource: project documentation
   └── Prompt: incident_summary
```

The server may interact with:

```text
Jira
Database
GitHub
File system
Internal API
Cloud service
```

### Mental model

> **MCP Server = provider of capabilities/context to the AI application through MCP.**

---

# 8. Host → Client → Server

Remember this relationship:

```text
                    HOST
                     │
                     ↓
                   CLIENT
                     │
                     ↓
                   SERVER
                     │
                     ↓
              External System
```

A common beginner mistake is:

> "The MCP server is the AI application."

No.

The server generally **provides capabilities**.

The host is the application interacting with the user/model.

---

# 9. MCP Tools

MCP servers can expose **tools**.

For example:

```text
MCP Server
   │
   ├── check_pipeline_status()
   ├── get_pipeline_logs()
   ├── compare_schemas()
   └── query_metadata()
```

The AI application can discover these capabilities and invoke the appropriate tool.

This connects directly to Day 2 Part 1.

```text
Part 1:
What is a tool?
        ↓
Part 2:
How can tools be exposed to AI applications through MCP?
```

---

# 10. MCP Resources

MCP can also expose **resources**.

A resource represents information/context that the AI application can access.

Conceptually:

```text
MCP Server
   │
   ├── Tools
   │    └── Actions
   │
   └── Resources
        └── Context / information
```

For example, a server might expose documentation, files, database information, or other contextual data as resources.

### Important mental model

```text
Tool
→ "Do something"

Resource
→ "Give me information/context"
```

---

# 11. MCP Prompts

MCP also supports **prompts**.

A prompt can represent a reusable prompt template or interaction pattern.

Conceptually:

```text
MCP Server
   │
   └── Prompt
         ↓
   Standardized instruction/template
```

For example:

```text
incident_analysis
```

could provide a reusable structure for analyzing an incident.

### Mental model

```text
Tools
→ Actions

Resources
→ Context

Prompts
→ Reusable interaction templates
```

---

# 12. Tools vs Resources vs Prompts

This distinction is important.

| MCP Capability | Mental model |
|---|---|
| **Tool** | Do something |
| **Resource** | Access information/context |
| **Prompt** | Reusable instruction/template |

Example:

```text
Tool:
restart_pipeline()

Resource:
pipeline://customer_ingestion/run/12345

Prompt:
"Analyze this pipeline failure and summarize the root cause."
```

---

# 13. Tool Discovery

One important benefit of a standardized protocol is that the AI application can discover what capabilities are available.

Conceptually:

```text
AI Application
      ↓
"What capabilities are available?"
      ↓
MCP Server
      ↓
Returns available tools/resources/prompts
      ↓
AI application understands capabilities
      ↓
Agent selects appropriate capability
```

This connects directly to Day 1 autonomy.

---

# 14. MCP and Agent Autonomy

Day 1:

> **Autonomy = freedom to decide how to accomplish a goal within constraints.**

Day 2 Part 2:

MCP can provide the agent/application with access to capabilities.

```text
Goal
 ↓
Agent decides
 ↓
Available MCP capabilities
 ↓
Select relevant tool
 ↓
Execute
 ↓
Observe result
 ↓
Continue
```

MCP provides the **connection mechanism**.

The agent still needs to decide what to do.

---

# 15. MCP Does Not Automatically Make Something an Agent

Important exam trap:

> "If an application uses MCP, it is automatically an AI agent."

**False.**

MCP provides a protocol for connecting AI applications to capabilities/context.

Whether the application behaves as an agent depends on the application's behavior and architecture.

For example:

```text
LLM + MCP Tool
```

does not automatically mean:

```text
Autonomous Agent
```

The system still needs appropriate goal-directed decision-making and execution behavior.

---

# 16. MCP Server and External Systems

An MCP server can act as an integration boundary.

Example:

```text
AI Application
      ↓
MCP Client
      ↓
MCP Server
      ↓
Jira API
```

The AI application does not necessarily need to know all the details of how the Jira API works.

The MCP server can provide a standardized interface.

Example exposed tool:

```text
get_jira_issue(issue_key)
```

Internally:

```text
get_jira_issue()
      ↓
Jira API
      ↓
Response
      ↓
MCP Server
      ↓
MCP Client
      ↓
AI Application
```

---

# 17. MCP Server as an Abstraction Layer

Think of the server as an abstraction layer:

```text
AI Application
       ↓
   MCP interface
       ↓
   MCP Server
       ↓
External API / DB / File System
```

The server can hide implementation details.

For example:

```text
AI sees:
query_customer_orders()

Server internally:
→ authenticate
→ connect to database
→ execute SQL
→ format result
→ return response
```

---

# 18. MCP Security

Connecting an AI application to external systems introduces security considerations.

Important areas include:

- Authentication
- Authorization
- Least privilege
- Sensitive data
- Tool permissions
- Access boundaries
- Destructive operations
- Human approval
- Logging and auditing

Example:

```text
MCP Server
   │
   ├── read_customer_data()
   ├── update_customer_data()
   └── delete_customer_data()
```

These should not necessarily have identical permissions or approval requirements.

---

# 19. Least Privilege with MCP

Connect this to Day 2 Part 1.

The MCP server should expose only the capabilities that are actually required.

Example:

A reporting agent may need:

```text
read_sales_data()
generate_report()
```

It probably does not need:

```text
delete_sales_data()
alter_production_schema()
```

### Principle

> **Expose only the capabilities necessary for the intended task.**

---

# 20. Human-in-the-Loop with MCP

Suppose an MCP server exposes:

```text
delete_production_table()
```

The architecture could enforce:

```text
Agent
 ↓
Requests destructive tool
 ↓
Approval required
 ↓
Human
 ↓
Approve?
 ↙       ↘
YES       NO
 ↓         ↓
Execute   Reject
```

MCP does not eliminate the need for application-level safety controls.

---

# 21. MCP Error Handling

Consider:

```text
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
External API
 ↓
TIMEOUT
```

The system needs appropriate error handling.

Potential strategies:

```text
Retry
 ↓
Alternative tool
 ↓
Escalate
 ↓
Inform user
```

But, as learned in Part 1:

> **Do not blindly retry every failure.**

Consider:

- Error type
- Timeout
- Retry limit
- Side effects
- Idempotency
- Availability
- Cost

---

# 22. MCP Server Failure

What if the MCP server itself is unavailable?

```text
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
❌ Unavailable
```

The application may need to:

- Retry connection
- Use another available capability
- Inform the user
- Gracefully degrade
- Escalate

The exact behavior depends on the architecture.

---

# 23. MCP and Tool Discovery — Example

Suppose your enterprise has:

```text
Pipeline MCP Server
```

It exposes:

```text
check_pipeline_status()
get_pipeline_logs()
compare_schemas()
```

The AI application connects to it.

Conceptually:

```text
AI Application
       ↓
MCP Client
       ↓
Pipeline MCP Server
       ↓
Available capabilities
       ├── check_pipeline_status
       ├── get_pipeline_logs
       └── compare_schemas
```

User asks:

> "Why did today's customer pipeline fail?"

The agent/application can use the available capabilities to investigate.

```text
Goal
 ↓
check_pipeline_status()
 ↓
FAILED
 ↓
get_pipeline_logs()
 ↓
Schema mismatch
 ↓
compare_schemas()
 ↓
Root cause
```

This is where **Day 1 + Day 2 Part 1 + Day 2 Part 2** come together.

---

# 24. MCP Architecture Example

Imagine:

```text
                         USER
                           ↓
                    AI APPLICATION
                           ↓
                      MCP CLIENT
                           ↓
                    ┌──── MCP ────┐
                    ↓             ↓
             Pipeline Server   Jira Server
                    ↓             ↓
              Data Platform      Jira
```

Pipeline Server may expose:

```text
Tools:
- check_pipeline_status()
- get_pipeline_logs()
- compare_schemas()

Resources:
- pipeline documentation
- metadata
```

Jira Server may expose:

```text
Tools:
- search_issues()
- get_issue()
- create_issue()
```

The AI application can use the relevant capability for the current task.

---

# 25. One Server vs Multiple Servers

Suppose an enterprise has:

```text
GitHub
Jira
Data Platform
Slack
```

You could conceptually have:

```text
MCP Server
 ├── GitHub
 ├── Jira
 ├── Data
 └── Slack
```

or:

```text
GitHub MCP Server
Jira MCP Server
Data MCP Server
Slack MCP Server
```

There is no universal rule that one architecture is always correct.

Consider:

- Security boundaries
- Ownership
- Scalability
- Deployment
- Permissions
- Maintainability
- Failure isolation

---

# 26. MCP Architecture Trade-offs

More servers can provide:

- Better isolation
- Clear ownership
- Independent deployment
- Separate permissions

But can also introduce:

- More infrastructure
- More configuration
- More coordination
- More operational overhead

Again:

> **Use the simplest architecture that reliably satisfies the requirements.**

This is the same principle from Day 1.

---

# 27. End-to-End Mental Model

Bring everything together:

```text
                         USER
                           ↓
                      AI HOST
                           ↓
                      MCP CLIENT
                           ↓
                     MCP SERVER
                           ↓
                 Available Capabilities
                    /       |                          ↓        ↓        ↓
                 TOOL    RESOURCE   PROMPT
                   ↓
             External System
                   ↓
                RESULT
                   ↓
              MCP CLIENT
                   ↓
                AI HOST
                   ↓
                  AGENT
                   ↓
            Decide next action
                   ↓
              Continue / Stop
```

---

# 28. Day 1 → Day 2 Connection

### Day 1

We learned:

```text
Goal
 ↓
Agent
 ↓
Planning
 ↓
Autonomy
 ↓
State
 ↓
Handoff
 ↓
Orchestration
```

### Day 2 Part 1

We learned:

```text
Agent
 ↓
Tool
 ↓
Input
 ↓
Execution
 ↓
Output
 ↓
Error / Retry / Safety
```

### Day 2 Part 2

We now add:

```text
AI Application
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tools / Resources / Prompts
 ↓
External Systems
```

Full picture:

```text
                    USER
                      ↓
                 AI APPLICATION
                      ↓
                    AGENT
                      ↓
               Decide / Plan
                      ↓
               Select Capability
                      ↓
                 MCP CLIENT
                      ↓
                 MCP SERVER
                      ↓
              ┌───────┼────────┐
              ↓       ↓        ↓
            TOOL   RESOURCE   PROMPT
              ↓
       EXTERNAL SYSTEM
              ↓
            RESULT
              ↓
             AGENT
              ↓
         Goal complete?
          ↙         ↘
        YES          NO
         ↓            ↓
       STOP        Continue
```

---

# 📌 Key Definitions

| Concept | Simple meaning |
|---|---|
| **MCP** | Model Context Protocol |
| **Protocol** | Standardized rules for communication |
| **MCP Host** | AI application/environment managing the interaction |
| **MCP Client** | Component that communicates with an MCP server |
| **MCP Server** | Provides capabilities/context through MCP |
| **MCP Tool** | Action/capability exposed by a server |
| **MCP Resource** | Context/information exposed through MCP |
| **MCP Prompt** | Reusable prompt/template exposed through MCP |
| **Tool Discovery** | Discovering available capabilities |
| **MCP Integration** | Connecting AI applications with external capabilities using MCP |

---

# 🔥 Architecture Principles

1. **MCP is a protocol, not an agent.**
2. **MCP does not automatically make an application autonomous.**
3. **Host, Client, and Server have different responsibilities.**
4. **MCP servers can expose tools, resources, and prompts.**
5. **Tools perform actions; resources provide context; prompts provide reusable instructions.**
6. **Security and permissions remain essential.**
7. **Use least privilege.**
8. **High-impact operations may require human approval.**
9. **MCP does not remove the need for error handling.**
10. **The simplest reliable architecture is usually preferable to unnecessary complexity.**

---

# 🧠 MCP Quick Memory Trick

Remember:

```text
HOST
  ↓
CLIENT
  ↓
SERVER
  ↓
CAPABILITIES
```

And:

```text
TOOLS     → DO
RESOURCES → KNOW
PROMPTS   → GUIDE
```

---

# 📝 Part 2 Revision Checklist

Before moving to the Day 2 combined scenarios, I should be able to explain:

- [ ] What MCP is
- [ ] Why MCP exists
- [ ] MCP as a protocol
- [ ] MCP Host
- [ ] MCP Client
- [ ] MCP Server
- [ ] Host vs Client vs Server
- [ ] MCP Tools
- [ ] MCP Resources
- [ ] MCP Prompts
- [ ] Tool discovery
- [ ] MCP security
- [ ] Least privilege
- [ ] Human approval
- [ ] MCP error handling
- [ ] MCP server failure
- [ ] Single vs multiple MCP servers
- [ ] MCP architecture trade-offs
- [ ] How MCP connects with Day 1 agent architecture
- [ ] How MCP connects with Day 2 Part 1 tool design

---

# 🔜 Next

```text
Day 2 Part 1
     ↓
Day 2 Part 2
     ↓
Combined Revision
     ↓
Tool + MCP Scenarios
     ↓
Day 2 Assessment
     ↓
Hands-on MCP
```

> **Goal:** Don't just memorize "Host → Client → Server." Be able to explain why each component exists, what it is responsible for, and how tools/resources flow through the architecture.
