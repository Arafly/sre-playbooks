# Complex Systems Safe Intervention Playbooks

A reusable SRE/platform engineering skill library built around:

**Map the terrain → Find the critical paths → Surface the risk → Externalize context → Validate with a fresh reviewer → Act or advise.**

## Included skills

- `cicd-pipeline-review`
- `design-review-adr`
- `due-diligence`
- `finops-review`
- `iam-access-review`
- `incident-response`
- `kubernetes-cluster-onboarding`
- `migration-readiness`
- `observability-slo-design`
- `repo-onboarding`
- `service-reliability-onboarding`
- `terraform-iac-review`

`service-reliability-onboarding` is the kernel. Other skills are standalone-triggerable modules.

## Core mechanisms

- **Tool, not toy:** interrogate, define acceptance criteria, then execute.
- **Elephant / Goldfish:** build context in a long-running session, then validate the durable artifact with a fresh context-free reviewer.
- **Two exits:** make a safe action or produce a defensible judgment.
