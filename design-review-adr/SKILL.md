---
name: design-review-adr
description: Use this to review, write, or strengthen a design document, RFC, Architecture Decision Record, or technical proposal — either as the author improving the document or as a reviewer evaluating someone else's. Applies the Elephant-Goldfish pattern directly: a design artifact is only trustworthy once a fresh, context-free reader can understand it, critique it, and determine whether it is implementation-ready. Trigger whenever the user asks for help reviewing, writing, challenging, or improving a design doc, RFC, ADR, architecture proposal, or says things like "review this design", "help me write an ADR", "is this implementation plan complete", or "what is missing from this RFC".
---

# Design Review / ADR

## Why this exists

A design artifact is the source of truth for intent. It should explain the problem, constraints, alternatives, risks, and implementation direction clearly enough that future readers do not need the original conversation to reconstruct the decision.

This skill is the most direct application of the Elephant-Goldfish model:

- the **Elephant** develops the design through a context-rich discussion
- the **Goldfish** tests whether the resulting document survives without hidden context

A design is not ready merely because its author understands it. It is ready when a fresh reviewer can understand, challenge, and act from it.

## Operating principles

- Design preserves intent; implementation proves reality.
- Rigor scales with reversibility and blast radius.
- Important answers belong in the artifact, not only in chat.
- A reviewer should separate blocking concerns from optional improvements.

## Phase 0 — Observe

Read the draft and its references:

- current architecture
- related ADRs and RFCs
- relevant code or infrastructure
- compliance and security constraints
- scale and performance requirements
- ownership and operational context
- previously rejected approaches

Prompt:

> Summarize the decision this document is trying to make, the current system it relates to, and the constraints that appear to apply.

Deliverable:

- decision summary
- context and constraint list
- missing-reference list

## Phase 1 — Prioritize what is at stake

Determine:

- what is reversible
- what is hard or expensive to reverse
- what changes system boundaries
- what affects data integrity, public APIs, security, compliance, or customer commitments
- what creates long-lived operational burden
- what alternatives were considered or omitted

Prompts:

> Which parts of this decision are difficult or expensive to reverse?

> What is the blast radius if this design is wrong?

Deliverable:

- reversibility map
- blast-radius note
- missing-alternatives list
- recommended review depth

## Phase 2 — Understand the proposed design

Trace:

- request, event, and data flow
- state transitions
- persistence and consistency
- dependencies
- retries, timeouts, and idempotency
- security boundaries
- failure handling
- deployment and rollback
- observability and ownership

Prompt:

> Trace how a request, event, or unit of data flows through this proposed design from entry to completion, including failure and retry paths.

Deliverable:

- end-to-end design trace
- architecture or state-transition diagram
- dependency and ownership map

## Phase 3 — Surface gaps and risk

Look for:

- unstated assumptions
- vague goals or acceptance criteria
- unsupported capacity assumptions
- missing security, privacy, or compliance analysis
- missing data-integrity or consistency model
- unaddressed failure modes
- no rollback or migration story
- unclear operational ownership
- missing observability
- cost blind spots
- incomplete alternatives
- irreversible steps without approval
- implementation details that contradict the stated design

Classify findings as:

- blocking
- required before implementation
- recommended
- nice to have

Deliverable:

- severity-tagged findings list

## Phase 4 — Run the three Goldfish gates

### Gate 1 — Comprehension

> You have no prior context. Read this design document and its references. Explain what it is trying to accomplish, how the current system works in this area, and how the proposed design changes it. List anything unclear or unsupported.

Pass when the Goldfish reconstructs the problem, current state, target state, and trade-offs correctly.

### Gate 2 — Critic

> Act as a skeptical senior technical reviewer. Identify every unjustified assumption, missing edge case, non-functional gap, operational risk, security concern, cost concern, and alternative the author failed to consider. Separate blocking findings from optional improvements.

Pass when blocking findings are resolved or explicitly accepted by named decision owners.

### Gate 3 — Readiness

> Act as the engineer who must implement this design. Does this document contain everything needed to build, test, deploy, observe, roll back, and operate the change correctly on a first pass? List every unresolved question.

Pass when implementation-critical questions are answered in the document and remaining unknowns are explicit and owned.

Deliverable:

- Goldfish-reviewed document
- gate results
- unresolved-question list

## Phase 5 — Revise and decide

As author: revise until the design is clear, testable, operable, and all blocking gates pass.

As reviewer: give concrete feedback tied to exact sections and separate blocking from non-blocking comments.

Possible outcomes:

- approved
- approved with conditions
- revision required
- rejected with reasons
- deferred pending evidence

## Phase 6 — Systematize

Store the final artifact where future readers can find it and record:

- status: proposed, accepted, rejected, deprecated, or superseded
- decision date
- owners and reviewers
- implementation links
- rollout and test links
- superseding document
- revisit trigger

Deliverable:

- durable ADR/RFC with explicit lifecycle status

## Stop Conditions

Do not approve if:

- the problem or decision is unclear
- material constraints are unstated
- irreversible steps lack ownership
- security, compliance, or data-integrity impact is unresolved
- implementation depends on undocumented chat context
- rollback, testing, migration, or operational ownership is missing for a high-risk change
- the Goldfish cannot reconstruct the design
- blocking findings are unresolved or unowned

## Final Deliverable

A design-review package containing the decision summary, reversibility and blast-radius analysis, end-to-end trace, findings, alternatives and trade-offs, Goldfish gate results, final decision, and durable ADR/RFC status.
