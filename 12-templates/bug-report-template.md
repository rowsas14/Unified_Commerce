---
title: Bug Report Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, bug-report, qa, unified-commerce]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
---

# Bug Report Template

Use this template for production bugs, QA defects and implementation defects.
Bugs in this system must be documented with module, tenant, data, permission and workflow impact.

## File location examples

```text
13-project-history/overall-bugs.md
07-modules/payments/features/refunds/bugs/BUG-001-refund-exceeds-payment.md
07-modules/offline-sync/features/sync-conflicts/bugs/BUG-002-duplicate-offline-sale.md
```

## Copy template

```markdown
---
title: BUG-<number> - <Bug Title>
owner: QA Team
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce, bug, <module>, <feature>]
module: <module>
feature: <feature>
severity: Critical / High / Medium / Low
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
related_docs:
  - `[[07-modules/<module>/features/<feature>/feature-spec]]`
---

# BUG-<number> - <Bug Title>

## 1. Summary

<One-paragraph summary of the bug.>

## 2. Severity

| Field | Value |
|---|---|
| Severity | Critical / High / Medium / Low |
| Priority | P0 / P1 / P2 / P3 |
| Status | Open / In progress / Fixed / Verified / Closed |
| Reported by | <role/person> |
| Reported date | <YYYY-MM-DD> |

## 3. Affected area

| Area | Value |
|---|---|
| Module | `<module>` |
| Feature | `<feature>` |
| Channel | POS / E-Commerce / Admin / Platform / Offline Sync |
| Tenant scoped | Yes/No |
| Outlet scoped | Yes/No |
| Device/session involved | Yes/No |

## 4. Environment

| Item | Value |
|---|---|
| Environment | Local / Dev / QA / Staging / Production |
| Browser/device | <value> |
| App version | <value> |
| API version | <value> |
| Tenant | <tenant code or anonymized ID> |
| Outlet | <outlet code or N/A> |
| User role | <role> |

## 5. Preconditions

- <Required setup>
- <Required data>
- <Required role/permission>

## 6. Steps to reproduce

1. <Step 1>
2. <Step 2>
3. <Step 3>

## 7. Expected result

<What should happen according to the feature spec/source documents.>

## 8. Actual result

<What actually happened.>

## 9. Evidence

| Evidence | Link/path |
|---|---|
| Screenshot | <path> |
| Log | <path> |
| API response | <path> |
| Database record | <safe reference> |

## 10. Impact assessment

| Impact area | Impact |
|---|---|
| Tenant isolation | None / Possible / Confirmed |
| Payment/refund | None / Possible / Confirmed |
| Stock/inventory | None / Possible / Confirmed |
| Order/fulfillment | None / Possible / Confirmed |
| Offline sync | None / Possible / Confirmed |
| Reporting | None / Possible / Confirmed |
| Audit | None / Possible / Confirmed |

## 11. Root cause

<Fill after investigation.>

## 12. Fix summary

<Fill after fix.>

## 13. Files changed

| Area | Files |
|---|---|
| Backend | <files> |
| Frontend | <files> |
| Database | <migration/config> |
| Docs | <docs updated> |

## 14. Tests added or updated

| Test | Location | Status |
|---|---|---|
| <test> | <path> | Added/Updated |

## 15. Verification checklist

- [ ] Bug no longer reproduces.
- [ ] Regression test added.
- [ ] Tenant isolation still works.
- [ ] Permission/feature access still works.
- [ ] Financial/stock/reporting impact checked where relevant.
- [ ] Feature history updated.
```

## Severity rules

| Severity | Use when |
|---|---|
| Critical | Data leakage, payment/refund error, stock corruption, duplicate transaction, security bypass, production outage. |
| High | Core POS/e-commerce workflow blocked, major report wrong, offline sync broken, fulfillment impossible. |
| Medium | Important edge case broken but workaround exists. |
| Low | Minor UI issue, wording issue, non-blocking validation issue. |

## Priority rules

| Priority | Meaning |
|---|---|
| P0 | Fix immediately before continuing release/deployment. |
| P1 | Fix in current sprint/release. |
| P2 | Fix soon, not release blocking unless repeated. |
| P3 | Backlog or polish. |

## Bugs that are always high risk

Treat these as High or Critical unless proven otherwise:

- Tenant A can see Tenant B data.
- User without permission can approve refund, discount, void or stock adjustment.
- Offline sync creates duplicate sale/payment.
- Stock deducted from wrong outlet.
- Refund exceeds original captured payment.
- Receipt reprint happens without audit.
- Payment status and order status are inconsistent.
- Tax/discount calculation differs between POS, order and receipt.
- Cash drawer expected cash is wrong.

## Root cause categories

| Category | Example |
|---|---|
| Validation | Missing server-side rule. |
| Authorization | Permission or feature access bypass. |
| Tenant context | Wrong or missing tenant/outlet filter. |
| Transaction boundary | Partial save caused inconsistent data. |
| Idempotency | Retry created duplicate record. |
| Offline sync | Bad dedupe/conflict handling. |
| Calculation | Wrong tax, discount, total, cash or stock calculation. |
| UI state | Zustand/local state mismatch. |
| API contract | Request/response mismatch. |
| Data migration | Bad seed/default/index/constraint. |

## Fix documentation rule

After a bug is fixed, update:

- The bug report.
- The related `feature-history.md`.
- The related test file under `10-testing-quality/`.
- The related feature spec if the rule was missing.
- A decision record if the fix required a design decision.

## Example bug title format

```text
BUG-001 - Offline sale sync creates duplicate payment on retry
BUG-002 - Cashier can reprint receipt without permission
BUG-003 - Online order refund exceeds captured card payment
```

## Evidence safety rule

Do not paste real secrets, passwords, card data, OTP codes, private keys or customer-sensitive data into bug reports.
Use anonymized IDs or safe screenshots.

## AI IDE rule

AI IDE tools fixing a bug must first read:

1. The bug report.
2. The related feature spec.
3. The related entity/API/backend/frontend docs.
4. The related test cases.

After fixing, AI IDE must update feature history and test coverage notes.

## Completion checklist

- [ ] Bug title is specific.
- [ ] Severity is justified.
- [ ] Steps reproduce the issue.
- [ ] Expected vs actual result is clear.
- [ ] Impact areas are checked.
- [ ] Root cause is recorded after investigation.
- [ ] Fix summary is recorded.
- [ ] Tests are added or updated.
- [ ] Feature history is updated.
- [ ] No sensitive data is pasted.
