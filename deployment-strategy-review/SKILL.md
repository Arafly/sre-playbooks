---
name: deployment-strategy-review
description: Use this to safely understand, audit, or improve how a new version is rolled out and rolled back at runtime — canary, blue-green, rolling, or shadow deployments, traffic shifting, progressive delivery with Argo Rollouts or Flagger, analysis/metric gates, automated abort, and whether rollback actually works. Covers mapping the rollout mechanism, tracing the promotion-and-rollback path, and finding risk such as canary-in-name-only (no metric analysis), untested or too-slow rollback, mutable tags that defeat rollback, and fail-open analysis. Gates any change to rollout or rollback behavior behind an Elephant-Goldfish design-note review and a non-production abort test. Trigger whenever the user mentions deployment strategy, rollout strategy, canary, blue-green, progressive delivery, Argo Rollouts, Flagger, traffic shifting, automated rollback, "does our rollback actually work", or "review our canary setup".
---

# Deployment Strategy Review

## Why this exists

The rollout is where a bad deploy is either contained or amplified. A canary that shifts 5% of traffic and aborts on rising errors turns a bad release into a non-event; a "rolling update" with no health gate turns it into an outage. And the rollback nobody ever tested is the one that fails during the incident, when it matters most.

Reading a `Rollout` manifest is not the same as knowing what actually shifts traffic, what metric gates each step, what triggers an automatic abort, and how fast — and whether the rollback has ever been exercised under load.

This skill builds an accurate model of the runtime rollout and rollback mechanism before recommending or making changes.

### Scope boundary

This skill owns the *runtime rollout and rollback mechanism*. It is deliberately bounded against two siblings so they do not overlap:

- `cicd-pipeline-review` owns the pipeline that *builds and delivers* the new version to the deploy step. This skill picks up at how that version is then rolled out to live traffic.
- `production-readiness-review` owns the pre-launch *gate* (is rollback tested, is a rollout plan defined). This skill audits the rollout mechanism itself, in place, whether or not a launch is pending.

The terminal output may be either:

- a safe, reviewed, low-risk rollout/rollback improvement
- a defensible risk/recommendation memo

---

## Operating principles

Start read-only.

Prefer:

- reading rollout/canary manifests and analysis templates
- reading traffic-routing config (service mesh, ingress, load balancer)
- reading recent rollout and abort history
- observing a rollout in a non-production environment
- dashboards and analysis-run results

Do not, without explicit authorization:

- trigger, promote, or abort a production rollout
- change traffic weights or routing in production
- change analysis thresholds that gate a production rollout
- change rollback or abort automation
- switch strategy (e.g. rolling to blue-green) on a live service

The only rollback worth trusting is one that has been tested. Treat an untested rollback as a broken rollback until proven otherwise. Confirm the strategy, the traffic-routing layer, and who owns rollout config before any write action.

---

## Phase 0 — Observe

Read:

- `Deployment`, `Rollout` (Argo), or `Canary` (Flagger) manifests
- analysis templates / metric provider config
- traffic-routing config (Istio, Linkerd, NGINX, ALB, Gateway API)
- deployment/rollback docs and runbooks
- recent rollout and abort history

Identify:

- the deployment controller and strategy in use (native RollingUpdate, Argo Rollouts, Flagger, Spinnaker, blue-green via LB/DNS, feature flags)
- whether rollout is stepped/weighted or all-at-once
- whether analysis/metric gates exist
- whether abort/rollback is automated or manual
- what "healthy" means during a rollout
- ownership of rollout config

Useful prompt:

> Summarize how a new version of this service is rolled out and rolled back, what gates each step, and what remains unclear.

Deliverable:

- One-paragraph rollout-mechanism summary.
- Strategy, traffic-routing, and analysis facts.
- Unknowns that require human confirmation.

Move on when:

- you know which strategy and controller are in use
- you know whether rollout is gated by metrics
- you know whether rollback is automated and where it is configured

---

## Phase 1 — Map the rollout mechanism

Inventory rollout resources and how traffic moves.

Useful commands:

```bash
# Argo Rollouts
kubectl argo rollouts list rollouts -A
kubectl argo rollouts get rollout <name> -n <ns>
kubectl get analysistemplates,clusteranalysistemplates -A
kubectl get analysisruns -A

# Flagger
kubectl get canaries -A
kubectl describe canary <name> -n <ns>

# Native
kubectl rollout history deploy/<name> -n <ns>

# Traffic routing
kubectl get virtualservices,destinationrules -A     # Istio
kubectl get httproutes,gateways -A                   # Gateway API
kubectl get ingress -A
```

Use only the commands appropriate to the estate and access level.

Map:

- each service's strategy and controller
- rollout steps and traffic weights
- the traffic-routing layer that enforces those weights
- stable vs canary/preview services
- analysis templates and their metric providers
- promotion gates (manual pause vs automated analysis)
- abort/rollback triggers and speed
- `scaleDownDelay` / preview retention
- which services have no strategy (all-at-once)

Useful prompts:

> Which services use canary/blue-green/rolling, and which just deploy all at once?

> For this rollout, what shifts traffic and what gates each step?

Deliverable:

- Rollout inventory by service and strategy.
- Traffic-routing map.
- Analysis/gate map.

Move on when:

- you know which services roll out progressively and which do not
- you know what actually shifts traffic
- you know where analysis and abort are configured

---

## Phase 2 — Understand the promotion and rollback path

Trace one rollout end to end: from a new version deployed, through each traffic step and gate, to either full promotion or automatic abort.

Follow:

- how the new ReplicaSet/version is created
- how traffic is shifted at each step and by what
- what analysis runs at each step and against which metrics
- what promotes to the next step (timer, manual, passing analysis)
- what triggers abort/rollback
- how fast rollback completes relative to the blast radius of each step
- what happens to in-flight requests during rollback
- whether the previous version is still warm to roll back to

Useful prompts:

> Trace a rollout of this service from 0% to 100%, including every traffic step, analysis gate, and abort condition.

> If the canary is bad, how is that detected, how fast does traffic return to stable, and what is the blast radius until then?

Deliverable:

- End-to-end rollout-and-rollback trace.
- Blast-radius note per step.
- Rollback-time estimate vs step blast radius.

Move on when:

- you know exactly how traffic progresses and reverts
- you know what detects a bad version and how fast it reacts
- you know whether rollback is fast enough to matter

---

## Phase 3 — Surface risk

Look for:

- canary-in-name-only: stepped weights with pauses but no metric analysis
- analysis that proves nothing: only a `/health` 200 check, not error rate or latency
- no automated abort — a bad canary just sits at its current weight
- untested rollback, or rollback that has never fired in anger
- rollback slower than the blast radius (100% step before analysis completes)
- mutable image tags (`:latest`) that make rollback non-deterministic
- single replica, so a canary weight is meaningless
- claimed traffic-based canary but only replica-ratio splitting (no mesh/ingress routing)
- `scaleDownDelay` too short, dropping in-flight requests on promotion
- blue-green with no smoke/analysis gate before cutover
- no rollback path for stateful components or schema changes
- feature flags with no kill switch or owner
- fully manual promotion with no SLO/error-budget gate
- analysis metric provider (e.g. Prometheus) unreachable → fail-open instead of fail-closed
- rollout ignoring PodDisruptionBudget or readiness during shifts
- no notification on abort, so failures are silent

Useful prompts:

> Where is this rollout strategy weaker than it looks, and where would a bad release still reach users?

> Does the analysis actually gate on user-facing signals, and does abort happen automatically and fast enough?

> What should not be changed without platform or SRE approval?

Deliverable:

- Findings list tagged by severity.
- Human-decision list.
- Safe-change candidates.

Move on when:

- gaps between the intended and actual safety of the rollout are explicit
- rollback trustworthiness is assessed, not assumed
- safe vs unsafe change candidates are separated

---

## Phase 4 — Operationalize validation

Establish how to prove the rollout and rollback work before changing anything.

Document:

- how to trigger a rollout in a non-production environment
- how to force a deliberately bad version and confirm automated abort fires
- how to measure rollback time
- how to confirm in-flight requests are not dropped on promotion/abort
- which dashboards and analysis runs to watch during a rollout
- how the analysis provider behaves when unreachable (fail-open vs fail-closed)
- who owns and approves rollout-config changes

