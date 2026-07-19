---
name: cicd-pipeline-review
description: Use this to safely understand, audit, or modify an unfamiliar CI/CD pipeline estate — inherited GitHub Actions, GitLab CI, Jenkins, CircleCI, Azure Pipelines, Buildkite, Argo Workflows, or Tekton that you did not write and do not fully trust yet. Covers mapping triggers, stages, runners, environments, and promotion flow; finding supply-chain and delivery risk such as leaked secrets, unpinned third-party actions, over-broad OIDC/deploy permissions, missing approval gates, fork-PR secret exposure, and no rollback path; and gating any non-trivial pipeline change behind an Elephant-Goldfish design-note review before it runs on a protected branch. Trigger whenever the user mentions reviewing, auditing, understanding, hardening, speeding up, or safely changing CI/CD, build pipelines, deployment workflows, GitHub Actions, GitLab CI, or Jenkins, or says things like "I inherited these pipelines", "audit our CI/CD", "why is our pipeline unsafe or slow", or "who can actually deploy to prod".
---

# CI/CD Pipeline Review

## Why this exists

A CI/CD pipeline you did not build is a production system in disguise.

Anything that can deploy to production *is* production. A pipeline holds deploy credentials, runs third-party code, and pushes artifacts straight to your users — so a misunderstood pipeline is both a reliability risk and a supply-chain attack surface.

Reading a workflow file is not the same as understanding what actually reaches production, who can trigger it, what secrets it can see, and what happens when a step fails.

This skill builds an accurate mental model of a delivery estate before recommending or making changes.

The terminal output may be either:

- a safe, reviewed, low-risk pipeline change
- a defensible risk/recommendation memo

---

## Operating principles

Start read-only.

Prefer:

- reading pipeline config in the repo
- reading run history and logs
- reading environment/branch protection settings
- workflow linting and static analysis
- testing in a fork or throwaway branch
- dashboards and audit logs, if available

Do not, without explicit authorization:

- push to a protected or deploy-triggering branch
- trigger a production deployment
- edit, add, or print secrets
- change OIDC trust, deploy roles, or runner configuration
- merge a pipeline change that executes on a protected branch
- re-run a failed production job to "see what happens"

Treat the pipeline as a supply-chain surface: an untrusted change to a workflow that runs with secrets can exfiltrate them. Confirm platform, trigger model, secret scope, and who is authorized to deploy before any write action.

---

## Phase 0 — Observe

Read:

- pipeline config (`.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`, `azure-pipelines.yml`, `.buildkite/`, Tekton/Argo manifests)
- CI/CD and release docs
- README badges
- environment and branch-protection settings
- recent pipeline runs
- deployment/rollback docs

Identify:

- CI/CD platform(s)
- runner model (hosted vs self-hosted)
- trigger types (push, PR, tag, schedule, manual, `workflow_run`, `pull_request_target`)
- environments (dev/staging/prod)
- what actually deploys, and to where
- secret backend (platform secrets, OIDC, Vault, cloud secret manager)
- promotion model (auto, gated, manual)
- whether this deploys to real production

Useful prompt:

> Summarize what these pipelines build, test, and deploy; which environments they target; how they authenticate to deploy; and what remains unclear.

Deliverable:

- One-paragraph delivery-estate summary.
- Secret/auth model facts.
- Unknowns that require human confirmation.

Move on when:

- you know which platform(s) run delivery
- you know what reaches production and how
- you know whether you are looking at real production pipelines

---

## Phase 1 — Map pipelines, triggers, stages, and runners

Inventory the estate.

Useful commands:

```bash
# Enumerate pipeline definitions
find . -path '*/.github/workflows/*.y*ml' -o -name '.gitlab-ci.yml' -o -name 'Jenkinsfile'

# GitHub, read-only, if gh is authorized
gh workflow list
gh run list --limit 20
gh api repos/{owner}/{repo}/branches/{branch}/protection

# History of who changed the pipelines
git log --stat -- '.github/workflows/' '.gitlab-ci.yml' 'Jenkinsfile' | head -60
```

Use only the commands appropriate to the repo and access level.

Map:

- each pipeline and its purpose
- triggers per pipeline
- stages/jobs and their order
- job dependencies and fan-out
- runners/executors (hosted vs self-hosted, labels)
- reusable/shared/templated workflows
- caches and artifacts
- concurrency groups
- environments and required reviewers
- which jobs hold secrets or deploy credentials

Useful prompts:

> Explain the structure of these pipelines: triggers, stages, and job dependencies.

> Which jobs can deploy, and what triggers them?

> Which pipelines run on protected branches or with production secrets?

Deliverable:

- Pipeline inventory.
- Trigger/stage/runner map.
- List of jobs that can deploy or read secrets.

Move on when:

- you know which pipelines exist and what triggers them
- you know which jobs touch production or secrets
- you know the runner model

---

## Phase 2 — Understand the delivery path and blast radius

Trace one commit from push to production, end to end.

Follow:

- trigger
- build
- test/quality gates
- artifact creation and where it is stored
- image/artifact tagging (immutable vs mutable)
- environment promotion (dev → staging → prod)
- approval/gate steps
- deploy mechanism (kubectl, Helm, Argo, Terraform, script)
- rollback path
- notifications

Then ask the blast-radius questions:

> Who or what can trigger a production deploy?

> If this pipeline were compromised or misconfigured, what could it deploy, delete, or leak?

> What stops a bad build from reaching production automatically?

Useful prompt:

> Trace how a commit to the main branch flows to production through these pipelines, including every gate, credential, and rollback step.

Deliverable:

- End-to-end commit-to-production trace.
- Blast-radius note for the deploy path.
- List of gates that stand between a commit and production.

Move on when:

