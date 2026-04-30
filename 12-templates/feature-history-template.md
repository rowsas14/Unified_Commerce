---
title: Feature History Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, feature-history, changelog, bugs]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
---

# Feature History Template

Use this template for every feature history file.
Feature history tracks implementation notes, bug fixes, scope changes and production decisions for one feature.

## File location

```text
07-modules/<module>/features/<feature>/feature-history.md
```

## Copy template

```markdown
---
title: <Feature Name> Feature History
owner: <Team or role>
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce, feature-history, <module>, <feature>]
module: <module>
feature: <feature>
related_docs:
  - `[[07-modules/<module>/features/<feature>/feature-spec]]`
---

# <Feature Name> Feature History

## Purpose

This file records the implementation history, bug fixes, production notes and important changes for `<feature>`.

## Current feature state

| Item | Value |
|---|---|
| Feature spec | `[[07-modules/<module>/features/<feature>/feature-spec]]` |
| Current status | draft / in-review / production-ready |
| Backend implemented | Yes/No/Partial |
| Frontend implemented | Yes/No/Partial |
| API implemented | Yes/No/Partial |
| Tests implemented | Yes/No/Partial |
| Production risk | Low/Medium/High |

## Change log

| Date | Change type | Summary | Owner | Related docs |
|---|---|---|---|---|
| YYYY-MM-DD | Added/Changed/Fixed/Deprecated | <summary> | <owner> | <links> |

## Implementation notes

| Area | Notes |
|---|---|
| Backend | <service/controller/domain notes> |
| Frontend | <page/component/state notes> |
| Database | <table/index/migration notes> |
| API | <endpoint/request/response notes> |
| Tests | <test notes> |

## Bug history

| Date | Bug | Severity | Status | Fix summary | Test added |
|---|---|---|---|---|---|
| YYYY-MM-DD | <bug summary> | Low/Medium/High/Critical | Open/Fixed/Closed | <fix> | Yes/No |

## Production incidents

| Date | Incident | Impact | Resolution | Follow-up |
|---|---|---|---|---|
| YYYY-MM-DD | <incident> | <impact> | <resolution> | <follow-up> |

## Decisions affecting this feature

| Decision | Link | Impact |
|---|---|---|
| <decision title> | `[[13-project-history/...]]` | <impact> |

## Known limitations

- <Limitation 1>
- <Limitation 2>

## Regression watchlist

- [ ] Tenant isolation still enforced.
- [ ] Permission and feature access still enforced.
- [ ] Financial totals still correct.
- [ ] Stock movements still auditable.
- [ ] Offline sync still idempotent where applicable.
- [ ] Audit logs still created for sensitive actions.

## Next review actions

- [ ] <Action 1>
- [ ] <Action 2>
```

## What belongs in feature history

Use this file for:

- Implementation changes.
- Bug fixes.
- Validation rule changes.
- API contract changes.
- Database migration notes.
- Frontend behavior changes.
- Production incidents.
- Regression risks.
- Notes about why a behavior changed.

## What does not belong here

Do not duplicate the full feature spec.
Do not write broad project release notes here.
Use [[13-project-history/release-notes]] for release-level notes.

## Change types

| Type | Meaning |
|---|---|
| `Added` | New behavior, endpoint, UI or rule. |
| `Changed` | Existing behavior changed. |
| `Fixed` | Bug fixed. |
| `Deprecated` | Behavior should no longer be used. |
| `Removed` | Behavior removed after replacement. |
| `Security` | Access, tenant isolation, audit or protection fix. |
| `Data` | Table, relationship, migration or seed change. |
| `Offline` | Offline cache, queue, sync or conflict change. |

## Severity scale

| Severity | Meaning |
|---|---|
| Critical | Payment, refund, stock, tenant isolation, security or data corruption risk. |
| High | Major workflow broken or important report incorrect. |
| Medium | Feature works partially or edge case broken. |
| Low | UI, copy, minor validation or non-blocking issue. |

## Production-specific regression examples

### POS sale feature

Watch for:

- Duplicate sale after offline sync retry.
- Stock deducted from wrong outlet.
- Receipt generated without completed payment.
- Cash sale not reflected in till session expected cash.

### Payment/refund feature

Watch for:

- Refund exceeding captured amount.
- Split payment refund allocation mismatch.
- Missing idempotency key on retry-prone endpoint.
- Gateway status mismatch with internal payment status.

### Inventory feature

Watch for:

- Negative stock created without tenant setting.
- Stock movement missing required source reference.
- Reservation not released on order cancellation.
- Damaged return restored as sellable stock incorrectly.

### RBAC/feature access feature

Watch for:

- Tenant-disabled feature visible or executable.
- Role missing permission but endpoint still succeeds.
- Outlet staff accessing another outlet.
- Platform user treated as tenant user.

## Link requirements

Every feature history should link to:

- Its feature spec.
- Related bug reports.
- Related decision records.
- Related test files.

Example:

```markdown
Related bug: [[13-project-history/overall-bugs]]
Related test: [[10-testing-quality/payment-refund-test-cases]]
```

## AI IDE rule

AI IDE tools must update `feature-history.md` after implementing or fixing a feature.
If the implementation changes scope, rules, tables, APIs, frontend behavior or tests, the feature history must record it.

## Completion checklist

- [ ] Current feature state is accurate.
- [ ] Changes are dated.
- [ ] Bug fixes include test coverage.
- [ ] Production incidents include impact and follow-up.
- [ ] Decisions are linked.
- [ ] Known limitations are visible.
- [ ] Regression watchlist includes tenant/RBAC/financial/data risks where relevant.
