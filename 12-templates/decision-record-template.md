---
title: Decision Record Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, decision-record, adr, unified-commerce]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
---

# Decision Record Template

Use this template when the team makes a meaningful product, architecture, database, backend, frontend, security or operational decision.
Decision records prevent repeated debates and help AI IDE tools understand why the system is designed a certain way.

## File location examples

```text
13-project-history/decisions/ADR-001-tenant-feature-access-model.md
13-project-history/decisions/ADR-002-offline-sync-conflict-handling.md
07-modules/payments/decisions/refund-allocation-rules.md
```

## Copy template

```markdown
---
title: ADR-<number> - <Decision Title>
owner: Solution Architect
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce, decision, adr, <module>]
module: <module-or-cross-cutting>
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
related_docs:
  - [[01-product/project-scope]]
---

# ADR-<number> - <Decision Title>

## Status

Draft / Accepted / Superseded / Deprecated

## Decision date

<YYYY-MM-DD>

## Context

<Explain the problem, ambiguity or design choice.>

## Decision drivers

- <Driver 1>
- <Driver 2>
- <Driver 3>

## Options considered

| Option | Description | Pros | Cons |
|---|---|---|---|
| Option A | <description> | <pros> | <cons> |
| Option B | <description> | <pros> | <cons> |

## Decision

<Write the selected decision clearly.>

## Rationale

<Explain why this option is strongest for the production Unified Commerce system.>

## Consequences

### Positive

- <positive consequence>

### Negative / trade-offs

- <trade-off>

### Required follow-up

- [ ] <follow-up action>

## Affected documents

| Document | Required update |
|---|---|
| [[01-product/project-scope]] | <update> |
| [[03-data/database-overview]] | <update> |
| [[04-api/api-overview]] | <update> |
| [[05-backend/backend-overview]] | <update> |
| [[06-frontend/frontend-overview]] | <update> |

## Affected database tables

| Table | Impact |
|---|---|
| `<table>` | <impact> |

## Affected modules

| Module | Impact |
|---|---|
| `<module>` | <impact> |

## Validation checklist

- [ ] Decision does not violate tenant isolation.
- [ ] Decision does not bypass backend authorization.
- [ ] Decision does not corrupt payment, refund, stock or audit records.
- [ ] Decision aligns with uploaded source documents or records a justified correction.
- [ ] Related docs will be updated.
```

## When to create a decision record

Create a decision record for decisions such as:

- How tenant feature access is evaluated.
- Whether POS can complete card/QR payments while offline.
- How offline stock conflicts are resolved.
- Whether online orders can be returned in store.
- How split payment refunds are allocated.
- Whether prices are tax-inclusive or tax-exclusive per tenant.
- How loyalty points reverse during returns.
- Whether product publishing needs separate channel publication tables.
- Whether an operation belongs in a domain service or application service.

## Decision status values

| Status | Meaning |
|---|---|
| Draft | Proposed but not accepted. |
| Accepted | Approved and should guide implementation. |
| Superseded | Replaced by a newer decision. |
| Deprecated | No longer valid, but retained for history. |

## Good decision qualities

A strong decision record:

- Names the real problem.
- Lists real options.
- Explains trade-offs.
- States the final decision clearly.
- Identifies affected modules, files and tables.
- Records follow-up work.
- Helps future developers avoid guessing.

## Poor decision example

```text
We decided to do offline sync because it is better.
```

This is weak because it does not explain options, risks or consequences.

## Strong decision example

```text
We decided that offline sale and payment queue tables are staging tables only. Accepted server-side records must be written to sales, sale_lines, payments, allocations, stock_movements and receipts. This prevents duplicate source-of-truth records and keeps reports based on canonical transaction tables.
```

This is strong because it states a precise implementation rule.

## Production decision risk checklist

Before accepting a decision, ask:

| Question | Must answer |
|---|---|
| Does it affect tenant isolation? | Yes/No and how. |
| Does it affect financial totals? | Yes/No and how. |
| Does it affect stock accuracy? | Yes/No and how. |
| Does it affect offline sync? | Yes/No and how. |
| Does it affect permissions or feature access? | Yes/No and how. |
| Does it require database migration? | Yes/No. |
| Does it require API contract change? | Yes/No. |
| Does it require UI/UX change? | Yes/No. |
| Does it require test updates? | Yes/No. |

## Decision numbering

Use stable numbering if stored in a central decisions folder.

```text
ADR-001-tenant-feature-access-model.md
ADR-002-payment-idempotency-rules.md
ADR-003-offline-sync-source-of-truth.md
```

For module-local decisions, numbering is optional but still recommended.

## Superseding a decision

If a decision changes, do not silently edit history.
Add a note:

```markdown
Superseded by: `[[13-project-history/decisions/ADR-010-new-decision]]`
```

Then mark status:

```yaml
status: superseded
```

## AI IDE rule

AI IDE tools must obey accepted decision records.
If a prompt conflicts with an accepted decision, the AI IDE must highlight the conflict before making code or documentation changes.

## Completion checklist

- [ ] Context is clear.
- [ ] Options are real.
- [ ] Decision is explicit.
- [ ] Rationale is evidence-based.
- [ ] Consequences are honest.
- [ ] Affected docs/tables/modules are listed.
- [ ] Follow-up actions are listed.
- [ ] Status is correct.
