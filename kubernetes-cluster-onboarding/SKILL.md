---
name: kubernetes-cluster-onboarding
description: Use this when encountering an unfamiliar Kubernetes cluster, namespace, or workload — a new employer's cluster, a cluster inherited from another team, or debugging in a cluster you have never worked in. Maps namespaces, workloads, ingress/service-mesh traffic flow, RBAC, network policies, secrets/config, GitOps ownership, and runtime risk before touching anything. Flags missing probes, missing resource requests/limits, missing PodDisruptionBudgets, unpinned images, broad RBAC, plaintext secrets, and unsafe manual changes. Trigger whenever the user says things like "I need to understand this cluster", "what's running in this namespace", "why does this ingress route here", "I'm taking over this Kubernetes workload", or asks for help getting oriented in a Kubernetes environment they did not build.
---

# Kubernetes Cluster Onboarding

## Why this exists

A Kubernetes cluster you did not build is full of implicit decisions:

- namespace boundaries
- RBAC grants
- ingress routing
- service mesh behavior
- GitOps ownership
- resource sizing
- rollout strategy
- disruption tolerance
- secret/config handling
- runtime assumptions

Many of those decisions are invisible until something breaks.

This skill builds an accurate map before you `kubectl apply` anything.

---

## Operating principles

Start read-only.

Prefer:

- `get`
- `describe`
- `logs`
- `top`
- events
- GitOps source inspection
- Helm/Kustomize manifests
- dashboards

Before:

- `edit`
- `patch`
- `delete`
- `apply`
- scaling manually
- restarting workloads
- changing ingress/RBAC/secrets/network policies

If the cluster is GitOps-managed, live `kubectl` changes are emergency-only unless the team explicitly works that way.

---

## Phase 0 — Observe

Identify the cluster and its management model.

Useful commands:

```bash
kubectl config current-context
kubectl cluster-info
kubectl version --short
kubectl get nodes -o wide
kubectl get namespaces
kubectl api-resources
kubectl get crds
```

Look for:

- cluster purpose
- environment
- Kubernetes version
- node pools
- ingress controller
- service mesh
- GitOps tooling
- Helm releases
- operators/controllers
- monitoring/logging stack
- docs or runbooks

Useful prompt:

> Summarize what this Kubernetes cluster appears to run, which tooling manages it, and what remains unclear.

Deliverable:

- One-paragraph cluster summary:
  - context
  - version
  - node shape
  - ingress/mesh
  - GitOps tooling
  - monitoring/logging
  - unknowns

Move on when:

- you are sure you are in the intended context
- you know whether the cluster is GitOps-managed
- you know the main cluster-level controllers

---

## Phase 1 — Map namespaces and workloads

Inventory workloads.

Useful commands:

```bash
kubectl get deploy,sts,ds,svc,ingress -A
kubectl get pods -A -o wide
kubectl get jobs,cronjobs -A
kubectl get hpa,pdb -A
kubectl get configmaps,secrets -A
kubectl get helmreleases -A
kubectl get applications -A
```

Use only commands supported by the cluster.

For each important namespace, identify:

- purpose
- owner
- workloads
- services
- ingress routes
- jobs/cronjobs
- stateful workloads
- secrets/config
- GitOps source
- criticality

Useful prompt:

> Based on these resources, group namespaces by likely purpose and criticality. What should I inspect first?

Deliverable:

- Namespace/workload inventory flagged by criticality.

Move on when:

- you know which namespaces are production-critical
- you know which workloads are stateful
- you know which namespaces/workloads need owner confirmation before changes

---

## Phase 2 — Understand traffic and data flow

Trace one critical request end to end.

Follow:

- DNS
- load balancer
- ingress/gateway
- service mesh, if present
- Kubernetes service
- pod
- sidecars
- database/queue/cache
- external API
- egress path

Useful commands:

```bash
kubectl get ingress -A -o wide
kubectl describe ingress -n <namespace> <name>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get networkpolicy -A
kubectl describe pod -n <namespace> <pod>
```

Useful prompt:

> Trace how a request flows through this system when `[feature]` is used, including every Kubernetes hop and external dependency.

Deliverable:

- Written input-to-output trace for the 1–2 most critical workloads.
- Traffic-flow sketch.

Move on when:

- you know how traffic reaches the workload
- you know how pods connect to dependencies
- you know whether network policies or service mesh rules affect the path

---

## Phase 3 — Surface risk

Check production-critical workloads for:

