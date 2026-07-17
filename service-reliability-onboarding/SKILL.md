---
name: service-reliability-onboarding
description: Use this when taking ownership of a production service or system as an SRE/platform engineer — joining a new team, joining an on-call rotation, inheriting a service from another team, or any "here's the pager, good luck" moment. This is the kernel skill: it runs the full safe-entry arc and invokes terraform-iac-review, kubernetes-cluster-onboarding, observability-slo-design, incident-response, and rca sibling skills at the right phase instead of duplicating them. Trigger whenever the user says things like "I'm taking over this service", "I just joined this team and need to get up to speed", "I'm now on-call for X", "walk me through onboarding to this system", or "help me understand this production service", even if they do not use the word SRE or onboarding explicitly.
---

# Service Reliability Onboarding

## Why this exists

Taking ownership of a production service you did not build is one of the highest-risk, highest-leverage moments in an SRE or platform engineer's relationship with a system.

Most damage new owners cause does not come from bad intentions. It comes from acting before understanding, or understanding without preserving that understanding for the next person.

This skill is the kernel. It runs the full safe-entry arc and calls out to sibling skills for domain-specific heavy lifting, so the shared arc is written once and reused everywhere.

The universal pattern is:

**Map the terrain → Find the critical paths → Surface the risk → Externalize context → Validate with a fresh reviewer → Act or advise.**

The service ownership arc is:

**Observe → Map Terrain → Understand Critical Paths → De-risk → Operationalize → Contribute → Systematize**

---

## Operating principles

### Tool, not toy

Do not ask AI to "explain the service" in one shot.

Use it as a thinking partner:

1. Give it real artifacts.
2. Make it interrogate assumptions.
3. Define what good understanding looks like.
4. Preserve the understanding in a durable artifact.
5. Validate that artifact with a fresh reviewer.

### Elephant / Goldfish

The Elephant is the long-running context-rich working session. It accumulates docs, architecture, corrections, decisions, and open questions.

The Goldfish is a fresh session with zero prior context. It knows only what you hand it.

If a Goldfish cannot understand your service brief, design note, runbook, or handover doc, the artifact is unclear or incomplete.

### Act or advise

Not every onboarding journey ends with a change.

There are two valid exits:

- **Safe action:** a small, reviewed, low-blast-radius improvement.
- **Defensible judgment:** a risk memo, recommendation, escalation, or "do not touch this yet" conclusion.

---

## Phase 0 — Observe

Read the service's own account of itself before doing anything else.

Look for:

- README
- architecture docs
- runbooks
- on-call docs
- dashboards
- alert routes
- service catalog entry
- ownership notes
- recent incident history
- release/deployment docs
- last handover doc, if one exists

Useful prompt:

> Summarize what this service does, who depends on it, who owns it, how it is operated, and what remains unclear from these docs.

Deliverable:

- A one-page service overview with:
  - purpose
  - owners
  - consumers
  - dependencies
  - environments
  - current unknowns

Move on when:

- you can explain the service's purpose in plain English
- you know who owns it
- you know where to find its operational docs, or you have documented that they are missing

---

## Phase 1 — Map the terrain

Build the service map.

Understand:

- upstream callers
- downstream dependencies
- data stores
- queues
- caches
- external APIs
- deploy path
- runtime platform
- infrastructure ownership
- secrets and config flow
- blast radius if the service fails

This is where the kernel invokes sibling modules.

### Invoke: `kubernetes-cluster-onboarding`

Use this module if the service runs on Kubernetes.

Apply it to:

- service namespaces
- workloads
- ingress/service mesh path
- services
- configmaps/secrets
- RBAC
- network policies
- probes/resources/PDBs
- GitOps/Helm/Kustomize ownership

Expected module output:

- namespace/workload inventory
- traffic path
- risk findings
- troubleshooting commands

### Invoke: `terraform-iac-review`

