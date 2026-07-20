---
name: secrets-management-review
description: Use this to safely understand, audit, or improve how secrets are stored, injected, scoped, rotated, and leaked across an unfamiliar estate — cloud secret managers, Vault, KMS, Kubernetes Secrets, CI/CD secrets, SOPS/Sealed Secrets/External Secrets, .env files, and secrets hiding in code or git history. Covers mapping secret backends and consumers, finding plaintext secrets, long-lived static credentials, over-broad read access, missing rotation, and secrets in logs or state; and gating any rotation, revocation, or migration behind an Elephant-Goldfish design-note review. Trigger whenever the user mentions secrets management, credential rotation, leaked secrets, KMS, Vault, secret sprawl, "are our secrets safe", "did we commit a secret", or "audit how we handle credentials".
---

# Secrets Management Review

## Why this exists

Secrets are the highest-value target in any estate and the most pervasive cross-cutting risk.

Almost every other review brushes against them — IaC flags "secrets/KMS", Kubernetes flags "plaintext secrets", CI/CD flags "leaked credentials", and nearly every skill's Stop Conditions say "touches secrets" — but none of them owns the secret estate itself: where secret material lives, who and what can read it, how it rotates, and where it leaks.

A misunderstood secret estate is how a single leaked credential becomes a breach.

This skill builds an accurate model of the secret estate before recommending or making changes.

The terminal output may be either:

- a safe, reviewed, low-risk secrets improvement
- a defensible risk/recommendation memo

---

## Operating principles

Start read-only, and never expose secret values.

Prefer:

- reading secret *references*, names, and metadata — not values
- reading access policies and audit logs
- reading how applications consume secrets
- read-only secret-scanning tools
- provider consoles in read-only mode, if available

Do not, without explicit authorization:

- print, echo, copy, or paste a secret value anywhere (logs, chat, files, commits)
- rotate, revoke, or delete a secret
- change KMS key policies or grants
- move secrets between backends
- broaden or narrow read access

A secret is not "unused" just because you cannot see who reads it. Rotating or revoking a live secret can cause an outage as surely as leaking it can cause a breach. If you discover an exposed secret, treat it as an incident: it must be rotated by its owner, not merely deleted from the file — the exposed value is already compromised.

Confirm the backend, the consumers, and the owner before any write action.

---

## Phase 0 — Observe

Read:

- secrets/security docs
- secret-manager configuration
- Kubernetes Secret and ConfigMap usage
- CI/CD secret configuration
- IaC that provisions secrets, KMS, or grants
- application config and startup code that loads secrets
- `.env`, `.env.example`, and ignored-file patterns

Identify:

- which secret backends exist (Vault, AWS/GCP/Azure secret managers, KMS, SOPS, Sealed Secrets, External Secrets Operator, CI platform secrets, plain files)
- how applications receive secrets (env vars, mounted files, CSI driver, init container, SDK call)
- encryption at rest and in transit
- rotation policy, if any
- ownership

Useful prompt:

> Summarize how this system stores, injects, and rotates secrets, and what remains unclear.

Deliverable:

- One-paragraph secret-estate summary.
- Backend and injection-model facts.
- Unknowns that require human confirmation.

Move on when:

- you know which backends hold secret material
- you know how applications consume secrets
- you know whether anything is stored in plaintext

---

## Phase 1 — Map the secret estate

Inventory secret stores and consumers — by reference, never by value.

Useful commands:

```bash
# Kubernetes: names/types only, never -o yaml on Secret values
kubectl get secrets -A
kubectl get externalsecrets,sealedsecrets -A

# Secret managers, read-only listing (no values)
aws secretsmanager list-secrets --query 'SecretList[].Name'
vault kv list -R secret/

# Read-only leak scanning of the repo and its history
gitleaks detect --no-banner
trufflehog git file://. --only-verified
```

Use only the commands appropriate to the estate and access level, and never dump values.

Map:

- each secret store and what it holds (by name/purpose)
- consumers of each high-value secret
- injection mechanism per consumer
- which secrets are encrypted at rest and with which key
- who and what can read each secret
- environment separation (are prod and non-prod secrets isolated)

Useful prompts:

> Group these secrets by purpose and sensitivity. Which are the highest value?

> For this secret, which workloads or pipelines consume it, and how?

Deliverable:

- Secret inventory (names/purpose, not values).
- Consumer map for high-value secrets.
- Backend/encryption map.

Move on when:

- you know where the highest-value secrets live
- you know who and what can read them
- you know whether environments are isolated

---

## Phase 2 — Understand access and blast radius

Prioritize the highest-value secret material:

- database and datastore credentials
- signing keys and private keys
- cloud provider credentials
- third-party API keys with spend or data access
- TLS private keys
- encryption/KMS keys
- admin or root tokens

For each, trace who and what can read it, where it flows, and what an attacker could do with it.

Useful prompts:

> If this secret leaked, what could an attacker access or do?

> Which identities and workloads can read this secret, and do they all need to?

Deliverable:

- Blast-radius note for the riskiest 3–5 secrets.
- Over-broad access findings.

