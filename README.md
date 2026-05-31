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
````

Every relevant interaction becomes a typed command or event.

```text
Command
  → policy validation
  → immutable event journal
  → outbox publication
  → projections
  → context packs
  → tickets, reviews, audits, dashboards
```

`desmos` treats Jira, GitHub, dashboards, and agent inboxes as **projections**.
The immutable event journal is the source of truth.

---

## Architectural Principles

### Storage First

State-changing actions are written to the journal before they are published, projected, or synchronized.

```text
Validate command
  → append event
  → write outbox entry
  → publish event
  → update projections
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
├─ koine        // shared language: commands, events, schemas, ACL, context packs
├─ pylai        // gateway: REST, MCP, A2A/agent protocol adapters
├─ nomos        // policy engine: authorization, validation, gates
├─ aletheia     // event journal: append-only source of truth
├─ emporion     // outbox, topics, broker integration
├─ eidos        // projections, read models, context-pack assembly
├─ graphe       // work items, tickets, change requests, approvals
├─ anakrisis    // clarifications, follow-up questions, evidence requests
├─ boule        // planning, coordination, scout work breakdown
├─ dokimasia    // audit, review gates, security checks
├─ proxenia     // external adapters: GitHub, Jira, CI/CD, agent runtimes
└─ agora        // human workspace and UI
```

Additional application facades may live above the core modules:

```text
apps/
├─ archon       // admin backend: agents, roles, capabilities, policies
├─ panoptes     // observability backend: monitoring, audit views, risk overview
├─ strategion   // admin/control UI
└─ skopia       // monitoring/audit UI
```

---

## Module Responsibilities

### `koine` — The Shared Language

`koine` defines the common language used by all agents and system modules.

Responsibilities:

* Command contracts
* Event contracts
* CloudEvent envelopes
* Agent communication semantics
* Context-pack schemas
* IDs and correlation identifiers
* Impact classifications
* Evidence references
* Status values
* Validation schemas

`koine` must stay free of business logic.

It defines what can be said, not what should happen.

---

### `pylai` — The Gateway

`pylai` is the controlled entry point into the system.

Responsibilities:

* REST command endpoints
* REST query endpoints
* MCP tool/resource exposure
* Optional A2A/agent-protocol adapters
* Agent inbox access
* Context-pack retrieval
* Authentication handoff
* Request normalization

Examples:

```http
POST /commands/work-items
POST /commands/clarifications
POST /commands/change-requests
POST /commands/approvals

GET /queries/work-items/{id}
GET /queries/context-packs/{id}
GET /queries/agent-inbox/{agentId}
GET /queries/traceability/{workItemId}
```

`pylai` does not decide whether an action is allowed.
It delegates policy decisions to `nomos`.

---

### `nomos` — The Policy Engine

`nomos` is the law of the system.

Responsibilities:

* Authorization
* Role and capability checks
* Command validation
* Status transition rules
* Gate determination
* Security impact handling
* Supply-chain impact handling
* Required review detection
* Policy versioning

Examples:

```text
A coder agent may answer implementation details.
A coder agent may not approve architecture decisions.
A dependency change requires an audit ticket.
A medium/high security impact requires dokimasia review.
A work item cannot be implemented before it is approved.
```

`nomos` decides whether a command is admissible.

---

### `aletheia` — The Event Journal

`aletheia` is the truth.

Responsibilities:

* Append-only event storage
* Event stream sequencing
* Optimistic concurrency
* Idempotency
* Correlation and causation tracking
* Event metadata
* Replay support
* Audit-grade persistence

`aletheia` does not build views.
It stores facts.

---

### `emporion` — The Event Exchange

`emporion` is the event distribution layer.

Responsibilities:

* Transactional outbox
* Event publication
* Broker integration
* Topic routing
* Retry handling
* Dead-letter handling
* Delivery status
* Event fan-out

Potential brokers:

* Kafka
* NATS
* RabbitMQ
* Redpanda
* In-memory broker for MVP/testing

`emporion` does not make business decisions.
It moves persisted events reliably.

---

### `eidos` — The Projection Layer

`eidos` forms readable views from the event journal.

Responsibilities:

* Read models
* Ticket boards
* Agent inboxes
* Traceability matrices
* Audit status views
* Risk registers
* Work item details
* Context-pack assembly
* LLM-optimized Markdown views

`eidos` turns event history into usable form.

Examples:

```text
Event Journal → WorkItemProjection
Event Journal → AgentInboxProjection
Event Journal → TraceabilityProjection
Event Journal → ContextPack
Event Journal → AuditDashboardProjection
```

`eidos` never invents facts.
It derives views from events.

---

