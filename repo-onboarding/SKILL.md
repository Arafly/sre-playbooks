---
name: repo-onboarding
description: Use this when entering an unfamiliar codebase — a new job, another team's repository, an open-source project, a legacy system, or any "I need to understand this code before I touch it" situation. Maps structure and architecture, identifies the vital 20% of files and workflows, surfaces risk from git and CI history, creates a durable repo context file, validates it with a fresh Goldfish session, and guides one small safe contribution. Trigger whenever the user is starting work in a repo they did not write, needs to understand an unfamiliar codebase, or says things like "help me get up to speed", "where should I start in this repo", "I just joined this project", or "I need to make a safe first contribution".
---

# Repo Onboarding

## Why this exists

The goal is not to read everything. The goal is to build enough accurate context to make a safe first contribution and leave the repository easier for the next person to understand.

The arc is:

**Observe → Map → Prioritize → Understand → De-risk → Operationalize → Contribute → Systematize**

## Operating principles

- Read less; map more.
- Run the system early.
- Preserve understanding in artifacts rather than chat history.
- Ask humans where artifacts stop.
- Scale the process to repo size and risk.

## Phase 0 — Observe and establish a feedback loop

Read:

- README
- CONTRIBUTING
- docs
- ADRs
- build and dependency files
- environment templates
- CI configuration
- local setup instructions

Then:

- build the project
- run it locally
- run the tests
- execute a smoke path
- record setup failures

Prompt:

> Summarize what this repository claims to do, how it is expected to run, and what remains unclear from the documentation.

Deliverable:

- working local loop or documented blocker
- one-screen repo summary
- setup-gotchas list

## Phase 1 — Map the terrain

Identify:

- application code
- entry points
- workers and schedulers
- tests
- deployment and infrastructure
- generated code
- shared libraries
- scripts and tooling
- data models
- external integrations

Useful commands:

```bash
tree -L 2
find . -maxdepth 2 -type f | sort
```

Prompts:

> Explain the purpose of the top-level directories and key files.

> Where are the application entry points, background workers, CLI roots, and deployment definitions?

Deliverable:

- annotated directory map
- stack and entry-point summary
- initial architecture sketch

## Phase 2 — Prioritize the vital 20%

Identify the 5–15 files and 1–3 workflows that explain most runtime behaviour.

Use:

- entry points
- dependency structure
- code references
- recent modifications
- incident history
- business importance

Useful commands:

```bash
git log --name-only --since="6 months ago" | sort | uniq -c | sort -nr | head -30
git log --stat --since="6 months ago"
```

Prompts:

> Which 20% of this codebase should I focus on to understand 80% of its functionality?

> Which files are most central to runtime behaviour, and why?

> Which workflows would cause the most damage if they broke?

Deliverable:

- must-know file shortlist
- critical-workflow list
- reading order

Goldfish cross-check:

> Based only on this architecture summary, which files and workflows should a new engineer inspect first? What appears to be missing from the summary?

## Phase 3 — Understand critical workflows

Trace one or two core paths from input to output.

Capture:

- entry point
- validation
- domain logic
- state changes
- persistence
- external calls
- async work
- retries
- error handling
- security checks
- observable signals

Prompt:

> Trace how `[feature or workflow]` moves through this system from input to completion, including validation, persistence, side effects, retries, and failure handling.

Deliverable:

- end-to-end workflow trace
- critical function/class notes
- domain glossary
- updated architecture diagram

## Phase 4 — Surface risk and history

Review:

- high-churn files
- failing or flaky CI
- recent merged PRs
- reverts and hotfixes
- postmortems
- security-sensitive paths
- skipped tests
- TODO/FIXME/HACK/workaround comments
- ownership history

Useful commands:

```bash
git log --oneline -- path/to/file
git blame path/to/file
grep -R -E "TODO|FIXME|HACK|workaround|temporary|skip" .
git shortlog -sn
```

Prompts:

> What recent history suggests this area is fragile?

> What should a new contributor avoid touching first?

> Which assumptions require confirmation from a human owner?

Deliverable:

- hot-zone and risk map
- historical-decision notes
- security/secrets note
- “who to ask about what” list

## Phase 5 — Operationalize debugging

Capture:

- test commands
- local reproduction steps
- logs, metrics, and traces
- health checks
- fixtures and seed data
- common errors
- debugger setup
- feature flags
- reset and rollback procedures

Prompts:

> What tools, commands, logs, and tests should I use to debug this component?

> What are the common sharp edges a new contributor should know?

Deliverable:

- troubleshooting runbook
- gotchas list

## Phase 6 — Build and Goldfish-test durable context

Create or update a repo context artifact such as:

- `ONBOARDING.md`
- `AGENTS.md`
- `CLAUDE.md`
- `repo-context.md`

Include purpose, architecture, stack, entry points, commands, conventions, critical workflows, must-know files, hot zones, ownership, debugging, tests, deployment path, security-sensitive areas, gotchas, open questions, and first-contribution ideas.

For large repositories, build concise human-reviewed directory READMEs from leaf directories upward.

Goldfish prompt:

> You have no prior context. Read this repo context file and referenced artifacts. Explain what the system does, how it is structured, how its critical workflows operate, where the risks are, and what a safe first contribution might be. List anything unclear or unsupported.

Deliverable:

- Goldfish-validated repo context file

## Phase 7 — Make one small safe contribution

Good first contributions:

- missing test
- documentation or setup fix
- clearer error message
- runbook fix
- small validation change
- observability improvement
- isolated bug fix

Avoid without owner approval:

- broad refactors
- public API changes
- database migrations
- auth changes
- CI/CD redesign
- deployment changes
- shared base configuration
- retry, timeout, or concurrency defaults
- core abstractions

For a non-trivial change, write and Goldfish-test a design note containing the problem, proposed change, affected files, acceptance criteria, validation, risk, rollback, and reviewers.

Deliverable:

- small reviewed contribution
- or defensible recommendation/escalation

## Phase 8 — Systematize

Commit or share the context file, improved docs, gotchas log, architecture diagram, ownership notes, troubleshooting runbook, remaining risks, and recommended next steps.

## Stop Conditions

Do not make the first change if:

- critical workflow understanding is incomplete
- ownership is unknown
- tests cannot be run or evaluated
- rollback is unclear
- the change touches auth, secrets, CI/CD, deployment, schema, public APIs, shared configuration, or critical data paths without approval
- the Goldfish cannot understand the context or design note
- the user lacks permission to modify the repository or related systems

## Final Deliverable

A repo-onboarding package containing the working local loop or blockers, architecture and directory maps, must-know files, critical workflow traces, risk and ownership map, debugging runbook, Goldfish-validated context file, one small safe contribution or defensible recommendation, and improved documentation for the next contributor.
