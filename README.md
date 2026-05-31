# desmos

**desmos** is an event-sourced coordination and audit framework for autonomous AI software agents.

Agents are probabilistic. They can reason, propose, misunderstand, hallucinate, and recover. **desmos** provides the deterministic enterprise backbone around them: typed commands, immutable events, policy gates, structured work items, formal clarifications, projections, audit trails, and integrations with developer systems such as GitHub, Jira, CI/CD, and agent runtimes.

The goal is not to let agents chat freely. The goal is to make agent collaboration **structured, traceable, replayable, reviewable, and auditable**.

---

## Core Idea

```text
No direct agent-to-agent chat.
No hidden project state.
No undocumented decisions.
No code work without a work item.
No architecture change without traceability.
No security-relevant change without an audit gate.
```

Every relevant interaction becomes a typed command or event.

```text
Command
   policy validation
   immutable event journal
   outbox publication
   projections
   context packs
   tickets, reviews, audits, dashboards
```

`desmos` treats Jira, GitHub, dashboards, and agent inboxes as **projections**.
The immutable event journal is the source of truth.

---

## Architectural Principles

### Storage First

State-changing actions are written to the journal before they are published, projected, or synchronized.

```text
Validate command
   append event
   write outbox entry
   publish event
   update projections
```

This keeps the system replayable, debuggable, and auditable.

### Event Sourcing

The current project state is derived from an append-only event stream.

Examples:

```text
work-item.created
work-item.assigned
clarification.requested
clarification.answered
architecture-decision.approved
audit.finding.created
change-request.raised
approval.granted
approval.rejected
```

### CQRS

Writes happen through commands.
Reads happen through projections.

```text
Commands mutate state by producing events.
Queries read projections built from events.
```

### Transactional Outbox

Events are persisted together with outbox records. The event bus publishes only after the write transaction succeeds.

This avoids dual-write problems between the journal and the message broker.

### Formal Agent Communication

Agents do not exchange informal messages. They communicate through structured artifacts:

```text
WorkItem
Clarification
ChangeRequest
Approval
AuditFinding
ContextPack
ArchitectureDecision
```

### Human-in-the-Loop by Design

Humans are not an afterthought. Human approvals, overrides, rejections, and clarifications are first-class events.

---

## Module Overview

```text
desmos/
 koine        // shared language: commands, events, schemas, ACL, context packs
 pylai        // gateway: REST, MCP, A2A/agent protocol adapters
 nomos        // policy engine: authorization, validation, gates
 aletheia     // event journal: append-only source of truth
 emporion     // outbox, topics, broker integration
 eidos        // projections, read models, context-pack assembly
 graphe       // work items, tickets, change requests, approvals
 anakrisis    // clarifications, follow-up questions, evidence requests
 boule        // planning, coordination, scout work breakdown
 dokimasia    // audit, review gates, security checks
 proxenia     // external adapters: GitHub, Jira, CI/CD, agent runtimes
 agora        // human workspace and UI
```

... (full README content continues) ...