Use this module if the service's infrastructure is defined in Terraform, OpenTofu, Pulumi, CloudFormation, CDK, or any IaC.

Apply it to:

- relevant root modules
- shared modules
- state/backend
- environments/workspaces
- IAM
- networking
- databases
- DNS
- queues
- secrets/KMS
- CI/CD roles

Expected module output:

- module/state map
- blast-radius notes
- drift/risk findings
- safe-change recommendations

Useful prompt:

> Based on the service docs, Kubernetes resources, and IaC, map how this service receives traffic, stores state, depends on other systems, and gets deployed.

Deliverable:

- Dependency map.
- Runtime map.
- Infrastructure map.
- Blast-radius summary.

Move on when:

- you know how traffic reaches the service
- you know where state lives
- you know what infrastructure supports it
- you know which sibling modules produced the infra/runtime details

---

## Phase 2 — Understand critical paths and history

Find the 2–3 workflows that matter most.

These are the workflows that would page someone, lose money, corrupt data, breach a promise, or create customer pain if broken.

Look for:

- revenue-critical journeys
- customer-facing journeys
- data correctness paths
- batch deadlines
- API dependencies
- background workers
- webhook paths
- admin or support workflows
- regulator/customer commitments

Mine history for:

- recent high-churn files/configs
- recent incident tickets
- recent PRs
- reverted changes
- flaky tests
- recurring alerts
- ownership patterns

Useful prompts:

> Which workflows in this service would cause the most damage if they broke?

> Trace this critical workflow from input to output, including validation, persistence, side effects, retries, and external calls.

> Summarize the purpose of the highest-churn files/configs touching this service over the last six months.

Deliverable:

- Critical-path list.
- Input-to-output trace for each critical path.
- Churn/risk map.
- Ownership notes.

Move on when:

- you can explain the service's most important workflows end to end
- you know what changed recently
- you know which paths are most fragile or business-critical

---

## Phase 3 — De-risk

Identify known and likely failure modes before making changes.

Read the last 3–6 months of:

- incidents
- postmortems
- high-severity tickets
- customer escalations
- recurring alerts
- failed deployments
- rollback events
- support tickets
- dependency outages

If no incidents or postmortems exist, do not assume stability. Treat that as a documentation and learning gap.

Use the `incident-response` skill only if something is actively broken right now.

### Invoke: `incident-response`

Use this module immediately if there is an active incident.

Its order overrides this kernel:

**Mitigate → Stabilize → Scope → Preserve Evidence → Diagnose → Repair → Communicate → Learn**

Do not continue onboarding while production is actively burning.

After the incident is resolved, the incident-response skill should hand off to the `rca` skill.

### Invoke: `rca`

Use this module when reviewing past incidents or writing a post-incident analysis.

Expected module output:

- evidence-based incident timeline
- root/contributing factor analysis
- corrective actions
- narrative-bias check
- Goldfish-tested RCA

Useful prompts:

> What known failure modes does this service have?

> Which failure modes have runbooks, alerts, or automated mitigations?

> Which past incidents suggest unresolved systemic risk?

Deliverable:

- Failure-mode list.
- Runbook coverage map.
- Open reliability risks.
- Escalations or recommendations where action is not safe yet.

Move on when:

- you know the service's known failure modes
- you know which failure modes are detected and which are blind spots
- you know which risks require action, escalation, or further review

---

## Phase 4 — Operationalize

Define what "healthy" means and prepare to debug the service under pressure.

This is where the kernel invokes observability work.

### Invoke: `observability-slo-design`

Use this module if:

- the service has no SLOs
- alerts are noisy
- dashboards are mostly CPU/memory
- health is defined by infrastructure metrics rather than user experience
- on-call cannot tell what user journey is broken from an alert
- runbooks are missing or not actionable

Expected module output:

- critical user journeys
- candidate SLIs
- SLO definitions
- burn-rate alert strategy
- dashboard/runbook package
- reasons not to formalize SLOs yet, if applicable

