---
name: finops-review
description: Use this to understand and reduce cloud or infrastructure spend responsibly — reviewing a cloud bill, explaining cost growth, finding waste, evaluating commitment options, or answering "why is our infrastructure spend so high" before cutting anything. Covers cost allocation, top spend drivers, unit economics, idle and underutilized resources, data-transfer costs, licensing, commitments, and safely delivering savings without removing quiet but load-bearing capacity. Trigger whenever the user mentions cloud costs, cost optimisation, FinOps, rightsizing, budgets, reservations, Savings Plans, cost anomalies, or says things like "our AWS bill doubled" or "help us find where our infrastructure spend is going".
---

# FinOps Review

## Why this exists

Careless cost cutting causes outages. Avoiding cost decisions wastes money. This skill separates genuine waste from resilience, compliance, seasonal, and business-critical spend.

## Operating principles

- Treat cost as an engineering signal, not only a finance number.
- Prefer unit cost over total cost where possible.
- Preserve reliability intent.
- Validate realised savings after implementation.

## Phase 0 — Observe

Review cost reports, budgets, forecasts, anomalies, tagging, account structure, commitments, marketplace and licensing, recent traffic or architecture changes, and FinOps policy.

Break down spend by service, account, region, environment, team, product, resource, tag, purchase model, and shared/direct allocation.

Deliverable:

- cost breakdown
- allocation/tagging quality note
- cost-data gaps

## Phase 1 — Find the 20% driving 80% of spend

Rank compute, databases, Kubernetes, storage, backups, data transfer, NAT, load balancers, observability, streaming, CDN, licences, support, idle environments, and commitment utilisation.

Compare current vs previous period, budget, forecast, usage, and business volume.

Deliverable:

- ranked cost drivers
- explanation of major changes

## Phase 2 — Understand why each driver costs what it does

Separate growth, waste, resilience, architecture, pricing model, overprovisioning, duplicate environments, data transfer, inefficient queries or storage, telemetry volume, managed-service premiums, missing autoscaling, unsafe floors, and commitment mismatch.

Deliverable:

- cause, owner, business context, and unit economics where possible

## Phase 3 — Separate waste from load-bearing cost

Classify findings into:

- provably safe candidates
- optimisation candidates requiring validation
- intentionally resilient or load-bearing spend

Record current cost, estimated saving, evidence, owner, observation window, reliability purpose, rollback, risk, and confidence.

Deliverable:

- safe-to-act list
- owner-confirmation list
- intentional-resilience list
- risk-adjusted savings estimate

## Phase 4 — Operationalize cost visibility

Improve tagging, ownership, budgets, anomaly detection, dashboards, forecasts, showback or chargeback, commitment tracking, and review cadence.

Deliverable:

- repeatable way to answer what a service or team costs

## Phase 5 — Deliver one safe saving

Prefer reversible steps: stop before delete, snapshot first, reduce gradually, test smaller sizes, schedule non-production shutdown, remove confirmed unused licences or resources, or make limited commitments after usage analysis.

Goldfish prompt:

> Is this genuinely safe to optimise? What quiet, seasonal, recovery, compliance, or infrequent dependency may have been missed, and is restoration credible?

Validate service health and realised savings afterwards.

Deliverable:

- one implemented and validated saving
- or a defensible recommendation

## Phase 6 — Systematize

Document top drivers, unit costs, allocation, tags, ownership, approved patterns, commitments, anomalies, review cadence, savings register, resilience costs, and unresolved opportunities.

## Two exits

### Safe action

Examples include removing a confirmed orphaned resource, scheduling non-production capacity, removing unused licences, reducing validated excessive retention, or right-sizing a low-risk workload.

### Defensible judgment

Use when ownership, seasonal usage, resilience intent, business requirements, commitment approval, or evidence is unclear.

## Stop Conditions

Stop before changing production, redundancy, backups, disaster recovery, security, audit, compliance, retention, scaling, availability, or long-term commitments when ownership, evidence, rollback, or authority is unclear.

## Final Deliverable

A FinOps review package containing the cost breakdown, allocation assessment, ranked drivers, causes and unit economics, classified opportunities, risk-adjusted savings, one validated action or recommendation, and a durable FinOps process.
