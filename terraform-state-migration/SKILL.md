---
name: terraform-state-migration
description: Use this when an already-existing remote resource must move between Terraform resource addresses, modules, states, workspaces, repositories, or ownership boundaries without recreating the real infrastructure object. Covers resource identity verification, source/destination state inspection, state backups, locking and change freezes, choosing same-state refactoring versus cross-state remove/import, zero-create/destroy plan invariants, post-migration reconciliation, and recovery. Trigger whenever the user says things like "move this resource to another Terraform state", "split this state", "move these resources to another repo/module", "Terraform wants to recreate an existing resource", "transfer ownership without destroying it", or "import this resource from one Terraform stack into another".
---

# Terraform State Migration

## Why this exists

Moving an already-existing remote resource between Terraform ownership boundaries is not an ordinary infrastructure change.

The desired infrastructure already exists. The task is to change **Terraform's binding to that object without changing the object itself**.

The governing invariant is:

```text
Before:
Remote object ID = X
Source state/address owns X

After:
Remote object ID = X
Destination state/address owns X

Terraform owner/address changed.
Remote object did not.
```

This skill sits at the intersection of `terraform-iac-review` and `migration-readiness`, but protects a narrower invariant: **ownership moves; infrastructure identity does not**.

---

## Operating principles

### Identity before state

Record the provider-side identity before changing Terraform ownership.

Examples:

- Datadog monitor/dashboard ID
- AWS ARN or resource ID
- Cloudflare object ID
- GitHub resource ID
- database identifier
- Kubernetes object identity

Names alone are not sufficient when the provider exposes a stable identifier.

### One remote object, one Terraform owner

At the end of the migration, the remote object must be bound to exactly one intended Terraform resource address.

Do not knowingly leave duplicate ownership across source and destination states.

### Prefer configuration-driven history

For same-state refactoring, prefer `moved` blocks where appropriate.

For cross-state migration on Terraform 1.7+, prefer a configuration-driven handoff using:

- `removed` with `destroy = false` on the source
- `import` on the destination

This leaves a reviewable record in configuration and allows normal plan/apply workflows.

Use direct state surgery only when the supported configuration-driven path is unsuitable and the reason is understood.

### Freeze concurrent writers

Do not migrate ownership while another engineer, CI/CD job, workspace, or automation may apply against either source or destination state.

### Never hand-edit state JSON

Use supported Terraform commands and language features.

### Zero unintended infrastructure change

A pure ownership migration should normally end with:

```text
0 unexpected to add
0 unexpected to change
0 unexpected to destroy
```

Any non-zero infrastructure mutation must be intentional, understood, and reviewed separately.

---

## Phase 0 — Classify the migration

Determine which class of move is required.

### A. Same-state address refactor

Examples:

```text
aws_instance.web
→ module.compute.aws_instance.web
```

or:

```text
aws_instance.old_name
→ aws_instance.new_name
```

Prefer a `moved` block when it can represent the refactor durably.

### B. Cross-state ownership transfer

Examples:

```text
monolith-prod-state
→ observability-prod-state
```

or:

```text
repo-a / workspace-a
→ repo-b / workspace-b
```

This is the primary use case for this skill.

### C. Ownership plus provider-context change

Examples:

- provider alias changes
- account/subscription/project changes
- region changes
- Datadog site/account changes
- tenant changes

Do not treat this as a pure ownership migration until provider identity equivalence is proven.

Deliverable:

- migration type
- source state/workspace/address
- destination state/workspace/address
- provider/account/region context

---

## Phase 1 — Establish remote identity

Before touching state, capture the real object's stable identity.

Record:

```text
Resource:
Provider:
Remote object ID:
Source Terraform address:
Source backend/workspace:
Destination Terraform address:
Destination backend/workspace:
Provider/account/region:
```

Inspect the source object:

```bash
terraform state show '<source-address>'
```

Where possible, verify independently through the provider API/UI/CLI.

Capture attributes useful for identity reconciliation, such as:

- immutable ID
- ARN
- account/project
- region
- name
- type
- critical immutable metadata

Useful prompt:

> Identify the remote object's stable identity and the attributes that would prove we are looking at the same object after the migration.

Deliverable:

- pre-migration identity record
- independent provider-side verification