Build a debugging toolkit:

- log queries
- trace entry points
- dashboards
- health checks
- synthetic checks
- admin/debug commands
- common failure signatures
- rollback commands
- escalation paths

Useful prompts:

> What signals prove this service is healthy from a user perspective?

> If this service pages at 3am, what should the on-call check first?

> Which alerts lack a clear action or runbook?

Deliverable:

- Service health definition.
- SLO/SLI package or documented reason not to formalize yet.
- Troubleshooting runbook.
- Alert/runbook ownership map.

Move on when:

- you know what healthy means
- you know how the service alerts
- you know how to debug the most likely failures
- the runbook is usable by someone who lacks hidden context

---

## Phase 5 — Contribute

Ship one small, correct, low-blast-radius improvement.

Good first contributions:

- missing runbook
- dashboard panel
- broken alert route
- noisy alert cleanup
- missing probe
- missing test
- clearer error message
- small validation fix
- documentation improvement
- safer deployment check
- observability improvement around a known failure path

Avoid as a first contribution unless explicitly approved:

- large refactors
- data model changes
- public API changes
- deployment/rollback logic changes
- IAM/security changes
- core abstractions
- database migrations
- shared base config changes
- timeout/retry/concurrency changes
- metric label/name changes used by dashboards or alerts

Before shipping anything non-trivial, write a short design note.

Design note should include:

- problem
- proposed change
- files/resources touched
- acceptance criteria
- tests/validation
- risk
- rollback
- owner/reviewer

Goldfish-test the note:

> You have no prior context except this design note and referenced artifacts. What change is being proposed, what does it affect, what risks or ambiguities remain, and could you implement or review it safely from this note alone?

If the Goldfish cannot explain the change, fix the note.

Deliverable:

- A small, reviewed contribution.
- Or a defensible judgment that the right next step is escalation, not a change.

Move on when:

- the blast radius is understood
- the test/validation path is credible
- the rollback path is clear
- a human reviewer can understand why the change is safe

---

## Phase 6 — Systematize

Leave a trail.

Write or update the service context document.

Include:

- service purpose
- architecture diagram
- owners
- upstream/downstream dependencies
- runtime platform
- infrastructure modules
- deployment path
- critical workflows
- known failure modes
- SLOs/SLIs
- dashboards
- alert routes
- runbooks
- gotchas
- unresolved risks
- who to ask
- suggested next safe improvements

Goldfish-test the context doc:

> You have no prior context. Read this service context doc. Explain what the service does, how it fails, how to debug it, what risks remain, and what a safe next contribution might be. What is unclear or unsupported?

Deliverable:

- Updated service onboarding/context doc.
- Gotchas log.
- Next-action list.

---

## Two exits

This skill may end with either:

1. **Safe action**
   - a small PR
   - runbook update
   - alert cleanup
   - dashboard improvement
   - probe/test/validation fix

2. **Defensible judgment**
   - risk memo
   - escalation
   - ownership clarification
   - recommendation not to touch yet
   - SLO readiness decision
   - migration/incident/compliance follow-up

Both are valid completions.

---

## Stop Conditions

Stop and ask a human owner before changing anything if:

- ownership is unclear
- production impact is possible
- the change touches data integrity
- customer money, orders, payments, or regulated workflows are involved
- the change touches auth, IAM, secrets, CI/CD, deployment, rollback, schema, or public APIs
- rollback is unclear
- the Goldfish cannot understand the design note
- your proposed action depends on an assumption you cannot verify from artifacts

Do not let AI fluency become an excuse to avoid the one Slack message that prevents an incident.

---

## Final Deliverable

A service reliability onboarding package:

- service overview
- dependency/runtime/infra maps
- critical workflows
- failure modes
- operational health definition
- SLO/SLI package or readiness judgment
- runbook/debugging toolkit
- risk map
- first safe contribution or defensible recommendation
- updated service context doc
