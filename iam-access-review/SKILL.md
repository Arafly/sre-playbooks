---
name: iam-access-review
description: Use this to understand, review, audit, or safely reduce identity and access permissions across cloud accounts, Kubernetes RBAC, internal systems, CI/CD, databases, and third-party SaaS. Covers human and machine identities, privilege sprawl, stale accounts, overly broad roles, shared credentials, cross-account trust, break-glass access, and safely revoking or right-sizing permissions without silently breaking production. Trigger whenever the user mentions access reviews, permission audits, least-privilege cleanup, offboarding access, privileged-access recertification, "who has access to X", "who can touch production", "our IAM is a mess", or asks whether a role, account, group, or service identity still needs its permissions.
---

# IAM / Access-Control Review

## Why this exists

Access control decays by default. Permissions accumulate, owners change, and machine identities often outlive the context that justified them. Revocation also has blast radius, so access reduction must be evidence-based and reversible.

The terminal output may be either:

- a safely revoked or right-sized permission
- a defensible access-risk finding with an owner and remediation path

## Operating principles

- Start read-only.
- Review effective access, not only direct grants.
- Include human and machine identities.
- Treat emergency, recovery, CI/CD, and infrequently used operational access as potentially load-bearing.
- Least privilege means the minimum permission, scope, and duration needed for the legitimate task.

## Phase 0 — Observe

Review access policy, joiner/mover/leaver process, privileged-access workflow, approvals, role catalogue, review cadence, offboarding, break-glass, evidence retention, and previous findings.

Inventory SSO, cloud IAM, Kubernetes RBAC, source control, CI/CD, databases, secrets managers, internal tools, SaaS, VPN/zero-trust, and customer-data systems.

Deliverable:

- access-governance summary
- identity-system inventory
- policy and ownership gaps

## Phase 1 — Map identities, roles, and grants

Prioritize identities that can affect production, customer data, billing, IAM, security controls, secrets, CI/CD, databases, backups, DNS, Kubernetes, encryption keys, or incident response.

Useful commands may include:

```bash
aws iam get-account-authorization-details
aws accessanalyzer list-findings
kubectl get rolebindings,clusterrolebindings -A -o wide
kubectl get roles,clusterroles -A
kubectl get serviceaccounts -A
kubectl auth can-i --list -n <namespace>
```

Deliverable:

- privileged-identity inventory
- effective-access map
- high-blast-radius grants

## Phase 2 — Understand why each grant exists

Determine the identity, purpose, approver, grant date, permanence, last use, narrower alternatives, dependencies, and emergency use.

“Nobody knows why this service account has admin” is a finding, not a dead end.

Deliverable:

- annotated privileged-grant inventory with justification status

## Phase 3 — Surface privilege sprawl and access risk

Look for stale accounts, role-change residue, unused or wildcard grants, standing production access, shared credentials, missing MFA, long-lived keys, broad trust, unrestricted role assumption, orphaned service identities, cluster-admin, excessive CI/CD access, SSO bypass, weak break-glass, self-escalation, and missing recertification.

Classify misuse blast radius and revocation blast radius separately.

Deliverable:

- access-risk register
- stale/orphaned identity list
- candidate right-sizing actions
- human-decision list

## Phase 4 — Operationalize the review loop

Define access requests, ownership, joiner/mover/leaver controls, service-account ownership, expiry, just-in-time access, break-glass controls, recurring recertification, evidence retention, privileged-use alerts, and exceptions.

Deliverable:

- repeatable access-review and recertification process
- method for answering “who can do X?” on demand

## Phase 5 — Revoke or right-size safely

Before changing access:

1. Write an access-change note.
2. Confirm the owner.
3. Inspect recent and historical usage.
4. Trace dependencies.
5. Define restoration.
6. Consider staged reduction.
7. Goldfish-test the reasoning.
8. Obtain approval.
9. Change one grant.
10. Monitor for unexpected failures.

Goldfish prompt:

> Does the evidence justify this access change? What infrequent, automated, emergency, inherited, or recovery use might have been missed, and is the restoration plan credible?

Deliverable:

- one safe revocation or right-sizing action
- or a defensible recommendation not to revoke yet

## Phase 6 — Systematize

Document identity providers, privileged roles, ownership, approvals, machine identities, trust, break-glass, offboarding, recertification, evidence, exceptions, restoration, risks, and escalation contacts.

## Two exits

### Safe action

Examples include removing a departed user, expiring temporary access, replacing administrator access with a scoped role, rotating a shared credential, or converting standing access to just-in-time.

### Defensible judgment

Use when operational dependency, recovery use, ownership, authority, or evidence is unclear.

## Stop Conditions

Stop before changing production automation, CI/CD, backup/failover/recovery, customer data/payments/regulatory roles, IAM/KMS/secrets/DNS/database/security controls, cross-account/federated access, last-admin or break-glass paths, or anything with unclear usage, restoration, ownership, or authority.

## Final Deliverable

An IAM review package containing the identity inventory, privileged-access map, inheritance paths, grant justifications, stale/sprawl findings, revocation-risk assessment, one safe action or defensible recommendation, and an updated review process.