### `graphe` — Formal Work

`graphe` owns formal work artifacts.

Responsibilities:

* Work items
* Subtasks
* Tickets
* Change requests
* Approval requests
* Implementation tasks
* Work package lifecycle
* Parent/child work structures
* Work item blocking/unblocking

Typical events:

```text
work-item.created
work-item.split
work-item.assigned
work-item.blocked
work-item.unblocked
change-request.raised
approval.requested
approval.granted
approval.rejected
```

`graphe` is the domain of structured work.

---

### `anakrisis` — Formal Clarification

`anakrisis` owns structured questions, clarifications, and evidence gathering.

Responsibilities:

* Clarification requests
* Clarification answers
* Follow-up questions
* Evidence requests
* Evidence submissions
* Escalation
* Clarification closure
* Blocking/unblocking related work items

Typical events:

```text
clarification.requested
clarification.assigned
clarification.context-prepared
clarification.answered
clarification.escalated
clarification.closed
evidence.requested
evidence.provided
```

A clarification is not chat.
It is a formal, event-sourced process.

---

### `boule` — Coordination and Planning

`boule` represents the operational council.

It coordinates work, but does not make final product or security decisions.

Responsibilities:

* Scout-style coordination
* Work breakdown
* Module mapping
* Task slicing
* Agent assignment proposals
* Conflict detection
* Planning events
* Context selection
* Progress synthesis
* Operational dependency tracking

Typical role:

```text
Read project map
Detect affected modules
Create work package proposal
Request architecture review
Assign implementation tasks after approval
```

`boule` coordinates.
It does not replace the architect, security reviewer, or human product owner.

---

### `dokimasia` — Audit and Review

`dokimasia` owns formal checks.

Responsibilities:

* Security audit
* Architecture review gates
* Supply-chain review
* Dependency review
* CI/CD review
* Policy compliance checks
* Approval/rejection findings
* Risk classification
* Merge blockers

Typical events:

```text
audit.requested
audit.finding.created
audit.approved
audit.rejected
security-review.requested
security-review.completed
dependency-review.failed
architecture-review.approved
```

`dokimasia` is focused on verification, not implementation.

---

### `proxenia` — External Mediation

`proxenia` integrates external systems.

Responsibilities:

* GitHub adapter
* Jira adapter
* CI/CD adapter
* Git provider integration
* Issue synchronization
* Pull request synchronization
* Status check synchronization
* External agent runtime adapters
* Webhook ingestion

Examples:

```text
desmos WorkItem → GitHub Issue
desmos AuditFinding → GitHub PR comment
desmos ChangeRequest → Jira issue
GitHub PR opened → desmos pull-request.created event
CI failed → desmos verification.failed event
```

External systems are not the source of truth.
They are projections or input channels.

---

### `agora` — Human Workspace

`agora` is the human-facing workspace.

Responsibilities:

* Dashboard UI
* Work item views
* Clarification views
* Audit views
* Approval screens
* Manual overrides
* Agent activity visualization
* Risk register UI
* Traceability UI

`agora` talks through `pylai`.
It does not bypass the core.

---

## Application Facades

### `archon`

`archon` is the administrative facade.

Responsibilities:

* Agent registry UI/backend
* Role assignment workflows
* Capability management
* Policy administration
* Manual overrides
* Lifecycle controls

Important boundary:

```text
archon may request policy changes.
nomos enforces policy.
aletheia records policy events.
```

`archon` must not become a god service.

---

### `panoptes`

`panoptes` is the observability facade.

Responsibilities:

* Monitoring aggregation
* Audit dashboards
* System health views
* Risk overview
* Agent activity timeline
* Replay inspection
* Operational reports

Important boundary:

```text
panoptes observes projections.
eidos builds projections.
aletheia stores truth.
```

---

### `strategion`

`strategion` is the control UI.

Responsibilities:

* Admin cockpit
* Agent lifecycle controls
* Manual intervention
* Gate management
* Policy change workflows

---

### `skopia`

`skopia` is the monitoring UI.

Responsibilities:

* Live observation
* Audit trails
* Risk register
* Agent flow visualization
* Event stream inspection

---

## Core Write Flow

```text
Agent / Human / External System
  → pylai
  → nomos
  → domain module
  → aletheia
  → emporion
  → topics
  → eidos / proxenia / boule / dokimasia
```

Example:

```text
POST /commands/work-items
  → nomos validates capability
  → graphe creates WorkItemCreated
  → aletheia appends event
  → emporion publishes event
  → eidos updates board
  → proxenia creates GitHub/Jira projection
```

---

## Core Read Flow

```text
Agent / Human / External System
  → pylai
  → eidos
  → projection / context pack / inbox / dashboard
```