Stop if remote identity cannot be proven.

---

## Phase 2 — Map source and destination ownership

Inspect both ownership boundaries.

For the source, record:

- backend and workspace
- current address
- module path
- provider alias
- dependencies
- outputs
- references from other resources
- CI/CD jobs capable of applying it
- remote-state consumers

For the destination, record:

- backend and workspace
- intended address
- module path
- provider alias
- variables/module inputs
- dependencies
- CI/CD jobs capable of applying it
- whether the remote object is already bound there

Useful prompts:

> What depends on the source address, state outputs, or provider binding?

> Could any other state, module, pipeline, or workspace already believe it owns this object?

Deliverable:

- source/destination ownership map
- dependency/reference list
- duplicate-ownership check

---

## Phase 3 — Freeze, snapshot, and establish recovery

Before changing ownership:

1. freeze automated applies to source and destination
2. coordinate human writers
3. confirm backend/workspace selection
4. confirm state locking behaviour
5. record Terraform and provider versions
6. back up source state
7. back up destination state
8. capture remote identity again
9. define the recovery path before execution

Typical state snapshots:

```bash
terraform state pull > source-before.tfstate
```

and from the destination context:

```bash
terraform state pull > destination-before.tfstate
```

Treat state files as sensitive material.

Deliverable:

- source backup
- destination backup
- freeze confirmation
- tool/provider version record
- documented recovery point

---

## Phase 4 — Prepare the destination configuration

Write the destination resource/module configuration before handing off ownership.

The destination should describe the existing remote object accurately enough that importing it does not immediately produce an unexpected update or replacement.

Compare:

- arguments
- defaults
- provider aliases
- lifecycle rules
- tags/labels
- parent module inputs
- `for_each`/`count` keys
- dependencies
- data sources
- computed versus configured values

Useful prompt:

> Compare the source and destination configurations. Identify every difference that could make Terraform update, replace, or destroy the existing object after import.

Deliverable:

- destination configuration
- configuration-difference report
- list of intentional versus accidental differences

---

## Phase 5 — Choose the migration mechanism

### Same state: durable refactor

Prefer `moved` blocks where appropriate:

```hcl
moved {
  from = old_resource.example
  to   = module.new_owner.new_resource.example
}
```

Use `terraform state mv` when an explicit state operation is required and the implications are understood.

### Cross state: preferred configuration-driven handoff

For Terraform 1.7+, prefer source-side removal without destruction:

```hcl
removed {
  from = old_resource.example

  lifecycle {
    destroy = false
  }
}
```

Prepare the destination resource plus an `import` block using the provider-specific identity:

```hcl
import {
  to = new_resource.example
  id = "<REMOTE_OBJECT_ID>"
}
```

The exact import identity is provider/resource-specific. Verify it from authoritative provider documentation or known working state before proceeding.

### Legacy/direct state movement

Direct cross-file `terraform state mv` / state pull-push workflows may still be required in some estates, especially older Terraform versions or constrained pipelines.

Treat that path as advanced state manipulation:

- preserve both state backups
- avoid concurrent writers
- verify serial/lineage implications
- review every state write
- prefer a rehearsed lower-risk environment first

Deliverable:

- selected mechanism
- reason for selection
- version compatibility note

---

## Phase 6 — Write the expected invariant

Before execution, define success explicitly.

Example:

```text
Remote object ID before: 123456
Remote object ID after:  123456

Source after handoff:
- old address no longer owns 123456
- source plan does not recreate it

Destination after handoff:
- new address owns 123456
- destination plan does not replace it

Provider:
- object 123456 still exists exactly once
- important attributes are unchanged
```

For a pure ownership migration:

```text
Unexpected creates  = 0
Unexpected destroys = 0
Unexpected replaces = 0
```

Deliverable:

- explicit pre/post migration contract

---

## Phase 7 — Goldfish readiness gate

Give a fresh session only:

- migration note
- source and destination configuration
- source/destination addresses
- backend/workspace facts
- remote object identity
- proposed mechanism
- recovery plan

Prompt:

> You have no prior context. Explain which existing remote object is moving, who manages it before and after, what must remain unchanged, what could cause recreation or duplicate ownership, and how recovery works if the handoff fails. Identify every ambiguity that would make this unsafe.