- you know exactly what reaches production and how
- you know who/what can trigger a prod deploy
- you know whether a bad change is stopped before prod

---

## Phase 3 — Surface risk

Look for:

- hardcoded secrets or credentials in config
- secrets printed to logs or passed on command lines
- unpinned third-party actions/orbs/images (mutable tags or branches instead of a pinned SHA/digest)
- overly broad token scope (`permissions: write-all`, default broad `GITHUB_TOKEN`)
- long-lived static cloud credentials instead of short-lived OIDC
- over-broad OIDC trust or deploy-role permissions
- `pull_request_target` or untrusted input used unsafely (script injection, checkout of PR code with secrets)
- fork PRs able to read secrets or run privileged jobs
- self-hosted runners exposed to untrusted (fork) workloads
- missing required reviewers / no production approval gate
- weak or missing branch protection on deploy branches
- missing environment protection rules
- no rollback path in the pipeline
- mutable image tags (`:latest`) deployed to prod
- no artifact provenance, SBOM, or signing
- `curl | bash` or unpinned tool installs in privileged jobs
- shared credentials across environments
- manual steps that bypass the pipeline
- disabled, skipped, or non-blocking tests used as gates
- missing concurrency control (racing deploys)
- flaky or very slow pipelines, missing caching

Useful tools, if available:

```bash
actionlint                 # GitHub Actions linting
zizmor .                   # GitHub Actions security static analysis
gitlab-ci-lint             # GitLab CI validation
```

Useful prompts:

> What supply-chain and delivery risks exist in these pipelines?

> Where could an untrusted contributor or a compromised dependency gain secrets or deploy access?

> What should not be changed without security or platform approval?

Deliverable:

- Findings list tagged by severity.
- Human-decision list.
- Safe-change candidates.

Move on when:

- high-risk findings are captured
- secret-exposure and deploy-access risks are explicit
- safe vs unsafe change candidates are separated

---

## Phase 4 — Operationalize validation

Establish how to change a pipeline safely before changing anything.

Document:

- how to lint/validate pipeline config locally
- how to test a change without touching production (fork, branch, `workflow_dispatch` in a sandbox, `act` for local Actions)
- how to read run logs and re-run safely
- how deploys actually happen
- how rollback actually happens
- which changes require security/platform approval
- how secrets are injected and scoped

Example validation loop:

```bash
actionlint
zizmor .
# test on a branch/fork that cannot deploy to prod, then watch the run
gh run watch
```

Adjust to the estate's actual platform and tooling.

Deliverable:

- Documented pipeline validation and safe-test loop.

Move on when:

- you know how to validate a pipeline change off the production path
- you know how deploy and rollback happen
- you know which changes need approval

---

## Phase 5 — Contribute or recommend

For any non-trivial change, write a design note before implementation.

Design note should include:

- what changes
- why it changes
- pipelines/jobs/triggers touched
- secrets, tokens, or environments involved
- blast radius (what it could deploy or expose)
- how it will be tested off the production path
- rollback or recovery plan
- human approvals needed

Goldfish comprehension test:

> You have no prior context except this design note and the referenced pipeline files. What is this change trying to accomplish, which jobs and triggers does it affect, and what could it deploy or expose?

Goldfish critic test:

> Assume the role of a skeptical release-engineering and security reviewer. What did I miss, how could this leak a secret or reach production unexpectedly, and what rollback path is required?

Only after these pass:

1. test on a branch or fork that cannot deploy to production
2. watch a real run and confirm no secret exposure and expected behavior
3. confirm token scope, OIDC trust, and environment protection are unchanged unless that is the point of the change
4. get required approval
5. merge only if you are authorized and the change runs safely on protected branches

Good first CI/CD contributions:

- pin a third-party action/image to a SHA/digest
- scope down `permissions:` on a workflow
- add a missing concurrency group
- make an existing status check required
- add caching to a slow job
- add `actionlint`/`zizmor` (or platform equivalent) as a CI gate
- document the promotion and rollback flow
- fix a flaky non-deploy step

Deliverable:

- Safe pipeline PR or defensible recommendation memo.

---

## Phase 6 — Systematize

Update context so the next engineer does not repeat the archaeology.

Document:

- what each pipeline does
- trigger and promotion model
- environments and gates
- secret/auth model (including OIDC trust)
- runner model
- deploy and rollback mechanism
- known risks
- ownership and approvals
- open decisions

Deliverable:

- Updated `cicd-review.md`, pipeline README, or delivery section in the service context doc.

---

## Two exits

A finding does not always warrant a fix by you.

Examples:

- broad OIDC trust or deploy-role scope may need cloud/security-team sign-off
- removing a manual approval may be removing a compliance control
- tightening `permissions:` can break a job that silently relied on the broader scope
- self-hosted runner changes need platform/security review
- changing tag/release automation needs release-owner buy-in

A documented, escalated risk with a recommended remediation is a complete use of this skill, even with zero pipeline changed.

---

## Stop Conditions

Stop and ask a human before changing anything if:

- the change runs on a protected or deploy-triggering branch
- the change touches secrets, tokens, OIDC trust, or deploy roles
- the change touches required approvals, branch protection, or environment protection
- the change touches self-hosted runner configuration
- the change touches release, tag, or versioning automation
- the change could expose secrets to fork PRs or untrusted input
- rollback is unclear
- the Goldfish cannot explain the design note
- you are not explicitly authorized to modify the production delivery path

Do not let AI fluency become an excuse to skip the one message to the platform or security owner that prevents a leaked credential.

---

## Final Deliverable

A CI/CD review package:

- delivery-estate summary
- pipeline/trigger/runner inventory
- commit-to-production trace
- blast-radius and supply-chain risk findings
- validation and safe-test loop
- safe first change or recommendation
- updated delivery documentation
