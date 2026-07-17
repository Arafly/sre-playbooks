---
name: observability-slo-design
description: Use this to move a service from generic platform-level monitoring, such as CPU and memory dashboards, to meaningful service-level reliability — defining SLIs, SLOs, error budgets, burn-rate alerts, dashboards, and runbooks tied to what users actually experience. Trigger whenever the user wants to design, review, or improve monitoring, alerting, dashboards, SLOs, SLIs, or error budgets, or says things like "our alerts are too noisy", "we don't have real SLOs", "help me figure out what to monitor", or "how do we define healthy for this service".
---

# Observability / SLO Design

## Why this exists

CPU and memory dashboards tell you about machines.

They do not necessarily tell you whether users are having a bad time.

This skill moves a service from platform-level monitoring to service-level reliability: signals, objectives, alerts, dashboards, and runbooks tied to user experience and business impact.

The goal is not to monitor everything.

The goal is to define what “healthy” means, measure it, and alert only when action is needed.

---

## Phase 0 — Observe

Inventory what exists today.

Look for:

- dashboards
- alerts
- paging policies
- on-call pain points
- runbooks
- SLOs or SLAs
- synthetic checks
- logs
- traces
- metrics
- incident history
- noisy alerts
- ignored alerts
- missing alerts

Ask whoever has been on-call:

> Which alerts do you mute, ignore, or dread, and why?

That answer often teaches more than the alert config.

Deliverable:

- Monitoring and alerting inventory.
- Noisy-alert list.
- Missing-signal list.

---

## Phase 1 — Prioritize critical user journeys

Identify the small number of user journeys that matter most.

Do not start with every endpoint.

Start with workflows where failure would cause:

- user pain
- revenue loss
- support tickets
- regulatory impact
- broken contractual expectations
- operational escalation
- data correctness concerns

Examples:

- checkout succeeds
- payment completes
- login works
- search returns results
- customer data is fresh
- report generation completes
- order status is correct
- webhook delivery succeeds
- batch pipeline completes before business deadline

Useful prompt:

> Which workflows would cause the most damage or complaints if they broke?

Deliverable:

- A short list of 2–4 critical user journeys.

---

## Phase 2 — Define candidate SLIs

For each journey, define a Service Level Indicator that reflects user experience.

Common SLI types:

- request success rate
- latency at a meaningful percentile
- data freshness
- job completion before deadline
- correctness of output
- availability of a critical endpoint
- queue processing delay
- webhook delivery success
- synthetic journey success
- error-free transaction rate

Avoid choosing metrics only because they are easy to collect.

A good SLI should answer:

> Are users getting the experience we promised?

Pull current baseline performance before choosing targets.

Do not guess 99.9% because it sounds professional.

Useful prompts:

> What SLI best represents user experience for this journey?

> What is the current baseline for this signal?

> What would users notice before we notice?

Deliverable:

- Candidate SLI and current baseline per critical journey.

---

## Phase 3 — Surface gaps and vanity metrics

Identify where current monitoring fails to represent user impact.

Flag:

- critical journeys with no coverage
- alerts on causes rather than symptoms
- CPU/memory alerts without user impact
- dashboards nobody uses
- alerts without runbooks
- alerts without owners
- alerts that fire too late
- alerts that fire too often
- missing lower-environment signals
- missing dependency visibility
- missing trace correlation
- missing business-flow metrics
- missing data freshness checks

Examples of weak alerts:

- CPU above 80% with no user impact context
- pod restart count without service impact
- queue depth without age or SLA
- HTTP 500 count without request volume
- static latency threshold without burn-rate context

Deliverable:

- Coverage-gap list.
- Vanity/noisy-metric list.
- Alert quality findings.

---

## Phase 4 — Define SLOs and burn-rate alerts

For each selected SLI, define an SLO.

An SLO should include:

- user journey
- SLI definition
- measurement window
- target
- exclusions, if any
- data source
- owner
- rationale
- alerting policy
- runbook link

Example:

```text
Journey: Checkout
SLI: Percentage of checkout requests completing successfully
Window: 30 days
Target: 99.9%
Error budget: 0.1%
Fast-burn alert: page if budget is burning rapidly over short windows
Slow-burn alert: ticket if budget is burning steadily over longer windows
```

Prefer burn-rate alerts over single static thresholds.

A burn-rate alert answers:

> If this continues, will we breach the reliability target?

Use multiple windows:

- fast burn: urgent, page now
- slow burn: non-urgent, ticket or investigate during working hours

Deliverable:

- SLO definitions.
- Error budget calculations.
- Burn-rate alert rules.
- Alert severity mapping.

---

## Phase 5 — Contribute one SLO end to end

Do not roll out ten SLOs at once.

Ship one useful SLO end to end as the proof of concept.

That means:

- SLI query works
- SLO target is documented
- dashboard exists
- burn-rate alert exists
- runbook exists
- owner is clear
- alert route is correct
- on-call knows what to do

Goldfish-test the alert and runbook.

Fresh session prompt:

> You are on-call at 3am with no prior context. Read this alert and linked runbook. Do you know what is broken, how severe it is, what to check first, and what action to take? What is unclear or missing?

If the Goldfish cannot act from the alert and runbook, the runbook is incomplete.

Deliverable:

- One live SLO.
- One burn-rate alert.
- One dashboard or panel.
- One runbook that passes the Goldfish test.

---

## Phase 6 — Systematize

Document the SLO catalog and the reasoning behind each target.

Include:

- why this journey matters
- why this SLI was chosen
- why this target was chosen
- baseline performance
- error budget policy
- alert routing
- runbook links
- owner
- review cadence
- known limitations

This prevents future teams from treating numbers as sacred without understanding their rationale.

Deliverable:

- SLO catalog.
- Alert/runbook ownership map.
- Review cadence.

---

## Two exits

Not every service is ready for a formal SLO.

A legitimate output may be a documented decision not to set one yet.

Reasons might include:

- traffic is too low for meaningful percentages
- service ownership is unclear
- instrumentation is not trustworthy
- user journeys are not well defined
- the service is too immature
- business risk tolerance is unknown
- there is no on-call path to act on alerts

In that case, the defensible judgment is:

> Do not create formal SLOs yet. First fix instrumentation, ownership, or journey definition.

That is a valid completion of this skill.

---

## Stop Conditions

Stop and ask a human owner before finalizing SLOs if:

- the SLO has customer, contractual, regulatory, or SLA implications
- the target could affect release velocity or error-budget policy
- the service has unclear ownership
- the user journey has business ambiguity
- the metric may not represent real user experience
- alert routing or on-call ownership is unclear
- data quality is not trusted

SLOs are not just technical metrics. They encode reliability promises.

---

## Final Deliverable

A service-level reliability package:

- critical user journeys
- SLIs
- SLOs
- error budget rationale
- burn-rate alerts
- dashboard links or definitions
- runbooks
- owners
- review cadence
- known gaps or reasons not to formalize yet
