---
name: migration-readiness
description: Use this before migrating a service, database, platform, cloud provider, region, runtime, observability system, or deployment model — assessing whether the migration is safe to execute, not performing the production cutover itself. Covers source and target mapping, dependencies, data integrity, compatibility, cutover sequencing, reversible and irreversible steps, rollback or roll-forward strategy, rehearsals, go/no-go criteria, observability, ownership, and communication. Trigger whenever the user is planning, reviewing, or preparing for a migration, replatforming, upgrade, or cutover, or says things like "we're planning to migrate X", "is this migration plan safe", "are we ready to cut over", or "review this migration runbook".
---

# Migration Readiness

## Why this exists

Migrations fail because the path is under-specified: hidden dependencies, incorrect sequencing, irreversible changes, weak rollback, incomplete reconciliation, poor observability, or a runbook executed for the first time in production.

This skill produces a go, conditional-go, or no-go judgment. It does not authorize or execute the production cutover.

## Operating principles

- Separate destination design from migration-path safety.
- Prefer reversible transitions.
- Treat data correctness as first-class.
- Define decision rights before the migration window.

## Phase 0 — Observe

Review the RFC, current and target state, project plan, cutover runbook, test, rollback and data plans, dependencies, compliance requirements, and rehearsal history.

Identify scope, driver, timeline, downtime, RPO/RTO, data and traffic volume, ownership, and success criteria.

Deliverable:

- migration summary
- constraints
- decision rights
- open assumptions

## Phase 1 — Map what is moving

Inventory services, data, schemas, traffic, users, clients, integrations, credentials, certificates, DNS, queues, jobs, reports, monitoring, backups, compliance controls, pipelines, and procedures.

Classify by coupling, criticality, sensitivity, compatibility risk, sequence, owner, and validation.

Deliverable:

- scoped inventory
- dependency map
- in-scope/out-of-scope boundary
- ranked risk list

## Phase 2 — Trace the cutover end to end

For every step document precondition, action, owner, duration, validation, evidence, failure mode, retry, rollback point, communication, and downstream consequence.

Trace traffic, replication, write ownership, schema, DNS, clients, credentials, jobs, monitoring, and rollback or roll-forward boundaries.

Deliverable:

- numbered cutover trace
- state-transition diagram
- partial-failure behavior

## Phase 3 — Surface irreversible and high-risk steps

Look for destructive schema changes, one-way transforms, split brain, dual-write divergence, lag, incompatible clients, DNS or certificate timing, credential cutover, duplication or loss, non-idempotent scripts, single-person knowledge, missing backups or restores, hidden third parties, insufficient capacity, weak observability, and rollback exceeding the window.

Classify each step as reversible, time-bounded reversible, compensatable, roll-forward only, or irreversible.

Deliverable:

- migration risk register
- point-of-no-return map
- integrity and reconciliation risks
- blockers requiring human decision

## Phase 4 — Define rehearsal, observability, and go/no-go controls

Use lower environments, representative data, tenant subsets, shadow traffic, canaries, synthetics, restore tests, failure injection, or tabletop rehearsal.

Define success metrics, explicit go criteria, pause, rollback and roll-forward triggers, and decision owners.

Deliverable:

- rehearsal plan
- observability plan
- go/no-go criteria
- triggers
- decision-rights list

## Phase 5 — Rehearse and Goldfish-test the runbook

Run the rehearsal, capture timings, deviations, failures, permissions, dependencies, manual fixes, observability gaps, rollback behavior, and reconciliation results, then update the runbook.

Goldfish prompt:

> Could an engineer execute, validate, pause, roll back or roll forward, communicate, and declare completion from this document alone? What is missing or ambiguous?

Deliverable:

- rehearsed and Goldfish-reviewed runbook
- evidence and updated timings

## Phase 6 — Produce the readiness decision

Return one of:

- Go
- Conditional go
- No-go

Include scope, window, decision owner, evidence, residual risks, accepted risks, blockers, rollback or roll-forward strategy, and next decision point.

## Two exits

### Ready to proceed

The migration package is reviewed, rehearsed, observable, owned, and decision-ready.

### Defensible no-go or conditional-go judgment

Use when rollback, reconciliation, compatibility, ownership, observability, rehearsal, or fresh-context usability is inadequate.

## Stop Conditions

Do not recommend go if ownership, scope, backups or restores, reconciliation, rollback or roll-forward, irreversible approvals, dependency testing, decision-makers, observability, rehearsal, or authority are missing.

## Final Deliverable

A migration-readiness package containing current and target state, scope, dependencies, cutover sequence, point of no return, integrity and reconciliation plan, risk register, rehearsal evidence, observability, go/no-go criteria, rollback and roll-forward triggers, communications, ownership, and final readiness judgment.