- missing liveness/readiness/startup probes
- missing resource requests/limits
- missing PodDisruptionBudgets
- `:latest` or unpinned image tags
- containers running as root
- privileged containers
- hostPath mounts
- broad RBAC
- plaintext secrets in ConfigMaps
- missing NetworkPolicies
- stateful workloads without backup/restore clarity
- fragile node affinity/tolerations
- missing HPA or unsafe HPA behavior
- manual drift from GitOps
- CrashLoopBackOff / high restart counts
- pending pods
- failing jobs
- noisy events

Useful commands:

```bash
kubectl get events -A --sort-by=.lastTimestamp
kubectl get pods -A --field-selector=status.phase!=Running
kubectl get rolebindings,clusterrolebindings -A -o wide
kubectl auth can-i --list -n <namespace>
kubectl top pods -A
kubectl top nodes
```

Useful prompts:

> Which workloads here look under-protected, over-privileged, or operationally risky?

> Which missing probes, resources, PDBs, or RBAC boundaries should be treated as high priority?

Deliverable:

- Findings list per namespace, tagged by severity.
- Human-decision list for risky fixes.

Move on when:

- risky workloads are identified
- unsafe manual-change areas are clear
- you know what requires owner/security/platform review

---

## Phase 4 — Operationalize debugging

Build the runtime debugging toolkit.

Confirm how to:

- view logs
- inspect events
- exec into pods, if allowed
- check rollout status
- inspect probes
- inspect resource pressure
- inspect ingress/service path
- inspect GitOps sync status
- access dashboards/traces/logs
- rollback via the actual deployment mechanism

Useful commands:

```bash
kubectl logs -n <namespace> <pod> --tail=100
kubectl logs -n <namespace> deploy/<deployment> --all-containers=true --tail=100
kubectl describe pod -n <namespace> <pod>
kubectl rollout status deploy/<deployment> -n <namespace>
kubectl rollout history deploy/<deployment> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Useful prompt:

> If this workload is unhealthy, what should the on-call check first, and what commands should they run?

Deliverable:

- Short troubleshooting runbook:
  - symptoms
  - commands
  - dashboards
  - escalation points
  - rollback path

Move on when:

- you can debug a pod/workload failure
- you know how deployment and rollback happen
- you know what observability exists and what is missing

---

## Phase 5 — Contribute or recommend

Pick one low-risk fix from Phase 3.

Good first Kubernetes contributions:

- missing readiness/liveness probe
- missing resource request/limit
- missing PDB for stateless workload
- documentation of namespace ownership
- runbook improvement
- dashboard/log query improvement
- fixing stale manifests
- adding safer rollout notes

Avoid without explicit owner/platform approval:

- ingress changes
- RBAC tightening
- NetworkPolicy changes
- PersistentVolume/PVC changes
- stateful workload changes
- secret changes
- service mesh routing changes
- node selector/affinity/toleration changes
- HPA/concurrency changes
- live manual `kubectl apply` in GitOps-managed clusters

If non-trivial, write a design note and Goldfish-test it.

Goldfish prompt:

> You have no prior context except this design note and referenced Kubernetes manifests/resources. What change is proposed, what workloads or routes could it affect, what risks are missing, and could you review it safely from this note alone?

Deliverable:

- Small reconciled, verified change.
- Or a defensible recommendation/escalation.

Move on when:

- the change path is clear
- GitOps or deployment ownership is respected
- verification includes actual cluster reconciliation, not only PR merge

---

## Phase 6 — Systematize

Document what you learned.

Include:

- namespace map
- workload inventory
- ingress/traffic path
- RBAC model
- GitOps ownership
- critical workloads
- stateful workloads
- known risks
- debugging commands
- dashboards/log queries
- gotchas
- safe first changes
- escalation paths

Deliverable:

- Cluster/namespace context doc or update to service context doc.

---

## Two exits

Not every finding should be fixed by you immediately.

Examples:

- broad RBAC may be intentional or relied on by legacy automation
- missing NetworkPolicy may require platform-wide design
- tightening security can silently break traffic
- stateful workload changes need backup/restore confidence
- ingress changes can affect production traffic immediately

A documented, specific recommendation can be the correct output.

---

## Stop Conditions

Stop and ask a human before changing anything if:

- the namespace is production-critical
- ownership is unclear
- the workload is stateful
- GitOps ownership is unclear
- the change touches ingress, RBAC, secrets, NetworkPolicy, PV/PVC, service mesh, or node scheduling
- rollback path is unclear
- the change would be applied manually to a GitOps-managed cluster
- the Goldfish cannot explain the design note
- you are not sure you are in the correct kube context

---

## Final Deliverable

A Kubernetes onboarding package:

- cluster summary
- namespace/workload inventory
- traffic-flow trace
- risk findings
- debugging runbook
- safe change or recommendation
- updated context documentation
