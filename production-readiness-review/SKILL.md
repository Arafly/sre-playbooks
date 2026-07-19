---
name: production-readiness-review
description: Use this before launching a new service to production, promoting a major feature, onboarding a service to on-call, or signing off a go-live — assessing whether a system is ready to be operated in production, not building it. Covers reliability, observability, operability, deployment and rollback, capacity and scale, dependency failure handling, security, and data safety, and produces a go, conditional-go, or no-go judgment. Trigger whenever the user mentions a production readiness review, PRR, launch readiness, go-live sign-off, "are we ready to launch", "is this service production-ready", or "review this before it goes to prod".
---

# Production Readiness Review

## Why this exists

Launching or promoting a service without a readiness gate is how avoidable incidents ship: the code passed its tests, but nobody could operate it, observe it, scale it, or roll it back when it failed at 3am.

A production readiness review asks a different question than "does it work?" It asks: can this be *operated* in production, and can we get out safely if it cannot?

This skill produces a go, conditional-go, or no-go judgment. It does not authorize or perform the launch.

## Operating principles

- Assess readiness to operate, not just readiness to build.
- Weight reversibility: a service you can roll back is safer than one that merely passed review.
- Judge the day-2 story — run, observe, debug, scale, recover — not the demo.
- Define decision rights and launch scope before the window.

## Phase 0 — Observe

Read the design doc, SLO/SLI definitions, runbooks, dashboards, deploy and rollback plan, dependency list, capacity/load-test results, security review, on-call and escalation plan, and the launch scope and timeline.

Identify the launch driver, blast radius, target environments, expected traffic, data involved, and who will own the service after launch.

Deliverable:

- launch summary
- scope and timeline
- decision rights
- open assumptions

## Phase 1 — Map what is launching

Inventory the service or feature, its upstream and downstream dependencies, data stores, external integrations, environments, traffic expectations, and post-launch owner.

Classify by criticality, blast radius, reversibility, and whether each dependency has a defined failure behavior.

Deliverable:

- scoped inventory of what goes live
- dependency map
- blast-radius summary
- post-launch ownership

## Phase 2 — Assess the readiness dimensions

Work the checklist, gathering evidence for each dimension rather than asserting it:

- **Reliability:** SLOs/SLIs defined, error budget, realistic targets.
- **Observability:** metrics, logs, traces, and dashboards that reflect user experience; alerts that are actionable and routed.
- **Operability:** runbooks, on-call rotation, escalation path, ownership.
- **Deployment and rollback:** repeatable deploy, a tested rollback, and safe rollout (canary/gradual).
- **Capacity and scale:** load tested to expected peak, resource limits set, autoscaling behavior understood.
- **Dependencies:** timeouts, retries, circuit breakers, and defined behavior when each dependency is down.
- **Security:** authn/authz, secret handling, data classification, and any required review complete.
- **Data:** backups, a restore that has actually been tested, and reversible or expand/contract migrations.
- **Failure handling:** graceful degradation and no single point of failure on the critical path.

Deliverable:

- readiness matrix with evidence per dimension
- dimensions marked ready, weak, or missing

## Phase 3 — Surface launch-blocking risk

Look for no tested rollback, unvalidated capacity, missing or non-actionable alerts, absent runbooks, an untested restore, single points of failure, no on-call ownership, dependencies without timeouts or fallback, missing error budget, unhandled failure modes, and irreversible launch steps.

Classify each gap as launch-blocking or non-blocking-with-follow-up.

Deliverable:

- readiness risk register
- launch-blocking list
- accepted-risk candidates requiring an owner

## Phase 4 — Define launch controls

Define explicit go/no-go criteria, the rollout plan (canary or gradual with checkpoints), rollback and pause triggers, the during-launch monitoring plan, decision owners, and the post-launch validation checks and review date.

Deliverable:

- go/no-go criteria
- rollout and rollback plan
- during-launch monitoring plan
- decision-rights list

## Phase 5 — Validate with a fresh reviewer

Goldfish-test the readiness package and the runbook it depends on.

Goldfish prompt:

> You are the on-call engineer the night after launch, with no prior context. From these docs alone, can you tell what healthy looks like, detect the top failure modes, debug them, and roll back? What is missing or ambiguous?

Deliverable:

- Goldfish-reviewed readiness package
- gaps found and closed

## Phase 6 — Produce the readiness decision

Return one of:

- Go
- Conditional go
- No-go

Include scope, launch window, decision owner, evidence, residual risks, accepted risks, blockers, rollback plan, and the post-launch review date.

## Two exits

### Ready to launch

The service is reliable, observable, operable, deployable with a tested rollback, capacity-validated, dependency-safe, and owned — with a fresh reviewer able to operate it from the docs alone.

### Defensible no-go or conditional-go judgment

Use when rollback, capacity, observability, on-call ownership, restore testing, dependency failure handling, or fresh-context operability is inadequate. A clear conditional-go with named blockers and owners is a complete result.

## Stop Conditions

Do not recommend go if rollback is untested, capacity is unvalidated, alerts are missing or non-actionable, runbooks are absent, restore has never been tested, on-call ownership is undefined, critical dependencies lack failure handling, or a fresh reviewer cannot operate the service from the documentation.

## Final Deliverable

A production-readiness package containing the launch scope, dependency and blast-radius map, readiness matrix with evidence, risk register, launch-blocking list, rollout and rollback plan, during-launch monitoring plan, go/no-go criteria, decision rights, and the final go, conditional-go, or no-go judgment.