Move on when:

- you know which secrets have the largest blast radius
- you know which grants look broader than needed
- you know where human review is required

---

## Phase 3 — Surface risk

Look for:

- secrets committed to git history
- plaintext secrets in ConfigMaps, env vars, or CI logs
- secrets baked into container images
- secrets hardcoded in application code
- secrets in Terraform/IaC state or plan output
- long-lived static credentials instead of short-lived/dynamic
- no rotation, or rotation that has never been exercised
- shared secrets across environments or teams
- over-broad read access or wildcard policies
- missing encryption at rest
- weak or over-permissive KMS key policies
- no audit logging on secret access
- no break-glass procedure
- secrets passed on command lines (visible in process lists)
- `.env` files not ignored, or committed `.env`
- expired or near-expiry certificates and keys

Useful tools, if available:

```bash
gitleaks detect
trufflehog git file://.
detect-secrets scan
```

Useful prompts:

> Where does secret material leak or persist in plaintext across this estate?

> Which credentials are long-lived and should be short-lived or dynamic?

> What should not be rotated or revoked without owner coordination?

Deliverable:

- Findings list tagged by severity.
- Human-decision list.
- Safe-change candidates.

Move on when:

- exposure and leakage findings are captured
- long-lived and over-broad credentials are explicit
- safe vs unsafe change candidates are separated

---

## Phase 4 — Operationalize safe handling

Establish the safe-change loop before touching any secret.

Document:

- how a secret is rotated for each backend
- how consumers pick up a rotated secret (restart, reload, TTL)
- how to rotate without breaking consumers (dual-secret / overlap window)
- how secret access is audited
- the break-glass procedure
- how secret scanning runs in CI and pre-commit
- who owns and approves rotation/revocation

Example safe controls:

```bash
# Pre-commit and CI leak prevention
detect-secrets scan > .secrets.baseline
gitleaks detect --redact
```

Deliverable:

- Documented rotation, audit, and leak-prevention loop.

Move on when:

- you know how to rotate safely for each backend
- you know how consumers pick up new values
- you know how leaks are prevented going forward

---

## Phase 5 — Contribute or recommend

For any non-trivial change, write a design note before implementation.

Design note should include:

- what changes
- why it changes
- secrets, backends, and consumers involved
- blast radius (what breaks if a consumer misses the new value)
- rotation/overlap strategy
- how it will be validated without exposing values
- rollback or recovery plan
- human approvals needed

Goldfish comprehension test:

> You have no prior context except this design note and referenced config. What secret is changing, which consumers depend on it, and what breaks if the rotation is mistimed?

Goldfish critic test:

> Assume the role of a skeptical security and reliability reviewer. What did I miss, how could this leak a value or break a consumer, and what is the rollback?

Only after these pass:

1. confirm every consumer of the secret
2. use an overlap/dual-secret window where consumers cannot reload instantly
3. never print the value; validate via reference and behavior
4. get required approval
5. rotate/apply only if authorized

Good first secrets contributions:

- add secret scanning to CI and pre-commit
- add or fix `.gitignore`/`.dockerignore` for secret files
- move one plaintext secret into the secret manager
- scope down one over-broad read policy (with owner sign-off)
- document the rotation procedure for one backend
- replace one long-lived credential with a short-lived/OIDC equivalent

If you found a leaked secret, the contribution is to open an incident and get it rotated by the owner — not to quietly delete it.

Deliverable:

- Safe secrets PR or defensible recommendation memo.

---

## Phase 6 — Systematize

Update context so the next engineer does not repeat the archaeology.

Document:

- secret backends and what each holds
- injection model per consumer
- encryption and KMS key model
- rotation procedures and cadence
- access and audit model
- break-glass procedure
- known risks
- ownership and approvals
- open decisions

Deliverable:

- Updated `secrets-review.md` or secrets section in the service context doc.

---

## Two exits

A finding does not always warrant a fix by you.

Examples:

- rotating a shared production credential needs coordinated consumer updates
- revoking a credential may break unknown automation
- moving a secret between backends needs a migration plan
- narrowing a read policy may break a job that silently relied on it
- KMS key-policy changes need security-team review

A documented, escalated risk with a recommended remediation is a complete use of this skill, even with zero secrets changed.

---

## Stop Conditions

Stop and ask a human before changing anything if:

- you would need to read or print a secret value to proceed
- the secret is in active use by consumers you cannot fully enumerate
- rotation could break production and the overlap strategy is unclear
- the change touches KMS key policies, signing keys, or root credentials
- the change broadens or narrows access to production secrets
- a leaked secret requires incident handling and owner-driven rotation
- rollback is unclear
- the Goldfish cannot explain the design note
- you are not explicitly authorized to modify secrets

Never resolve uncertainty by printing or exfiltrating a secret value.

---

## Final Deliverable

A secrets review package:

- secret-estate summary
- inventory and consumer map (references, not values)
- blast-radius and access findings
- leakage/plaintext findings
- rotation, audit, and leak-prevention loop
- safe first change or recommendation
- updated secrets documentation