Example:

```text
GET /queries/context-packs/clarifications/CLAR-42
  → eidos assembles LLM-optimized context
  → pylai returns context pack
```

Reads do not come directly from `aletheia`.

---

## Clarification Workflow

Clarifications are formal, asynchronous, and auditable.

### 1. Agent Requests Clarification

```http
POST /commands/clarifications
```

```json
{
  "workItemId": "WP-042",
  "requestedBy": "agent:coder-qwen",
  "question": "Which date format should the import endpoint accept?",
  "reason": "The acceptance criteria do not define date serialization.",
  "requiredAnswerType": "decision",
  "urgency": "normal",
  "impact": {
    "architecture": "low",
    "security": "low"
  }
}
```

### 2. Policy Validation

`nomos` checks:

```text
Is the agent allowed to ask this question?
Is the work item active?
Does the question need escalation?
Does the related work item become blocked?
```

### 3. Event Persistence

`anakrisis` produces:

```text
clarification.requested
```

`aletheia` appends it.

### 4. Event Dispatch

`emporion` publishes the event to:

```text
desmos.anakrisis.clarifications
```

### 5. Projection Update

`eidos` updates:

```text
Agent inbox
Work item blocked state
Clarification detail view
Context-pack availability
Traceability links
```

### 6. Target Agent Retrieves Context

```http
GET /queries/context-packs/clarifications/CLAR-042
```

The returned `ContextPack` contains:

```text
Question
Reason
Related work item
Relevant decisions
Affected files
Policies
Evidence
Required output format
Token budget
```

### 7. Clarification Answer

```http
POST /commands/clarifications/CLAR-042/answer
```

```json
{
  "answeredBy": "agent:scout",
  "answer": "Use ISO 8601 date-time format with UTC normalization.",
  "decision": "accepted",
  "affectedAcceptanceCriteria": [
    "Imported date fields must use ISO 8601."
  ],
  "followUpRequired": false
}
```

### 8. Work Continues

`anakrisis` produces:

```text
clarification.answered
```

`eidos` unblocks the work item if no other blockers remain.

---

## Context Packs

A `ContextPack` is the common denominator for effective LLM communication.

It is not just JSON.
It is a structured envelope plus an LLM-optimized body.

### ContextPack Goals

* Reduce token waste
* Provide the exact relevant context
* Preserve evidence
* Define allowed scope
* Define required output
* Prevent free-form drift
* Make agent responses comparable
* Make decisions auditable

### Example Shape

```yaml
contextPackId: CP-000142
purpose: answer-clarification
audience: scout
tokenBudget: 12000

correlation:
  workItemId: WP-042
  clarificationId: CLAR-009
  causationEventId: EVT-0008112

question:
  askedBy: agent:coder-qwen
  text: Which date format should the import endpoint accept?
  reason: Acceptance criteria do not define date serialization.

workItem:
  title: Implement Customer Import Endpoint
  goal: Import customer data through the public REST API.
  nonGoals:
    - Do not change database schema.
    - Do not introduce Java features beyond Java 8.

relevantDecisions:
  - id: ADR-004
    summary: External APIs use ISO 8601 for temporal values.

affectedFiles:
  - customer-api/src/main/java/.../CustomerImportController.java
  - customer-application/src/main/java/.../ImportCustomerUseCase.java

policies:
  javaVersion: 8
  securityImpact: low
  architectureImpact: medium

requiredOutput:
  type: ClarificationAnswer
  fields:
    - answer
    - decision
    - affectedAcceptanceCriteria
    - followUpRequired
```

A rendered Markdown body may be included for LLMs:

```markdown
# Task

Answer the open clarification.

# Question

Which date format should the import endpoint accept?

# Relevant Context

...

# Output Contract

Return only a ClarificationAnswer.
Do not redesign the endpoint.
```

---

## Communication Semantics

`desmos` may support an Agent Communication Language inspired by performatives such as:

```text
request
propose
inform
approve
reject
delegate
escalate
clarify
```

These performatives describe the intent of a message.
CloudEvents describe the event envelope.
Context Packs describe the useful working context.

```text
Performative = intent
CloudEvent   = transport and persistence envelope
ContextPack  = LLM-optimized working context
```

---

## Agent Capability Model

Agents do not receive broad system access.

They receive explicit capabilities.

Examples:

```text
read_context_pack
request_clarification
answer_clarification
propose_work_breakdown
create_implementation_result
request_audit
commit_code
open_pull_request
approve_architecture
approve_security
```

Capability exposure may be implemented through MCP tools and resources.

Important rule:

```text
nomos grants capability.
pylai exposes the allowed interface.
aletheia records usage.
```

