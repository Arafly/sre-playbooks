---
name: rca
description: Use this AFTER an incident has been mitigated and service restored to produce a rigorous, evidence-based Root Cause Analysis (RCA). This skill transforms a raw incident timeline into a defensible postmortem that survives technical, operational, and leadership scrutiny. It applies the Elephant-Goldfish model: build the full evidence trail in a context-rich session (Elephant), then validate the RCA through independent Goldfish reviews for comprehension, alternative hypotheses, unsupported claims, hindsight bias, and operational readiness before publication. Trigger whenever the user wants to write, review, strengthen, or challenge a postmortem, incident report, or RCA, or says things like "help me write the RCA", "review this postmortem", or "are we missing anything from this incident analysis".
---

# Root Cause Analysis (RCA)

## Why this exists

The purpose of an RCA is **not** to assign blame.

The purpose is to understand:

- what happened
- why it happened
- why the system allowed it to happen
- how to reduce the probability or impact of recurrence

The most common RCA failure is not missing data.

It is **narrative bias**—constructing a clean story that confirms the first plausible explanation instead of following the evidence wherever it leads.

This skill exists to reduce that bias.

---

## Relationship to Incident Response

This playbook begins **after service has been restored**.

During a live incident use the **incident-response** skill.

That skill optimizes for:

> mitigate → stabilize → diagnose

This skill optimizes for:

> understand → validate → improve

---

## Operating principles

### Evidence before explanation

Every important claim should be supported by:

- logs
- metrics
- traces
- deployment history
- monitoring
- configuration
- chat transcripts
- timestamps
- ticket history

Avoid conclusions that rely only on memory.

---

### Root causes are usually systemic

Avoid stopping at:

> Engineer X made a mistake.

Instead ask repeatedly:

> Why did the system allow that mistake?

until reaching systemic contributors such as:

- missing validation
- inadequate testing
- weak review
- insufficient monitoring
- unclear ownership
- poor documentation
- unsafe defaults
- process gaps

---

### Separate facts from interpretation

Tag statements as:

- Verified fact
- Supported inference
- Plausible hypothesis
- Unknown

Readers should never need to guess which is which.

---

## Phase 1 — Build the Elephant

Assemble every available artifact.

Typical inputs:

- incident timeline
- logs
- traces
- metrics
- dashboards
- alerts
- deployment history
- configuration changes
- feature flags
- cloud audit logs
- infrastructure events
- chat transcripts
- ticket history
- customer reports
- stakeholder notes

Interview responders if needed.

Ask:

> What did you actually observe?

instead of

> What do you think caused it?

Build a timestamped timeline.

Every event should cite its evidence.

Deliverable:

- Evidence inventory
- Source map
- Complete incident timeline

Move on when:

- every major event has supporting evidence
- uncertainties are explicitly identified
- conflicting evidence is recorded rather than discarded

---

## Phase 2 — Build the causal model

Draft the RCA incrementally.

Avoid generating the whole document in one pass.

Suggested sections:

- Executive summary
- Timeline
- Customer impact
- Technical impact
- Detection
- Response
- Recovery
- Contributing factors
- Root causes
- What worked well
- What didn't
- Lessons learned
- Corrective actions
- Preventive actions
- Owners
- Target dates

Use causal chains instead of single-cause thinking.

Ask repeatedly:

> Why did this happen?

until further "why" no longer produces actionable system improvements.

Deliverable:

- Draft RCA
- Cause-and-effect chain
- Action register

---

## Phase 3 — Goldfish Gate 1: Comprehension

Use a completely fresh session.

Provide only:

- the RCA
- referenced artifacts

Prompt:

> Read this RCA and its referenced evidence.
>
> Explain:
>
> • what happened
> • why it happened
> • how it was detected
> • how it was resolved
> • what changes are planned.
>
> List every place where the document assumes knowledge it never explains.

Pass condition:

A new engineer can accurately reconstruct the incident without outside context.

---

## Phase 4 — Goldfish Gate 2: Critic

Use another fresh session.

Prompt:

> Assume the role of an experienced incident reviewer.
>
> Challenge every conclusion.
>
> Identify:
>
> • unsupported claims
> • alternative explanations
> • evidence gaps
> • logical leaps
> • hindsight bias
> • survivorship bias
> • confirmation bias
> • missing contributing factors
> • weak corrective actions
> • risks that remain unresolved.

Separate:

- Blocking findings
- Recommended improvements
- Nice-to-have observations

Pass condition:

Material objections are either addressed or explicitly documented.

---

## Phase 5 — Goldfish Gate 3: Operational readiness

Use another fresh session.

Prompt:

> Imagine this incident happens again at 03:00.
>
> Based only on this RCA:
>
> • would you recognize it?
> • would you know what to check?
> • would you know how to mitigate it?
> • would you know who owns it?
> • would the listed actions actually reduce recurrence?
>
> Identify anything missing.

Pass condition:

The RCA functions as an operational learning document, not merely historical documentation.

---

## Phase 6 — Publish and improve the system

Publish through the organization's review process.

Track every corrective action.

Recommended fields:

- owner
- priority
- due date
- success criteria
- verification method
- completion status

Feed improvements back into the wider system.

Examples:

- new alert → observability-slo-design
- deployment safeguard → service-reliability-onboarding
- Terraform improvement → terraform-iac-review
- Kubernetes hardening → kubernetes-cluster-onboarding
- IAM improvement → iam-access-review
- migration safeguard → migration-readiness

Deliverable:

- Published RCA
- Action tracker
- Cross-playbook improvements

---

## Two exits

An RCA may legitimately conclude:

### Root cause established

Evidence supports a defensible causal chain.

or

### Insufficient evidence

The available evidence cannot distinguish between competing explanations.

In that case, document:

- what remains unknown
- why
- what telemetry, logging, tracing, or evidence should exist next time

An honest "insufficient evidence" conclusion is stronger than an invented explanation.

---

## Stop Conditions

Do not publish an RCA if:

- major claims lack evidence
- timeline conflicts remain unresolved
- contributing factors are collapsed into a single simplistic cause
- action items lack owners
- action items lack measurable success criteria
- the Goldfish cannot reconstruct the incident
- the RCA blames individuals instead of improving systems
- corrective actions do not address the identified causes

---

## Final Deliverable

An RCA package containing:

- evidence inventory
- timestamped timeline
- verified facts vs. inferences
- causal chain
- customer and technical impact
- contributing factors
- systemic root causes
- corrective and preventive actions
- Goldfish review results
- action tracker
- links to related playbooks updated as a result