Example rollback fire-drill (non-production only):

```bash
# deploy a known-bad image to the canary in a non-prod environment
kubectl argo rollouts get rollout <name> -n <ns> --watch
kubectl argo rollouts abort <name> -n <ns>      # confirm traffic returns to stable
# measure: time from bad-version detection to full stable restoration
```

Deliverable:

- Documented rollout/rollback validation loop.
- Rollback fire-drill result (time, in-flight behavior), or a plan to run one.

Move on when:

- you have seen (or have a concrete plan to see) an automated abort actually fire
- you know the real rollback time
- you know how analysis behaves when its provider is unavailable

---

## Phase 5 — Contribute or recommend

For any non-trivial change, write a design note before implementation.

Design note should include:

- what changes (strategy, steps, weights, analysis, abort, routing)
- why it changes
- services and traffic-routing layer affected
- blast radius (what a mistuned rollout could expose to users)
- how it will be validated in non-production, including an abort test
- rollback or recovery plan for the change itself
- human approvals needed

Goldfish comprehension test:

> You have no prior context except this design note and the referenced rollout/analysis manifests. What is changing about how this service rolls out or rolls back, what does it affect, and how would a bad version be caught?

Goldfish critic test:

> Assume the role of a skeptical release-engineering and reliability reviewer. What did I miss, how could this let a bad version reach users or make rollback slower/less reliable, and what is the rollback for this change?

Only after these pass:

1. validate in a non-production environment, including a forced-abort test
2. confirm the traffic-routing layer enforces the weights you expect
3. confirm analysis gates on user-facing signals and fails closed
4. get required approval
5. apply to production only if authorized and the abort test passed

Good first deployment-strategy contributions:

- add a real metric-based analysis (error rate/latency) to a canary that only checks `/health`
- tune `failureLimit`/consecutive-error thresholds to reduce false aborts (backed by data)
- add an automated abort to a canary that currently only pauses
- add or lengthen `scaleDownDelay` to protect in-flight requests
- pin image digests so rollback is deterministic
- document and rehearse the rollback procedure
- add abort notifications

Deliverable:

- Safe rollout/rollback PR or defensible recommendation memo.

---

## Phase 6 — Systematize

Update context so the next engineer does not repeat the archaeology.

Document:

- strategy and controller per service
- rollout steps and traffic-routing layer
- analysis metrics and thresholds
- abort/rollback behavior and measured rollback time
- fire-drill results
- known risks
- ownership and approvals
- open decisions

Deliverable:

- Updated `deployment-strategy.md` or rollout section in the service context doc.

---

## Two exits

A finding does not always warrant a fix by you.

Examples:

- switching strategy (rolling to canary/blue-green) affects every deploy and needs platform sign-off
- tightening analysis thresholds can cause false aborts and needs baseline data first
- blue-green needs capacity headroom for two full versions
- traffic-routing changes can affect production immediately
- rollback for stateful services may need data/restore design, not just a manifest change

A documented, escalated risk with a recommended remediation is a complete use of this skill, even with zero rollout config changed.

---

## Stop Conditions

Stop and ask a human before changing anything if:

- the change affects a production rollout, traffic weights, or routing
- the change touches analysis thresholds or abort automation that gate production
- rollback is unverified for the affected service
- you would need to trigger or abort a production rollout to test
- the analysis provider's fail-open/fail-closed behavior is unclear
- the service is stateful and rollback has no data story
- the Goldfish cannot explain the design note
- you are not explicitly authorized to modify rollout or routing config

Do not tune a production rollout blind; validate the change and its abort behavior in a non-production environment first.

---

## Final Deliverable

A deployment-strategy review package:

- rollout-mechanism summary
- rollout/strategy inventory and traffic-routing map
- promotion-and-rollback trace with per-step blast radius
- risk findings (analysis quality, abort, rollback trustworthiness)
- validation loop and rollback fire-drill result
- safe first change or recommendation
- updated rollout documentation
