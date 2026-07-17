---
name: incident-response
description: Use this DURING an active, ongoing incident — something is currently broken, degraded, or at risk right now. This is the real-time, mitigate-first playbook: Mitigate -> Stabilize -> Scope -> Preserve Evidence -> Diagnose -> Repair -> Communicate -> Learn. It deliberately does NOT tell you to fully understand the system before acting, because that is the wrong move mid-incident. Trigger immediately whenever the user describes a live outage, firing alert, production degradation, user-visible errors, data-risk event, or anything indicating an active incident in progress. Hand off to the rca skill once the incident is resolved.
---

# Incident Response

## Why this exists

Most playbooks in this family start with understanding before action.

This one does not.

During a live incident, the priority is to reduce user and business impact first. Deep understanding comes after the bleeding is controlled.

The correct order is:

**Mitigate → Stabilize → Scope → Preserve Evidence → Diagnose → Repair → Communicate → Learn**

If you catch yourself trying to fully map the system before doing anything, stop. That instinct is right for onboarding and wrong during an active incident.

---

## Phase 1 — Mitigate

Protect users and the business first.

Ask:

- Can we roll back?
- Can we fail over?
- Can we disable the broken feature?
- Can we shed load?
- Can we route around the dependency?
- Can we pause the risky job?
- Can we reduce impact while preserving evidence?

Do the fastest safe thing that is likely to reduce impact, even if you do not yet know the root cause.

A rollback that fixes symptoms without explaining them is a correct incident-response action.

Useful prompt:

> Given these symptoms, what is the fastest reversible action that would likely reduce user impact right now?

Deliverable:

- A mitigation action or explicit decision that no safe mitigation is available yet.

---

## Phase 2 — Stabilize

Confirm the mitigation actually worked.

Check the metric or user-impact signal that matters, not just whether things “look better.”

Examples:

- error rate
- latency
- failed transactions
- queue depth
- dropped events
- data freshness
- failed logins
- customer support reports
- synthetic checks
- payment or order success rate

If the mitigation did not help, revert it where appropriate and try the next safest option.

Do not stack unverified changes on top of each other.

Deliverable:

- A clear statement of whether impact is reduced, unchanged, or worsening.

---

## Phase 3 — Scope

Now spend a short, focused period understanding how bad the incident is.

This is triage, not deep investigation.

Ask:

- Which users are affected?
- Which regions are affected?
- Which services are affected?
- Which workflows are broken?
- Is data correctness at risk?
- Are downstream systems affected?
- Is this full outage, partial degradation, or elevated risk?
- When did it start?
- Is impact still ongoing?

Deliverable:

- A short impact statement.

Example:

> Since 14:05 UTC, checkout requests in eu-west-1 have had a 35% failure rate. Browsing is unaffected. No evidence of data corruption yet. Mitigation reduced failures to 4% by 14:21 UTC.

---

## Phase 4 — Preserve Evidence

Preserve evidence continuously, in parallel with the response.

Do this before logs roll off, autoscaling settles, pods restart, queues drain, or someone “helpfully” clears the state.

Capture:

- timestamps
- alerts
- dashboards
- relevant logs
- traces
- deploys
- config changes
- feature-flag changes
- infrastructure changes
- upstream provider status
- commands run
- mitigations attempted
- who did what
- what changed after each action

The goal is not to write the RCA yet. The goal is to prevent the RCA from relying on memory.

Deliverable:

- A raw incident timeline with evidence links.

---

## Phase 5 — Diagnose

With the bleeding stopped or reduced, go deeper.

Correlate the timeline against:

- deployments
- config changes
- traffic patterns
- dependency failures
- infrastructure events
- database changes
- queue behavior
- autoscaling events
- provider status
- feature flags
- recent incidents
- recent PRs or releases

This is where you start building the Elephant: a context-rich working session that accumulates the full incident story.

Useful prompts:

> Based on this timeline, what hypotheses explain the symptoms?

> What evidence supports or weakens each hypothesis?

> What changed before the incident started?

> What alternative explanations are still plausible?

Deliverable:

- A ranked hypothesis list with supporting and contradicting evidence.

---

## Phase 6 — Repair

Implement the actual fix.

Keep the repair distinct from the mitigation.

Mitigation answers:

> How do we reduce impact right now?

Repair answers:

> How do we correct the underlying fault safely?

Validate the fix as you would any other production change unless ongoing impact genuinely does not allow for it.

Prefer:

- tests
- canary
- staged rollout
- feature flag
- targeted config change
- rollback-ready deployment
- peer review where possible

Deliverable:

- A validated fix or a documented temporary workaround with follow-up action.

---

## Phase 7 — Communicate

Communicate on a cadence appropriate to severity.

Do not go silent just because you do not have the answer yet.

Useful update shape:

```text
Status: Investigating / Mitigated / Recovering / Resolved
Impact: Who or what is affected
Action: What we have done
Next: What we are checking or doing next
ETA: Only if known; do not invent one
```

“Still investigating, no material change” is a valid update.

Avoid:

- speculation presented as fact
- blame
- overconfident root cause claims before evidence exists
- promising timelines you cannot control

Deliverable:

- A stakeholder update trail.

---

## Phase 8 — Learn and hand off to RCA

Once the incident is resolved, hand the preserved evidence to the `rca` skill.

Do not write the postmortem from memory.

Memory is exactly what Phase 4 exists to avoid.

The RCA should start from:

- raw timeline
- impact statement
- mitigations
- repair actions
- evidence links
- unresolved questions
- candidate contributing factors

Deliverable:

- A complete RCA handoff bundle.

---

## Goldfish's role here is smaller

During the hot phase, do not spend time Goldfish-testing your full understanding.

There is no time, and the cost of delaying mitigation is too high.

Goldfish becomes central again in the RCA skill, where it is used to challenge narrative bias, unsupported causal claims, and missing evidence.

---

## Stop Conditions

Escalate immediately if:

- user impact is ongoing and you do not have authority to mitigate
- data corruption is possible
- security or privacy exposure is suspected
- customer money, orders, payments, or regulated workflows are affected
- rollback or failover authority is unclear
- multiple teams or external providers are involved
- the incident exceeds your role, access, or confidence level

Do not wait to “understand more” before escalating a high-severity incident.

---

## Final Deliverable

A raw incident response bundle, not a polished RCA.

It should include:

- impact statement
- timeline
- alerts
- actions taken
- evidence links
- mitigations attempted
- current status
- unresolved questions
- handoff notes for RCA

The polished causal narrative belongs in the `rca` skill.