Do not proceed if a fresh reviewer cannot reconstruct the handoff correctly.

Deliverable:

- Goldfish readiness result
- unresolved ambiguity list

---

## Phase 8 — Execute the ownership handoff

Execute only the reviewed mechanism.

For cross-state configuration-driven handoff, the sequence must prevent an unsafe overlap or gap from becoming an infrastructure mutation.

At each step:

- confirm the correct backend/workspace
- confirm the expected plan before apply
- record commands, plans, timestamps, and approvals
- verify the remote object still exists
- stop on any unexpected create/destroy/replace

Do not combine the state migration with unrelated cleanup or configuration changes.

Deliverable:

- command/apply record
- plan evidence
- ownership-transfer record

---

## Phase 9 — Reconcile all three layers

After the handoff, prove that provider reality, destination state, and source state agree.

### Provider reality

Confirm:

```text
Object exists.
Object ID = original ID.
Critical immutable attributes are unchanged.
```

### Destination

Inspect:

```bash
terraform state show '<destination-address>'
terraform plan
```

Confirm the imported/bound object has the original provider identity and no unintended replacement/update is planned.

### Source

Confirm:

- old address is no longer owned
- source plan does not attempt to recreate the object
- old outputs/references are intentionally removed or redirected

Useful prompt:

> Reconcile provider reality, source state/configuration, and destination state/configuration. Identify any disagreement that means ownership transfer is incomplete.

Deliverable:

- post-migration reconciliation report

---

## Phase 10 — Recover if the invariant fails

Examples:

### Destination import fails

Stop. The remote object should still exist.

Correct destination configuration/import identity or restore the original ownership binding according to the pre-written recovery procedure.

### Destination wants replacement

Stop. Do not apply the replacement.

Investigate configuration, provider context, schema/version, and lifecycle differences.

### Source wants recreation

Stop. The source still declares or derives ownership somewhere.

Find the remaining declaration/reference before continuing.

### Wrong object was bound

Stop. Remove only the incorrect Terraform binding using a non-destructive supported mechanism, re-verify provider identity, and correct the handoff.

### State integrity is uncertain

Stop all writers and escalate. Use the preserved backups and backend-specific recovery procedure. Do not improvise state edits.

Deliverable:

- recovery record or validated restored state

---

## Phase 11 — Systematize

Document:

- why ownership moved
- original state/address
- destination state/address
- provider/account/region
- stable remote ID
- migration mechanism
- plans and verification evidence
- backup locations
- migration date
- approver/owner
- old references removed
- follow-up work

Retain configuration-driven `removed`, `import`, or `moved` history according to team standards where it provides useful institutional memory.

Deliverable:

- durable state-migration record
- updated Terraform estate documentation

---

## Two exits

### Successful ownership transfer

Terraform ownership changed while remote object identity remained constant.

### No-go / postpone

Do not migrate yet if:

- remote identity is uncertain
- source/destination ownership is unclear
- provider context differs unexpectedly
- state locking or writer coordination cannot be controlled
- destination configuration would mutate the object
- provider import semantics are unclear
- recovery is unproven

A defensible “not safe yet” is a complete result.

---

## Stop Conditions

Stop immediately if:

- source or destination backend/workspace is unclear
- concurrent applies cannot be frozen
- remote object identity cannot be proven
- destination already appears to own the object unexpectedly
- source dependencies are unknown
- source plan would recreate the object
- destination plan would destroy, replace, or unexpectedly mutate it
- provider alias/account/region/site differs from expectation
- state backups are unavailable
- recovery has not been defined
- state lineage or integrity becomes uncertain
- the user lacks authority to manipulate production Terraform ownership

---

## Final Deliverable

A Terraform state-migration package containing:

- migration classification
- source state/workspace/address
- destination state/workspace/address
- provider context
- stable remote object ID
- source/destination state backups
- dependency/reference map
- destination configuration comparison
- selected handoff mechanism
- expected plan invariant
- Goldfish readiness result
- execution record
- source post-migration plan
- destination post-migration plan
- provider-side identity verification
- recovery procedure
- final ownership record

The migration is complete only when:

```text
Remote object before = X
Remote object after  = X

Terraform owner before != Terraform owner after

Remote infrastructure did not change unintentionally.
```