---

## Suggested Topics

```text
desmos.graphe.work-items
desmos.graphe.change-requests
desmos.anakrisis.clarifications
desmos.anakrisis.evidence
desmos.boule.planning
desmos.dokimasia.audit
desmos.proxenia.github
desmos.proxenia.jira
desmos.system.policy
desmos.system.agent-lifecycle
```

---

## Suggested Commands

```text
CreateWorkItem
SplitWorkItem
AssignWorkItem
BlockWorkItem
RaiseChangeRequest
RequestApproval
GrantApproval
RejectApproval

RequestClarification
AnswerClarification
EscalateClarification
RequestEvidence
ProvideEvidence

RequestAudit
CreateAuditFinding
ApproveAudit
RejectAudit

RegisterAgent
AssignAgentRole
GrantCapability
RevokeCapability
```

---

## Suggested Events

```text
work-item.created
work-item.split
work-item.assigned
work-item.blocked
work-item.unblocked
work-item.completed

change-request.raised
change-request.approved
change-request.rejected

approval.requested
approval.granted
approval.rejected

clarification.requested
clarification.assigned
clarification.answered
clarification.escalated
clarification.closed

evidence.requested
evidence.provided

audit.requested
audit.finding.created
audit.approved
audit.rejected

agent.registered
agent.role-assigned
agent.capability-granted
agent.capability-revoked

projection.updated
external.github.issue-created
external.github.pull-request-opened
external.ci.check-failed
```

---

## Repository Structure

Initial structure:

```text
desmos/
├─ README.md
├─ docs/
│  ├─ architecture/
│  ├─ adr/
│  ├─ commands/
│  ├─ events/
│  ├─ context-packs/
│  └─ workflows/
├─ modules/
│  ├─ koine/
│  ├─ pylai/
│  ├─ nomos/
│  ├─ aletheia/
│  ├─ emporion/
│  ├─ eidos/
│  ├─ graphe/
│  ├─ anakrisis/
│  ├─ boule/
│  ├─ dokimasia/
│  └─ proxenia/
└─ apps/
   ├─ archon/
   ├─ panoptes/
   ├─ strategion/
   └─ skopia/
```

For the first MVP, the core modules are:

```text
koine
pylai
nomos
aletheia
emporion
eidos
graphe
anakrisis
proxenia
```

`boule`, `dokimasia`, and `agora` can be introduced once coordination, audit workflows, and UI become first-class.

---

## Module Dependency Rules

```text
koine may be used by all modules.
aletheia stores events but knows no domain decisions.
emporion publishes persisted events but does not create facts.
eidos reads events and builds projections.
pylai exposes commands and queries.
nomos validates commands and gates.
graphe owns work artifacts.
anakrisis owns clarification flows.
boule coordinates planning through commands.
dokimasia reviews through commands.
proxenia synchronizes external systems.
agora reads and writes only through pylai.
```

Forbidden shortcuts:

```text
No module writes directly to projections as truth.
No UI reads the event store directly.
No agent writes to the database.
No adapter becomes the source of truth.
No policy decision is hidden in an app facade.
No direct agent-to-agent state mutation.
```

---

## Naming Summary

```text
desmos     = the binding framework
koine      = shared language
pylai      = controlled gates
nomos      = law and policy
aletheia   = truth
emporion   = exchange place
eidos      = formed views
graphe     = formal work records
anakrisis  = formal inquiry and clarification
boule      = coordination council
dokimasia  = examination and audit
proxenia   = external mediation
agora      = human workspace
archon     = administrator facade
panoptes   = observer facade
strategion = control UI
skopia     = monitoring UI
```

---

## Project Status

`desmos` is currently in concept and architecture definition.

Planned first implementation steps:

1. Define `koine` command and event contracts.
2. Define the initial CloudEvent envelope.
3. Define the first `ContextPack` schema.
4. Implement `aletheia` as an append-only PostgreSQL event journal.
5. Implement transactional outbox support in `emporion`.
6. Implement basic projections in `eidos`.
7. Implement work items in `graphe`.
8. Implement clarifications in `anakrisis`.
9. Expose command/query endpoints through `pylai`.
10. Add a minimal GitHub/Jira projection adapter in `proxenia`.

---

## Vision

`desmos` is an enterprise backbone for autonomous software agents.

It combines:

* event sourcing
* CQRS
* transactional outbox
* typed agent communication
* structured work items
* formal clarifications
* context packs
* policy gates
* audit trails
* external ticket/code-system projections

The result is a system where autonomous agents can collaborate without turning software delivery into an untraceable prompt chain.

**Agents may be probabilistic. The process must not be.**

```
