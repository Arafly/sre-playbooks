---
name: due-diligence
description: Use this for technical due diligence on a company, product, platform, or system being acquired, invested in, partnered with, or evaluated for integration. This is a time-boxed, evidence-based assessment conducted under limited access and usually without authority to change the system. Covers scoping to the deal thesis, architecture and operational maturity, security, compliance, licensing, key-person dependency, technical debt, integration cost, and producing a defensible go, no-go, or go-with-conditions recommendation. Trigger whenever the user mentions M&A, acquisition, investment, technical due diligence, vendor evaluation, integration assessment, or says things like "we're acquiring this company", "assess their technology", "evaluate this codebase before the deal", or "what technical risks should affect the price or terms".
---

# Due Diligence

## Why this exists

Technical due diligence has two defining constraints:

- a hard deadline
- limited access and usually no authority to change the system

The goal is not exhaustive understanding. The goal is to identify the technical facts and uncertainties capable of changing deal terms, valuation, integration cost, timeline, regulatory exposure, operational risk, or confidence in the deal thesis.

## Operating principles

- Scope to the deal thesis rather than reviewing everything equally.
- Treat missing evidence and restricted access as findings.
- Separate verified facts, supported inference, management assertion, and unknowns.
- Translate technical findings into decision impact.
- Use ranges and confidence levels rather than false precision.

## Phase 0 — Observe

Inventory the evidence available:

- data room
- repositories
- architecture docs
- cloud inventories
- incident records
- security assessments
- compliance evidence
- dependency and licence reports
- team interviews
- cost data
- customer commitments
- product roadmap
- operational metrics
- contracts with technical implications

Record what was reviewed, what was requested but unavailable, access limitations, time limitations, source reliability, and contradictions.

Prompt:

> Summarize the evidence available for this assessment, what remains unavailable, and how those gaps limit confidence.

Deliverable:

- evidence inventory
- access-gap register
- confidence constraints

## Phase 1 — Scope to the deal thesis

Identify why the transaction is happening:

- intellectual property
- revenue or customers
- product capability
- market entry
- team acquisition
- strategic data
- vendor consolidation

Translate the thesis into technical questions:

- Which systems create the claimed value?
- Which workflows generate revenue?
- Which IP is differentiated?
- Which customer promises depend on the technology?
- Which components must integrate with the buyer?
- Which risks could invalidate the thesis?

Deliverable:

- deal-thesis statement
- prioritized systems list
- time allocation
- material technical questions

## Phase 2 — Understand architecture, operations, and people

Map:

- architecture and stack
- infrastructure and data flows
- deployment and release model
- observability and incident history
- security boundaries
- third-party dependencies
- operational processes
- team structure and ownership
- bus factor
- scalability constraints
- customer-specific customisation

Identify key-person dependencies, undocumented manual processes, fragile contractor/vendor reliance, and access held by departing individuals.

Deliverable:

- architecture summary
- operational-maturity assessment
- team and bus-factor map
- critical workflow list

## Phase 3 — Surface deal-changing findings

Review for:

### Security and privacy

- known vulnerabilities
- weak IAM and secret management
- missing logging
- poor incident response
- unresolved penetration-test findings
- unsupported software
- privacy exposure

### Compliance and contracts

- control gaps
- audit findings
- data-residency issues
- unsupported certifications
- customer commitments

### Licensing and IP

- copyleft exposure
- unclear code ownership
- contractor IP ambiguity
- unapproved third-party code
- dependency licence risk

### Architecture and operations

- scalability limits
- single points of failure
- weak backup/restore
- fragile deployment
- undocumented systems
- high manual toil
- large deferred upgrades
- weak observability

### Vendor and integration risk

- vendor lock-in
- unsupported platforms
- expensive contracts
- data-model mismatch
- incompatible identity or operating models
- migration difficulty

Classify each finding by severity, evidence quality, probability, impact, remediation cost, deal-term impact, timeline impact, price impact, and owner.

Deliverable:

- risk-weighted findings register
- deal-changing findings list
- unverified material-risk list

## Phase 4 — Translate findings into decision impact

Convert findings into:

- integration-cost ranges
- remediation-cost ranges
- timeline effects
- staffing requirements
- operating-cost impact
- transition-service requirements
- retention requirements
- contractual protections
- closing conditions
- post-close priorities

Recommended finding format:

```text
Finding:
Evidence:
Confidence:
Technical impact:
Business/deal impact:
Estimated remediation:
Timeline effect:
Recommended condition or action:
Owner:
```

Deliverable:

- integration-cost estimate
- remediation roadmap
- risk register mapped to terms, timeline, and price

## Phase 5 — Goldfish-test the report

### Comprehension

> Based only on this report, what is being acquired or evaluated, what technology creates the value, and what are the most material risks?

### Decision

> Based only on this report, would you recommend go, no-go, or go with conditions? Explain which evidence drives that conclusion.

### Critic

> What material risk, unsupported claim, confidence gap, integration cost, key-person dependency, security issue, licensing concern, or contradictory evidence is missing?

If the Goldfish reaches a materially different conclusion, resolve whether the report is missing reasoning or the recommendation does not follow from the evidence.

Deliverable:

- Goldfish-reviewed report
- explicit evidence-to-recommendation chain

## Phase 6 — Advise and hand off

Produce one of:

- go
- go with conditions
- no-go
- insufficient evidence to decide

Possible conditions include remediation before close, price adjustment, warranties, access to missing evidence, key-person retention, transition services, or post-close deadlines.

Assign owners for every condition and post-close action.

## Stop Conditions

Do not issue an unconditional recommendation if:

- evidence central to the deal thesis is unavailable
- material security, licence, IP, compliance, or data risk remains unverified
- architecture claims conflict with observed systems
- key-person dependency is not understood
- integration cost lacks a credible basis
- facts and assumptions are mixed without distinction
- the Goldfish cannot reproduce the recommendation
- appropriate legal, security, financial, or domain review is missing for material findings

## Final Deliverable

A due-diligence package containing the evidence inventory, confidence limits, deal-thesis scope, architecture and operational assessment, team dependency analysis, material findings, integration and remediation ranges, deal implications, and a Goldfish-reviewed go, conditional-go, no-go, or insufficient-evidence recommendation.
