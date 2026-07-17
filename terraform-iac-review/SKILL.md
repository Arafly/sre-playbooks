---
name: terraform-iac-review
description: Use this to safely understand, audit, or modify an unfamiliar Terraform/IaC estate — inherited infrastructure code, a module you did not write, or any "I need to touch this infrastructure code and do not fully trust it yet" situation. Covers mapping providers, modules, environments, state, drift, blast radius, and risky patterns such as hardcoded secrets, missing encryption, broad IAM, missing deletion protection, and unclear backends. Gates any non-trivial change behind an Elephant-Goldfish design-note review before running apply. Trigger whenever the user mentions reviewing, auditing, understanding, or safely changing Terraform, OpenTofu, CloudFormation, Pulumi, CDK, or other IaC.
---

# Terraform / IaC Review

## Why this exists

Infrastructure code is uniquely dangerous to misunderstand.

A bad `apply` can delete a database, weaken IAM, break networking, rotate secrets unexpectedly, or replace production infrastructure.

`terraform plan` is not a magic safety net if you do not understand the backend, workspace, provider scope, state, or blast radius.

This skill builds an accurate mental model of an IaC estate before recommending or making changes.

The terminal output may be either:

- a safe, reviewed, low-risk IaC change
- a defensible risk/recommendation memo

---

## Operating principles

Start read-only.

Prefer:

- `fmt`
- `validate`
- `plan`
- `show`
- state inspection
- module reading
- CI review
- cloud console read-only inspection, if available

Do not run `apply` from:

- an unclear backend
- an unknown workspace
- stale local state
- unreviewed credentials
- unclear target environment
- a plan you have not read fully

Confirm backend, workspace, credentials, and target environment before any write action.

---

## Phase 0 — Observe

Read:

- README
- `versions.tf`
- `providers.tf`
- `backend.tf`
- environment docs
- CI/CD workflows
- module READMEs
- recent plans/applies, if available

Identify:

- Terraform/OpenTofu version
- providers
- cloud accounts/subscriptions/projects
- regions
- environments/workspaces
- backend
- state locking
- plan/apply workflow
- whether this targets sandbox/local cloud or real infrastructure

Useful prompt:

> Summarize what this IaC configuration provisions, which environments it targets, and how state is managed.

Deliverable:

- One-paragraph estate summary.
- State/backend facts.
- Unknowns that require human confirmation.

Move on when:

- you know what this IaC claims to manage
- you know where state lives
- you know which environment/workspace you are looking at

---

## Phase 1 — Map providers, modules, and state

Inventory the estate.

Useful commands:

```bash
terraform version
terraform providers
terraform init -backend=false
terraform fmt -check -recursive
terraform validate
terraform state list
terraform show
git log --stat -- '*.tf' | head -50
```

Use only the commands appropriate to the repo and access level.

Map:

- root modules
- shared modules
- provider aliases
- environment structure
- workspaces
- remote state references
- module inputs/outputs
- resources in state
- resources in code
- resources that appear unmanaged

Useful prompts:

> Explain the structure of this Terraform repo.

> Which modules are root modules and which are shared modules?

> What resources are managed by each module?

> What exists in state that is not obvious from the code?

Deliverable:

- Module inventory.
- State/backend map.
- Code-vs-state reconciliation note.

Move on when:

- you know which modules own which resources
- you know how environments are separated
- you know whether code and state appear aligned

---

## Phase 2 — Understand dependencies and blast radius

For each important module, identify what it depends on and what depends on it.

Prioritize high-blast-radius areas:

- networking
- IAM
- databases
- Kubernetes clusters
- DNS
- load balancers
- secrets/KMS
- queues
- production compute
- observability pipelines
- CI/CD roles
- shared modules used across environments

Trace one high-risk resource end to end.

Example:

> If this security group changes, what workloads, databases, load balancers, or external integrations could be affected?

Useful prompts:

> Describe the relationship between `[module_A]` and `[module_B]`. How do they depend on each other?

> Which resources here have the highest blast radius?

> What could be destroyed or replaced accidentally?

Deliverable:

- Dependency map.
- Blast-radius note for the riskiest 2–3 resources/modules.

Move on when:

- you know the riskiest modules/resources
- you know which changes may affect multiple environments or services
- you know where human review is required

---

## Phase 3 — Surface risk

Look for:

- hardcoded secrets or credentials
- missing encryption at rest or in transit
- broad IAM policies
- overly open security groups
- resources without deletion protection in production
- missing state locking
- shared state across environments
- provider alias confusion
- implicit dependencies
- risky `count` or `for_each` changes
- module version drift
- resources in state but not code
- resources in code but not state
- missing variable validation
- missing lifecycle rules
- unsafe defaults
- manual resources not imported
- drift between code and real infrastructure
- unclear ownership
- missing plan/apply approvals

Useful tools, if available:

```bash
terraform fmt -check -recursive
terraform validate
tflint
checkov
tfsec
```

Useful prompts:

> What drift or blast-radius risks exist in this Terraform?

> Where are dependencies implicit rather than explicit?

> What should not be changed without human approval?

Deliverable:

- Findings list tagged by severity.
- Human-decision list.
- Safe-change candidates.

Move on when:

- high-risk findings are captured
- unknown ownership or state issues are explicit
- safe vs unsafe change candidates are separated

---

## Phase 4 — Operationalize validation

Establish the exact validation loop before changing anything.

Document:

- formatting command
- validation command
- lint/security scanning
- plan command
- plan review process
- apply process
- approval process
- CI workflow
- rollback/recovery path
- environment/workspace selection

Example validation loop:

```bash
terraform fmt -check -recursive
terraform validate
tflint
checkov -d .
terraform plan -out=tfplan
terraform show -no-color tfplan
```

Adjust to the repo's actual tooling.

Deliverable:

- Documented local and CI validation loop.

Move on when:

- you know how to produce and review a plan
- you know how approval/apply happens
- you know what rollback or recovery would look like

---

## Phase 5 — Contribute or recommend

For any non-trivial change, write a design note before implementation.

Design note should include:

- what changes
- why it changes
- modules/resources touched
- state/backend/workspace involved
- blast radius
- plan expectations
- validation steps
- rollback or recovery plan
- human approvals needed

Goldfish comprehension test:

> You have no prior context except this design note and referenced Terraform files. What is this change trying to accomplish, which resources does it affect, and what is the blast radius?

Goldfish critic test:

> Assume the role of a skeptical infrastructure reviewer. What did I miss, what could this break, what plan output would worry you, and what rollback or recovery path is required?

Only after these pass:

1. run `plan`
2. read the full diff, not only the summary
3. confirm workspace/backend/credentials
4. get required approval
5. apply only if you are authorized and the plan matches expectations

Good first IaC contributions:

- module README
- variable validation
- formatting/lint cleanup
- plan summary improvement
- documentation of state/backend
- non-invasive tags
- drift detection job
- CI validation improvement
- policy-as-code checks

Deliverable:

- Safe IaC PR or defensible recommendation memo.

---

## Phase 6 — Systematize

Update context so the next engineer does not repeat the archaeology.

Document:

- what the IaC manages
- environment layout
- backend/state model
- module ownership
- risky resources
- validation loop
- apply process
- known drift
- open decisions
- human approval requirements

If you found drift, decide or escalate whether to:

- import into code
- remove from state
- destroy orphaned resource
- document intentional manual management

Do not silently leave drift as a footnote.

Deliverable:

- Updated module README, `iac-review.md`, or infra section in service context doc.

---

## Two exits

A finding does not always warrant a fix by you.

Examples:

- missing encryption on a production database may require change window and stakeholder sign-off
- broad IAM may need owner review before tightening
- state split may need migration planning
- deletion protection may need staged rollout
- drift may need import strategy before any change

A documented, escalated risk with a recommended remediation is a complete use of this skill, even with zero code changed.

---

## Stop Conditions

Stop and ask a human before applying if:

- backend or workspace is unclear
- credentials or target environment are unclear
- plan includes replacement or destruction
- state appears stale, shared, or inconsistent
- resource ownership is unknown
- rollback or recovery path is unclear
- the change touches IAM, networking, databases, DNS, secrets, KMS, or production compute
- the change affects shared modules used by multiple services/environments
- the Goldfish cannot explain the design note
- plan output differs from expectation
- you are not explicitly authorized to apply

---

## Final Deliverable

An IaC review package:

- estate summary
- module inventory
- backend/state map
- dependency and blast-radius map
- drift/risk findings
- validation loop
- safe first change or recommendation
- updated documentation
