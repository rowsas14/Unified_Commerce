---
title: Security Review Checklist
owner: Security / QA / Architecture
status: draft
last_reviewed: 2026-04-30
tags:
  - security-review
  - checklist
  - qa
  - release-readiness
---

# Security Review Checklist

## Purpose

Use this checklist before implementing, reviewing, merging, or releasing security-relevant changes.
The checklist is specific to the Unified Commerce production system and its uploaded scope/database/architecture documents.

## How to use

- Product owners use this to confirm business rules are documented.
- Architects use this to confirm tenant, security, and data boundaries.
- Backend developers use this before implementing services and APIs.
- Frontend developers use this before implementing guards, POS state, and offline UI.
- QA engineers use this to create test cases.
- AI IDE tools use this as a gate before code changes.

## 1. Source alignment

| Check | Pass |
|---|---|
| Feature exists in uploaded scope or current approved 2nd Brain | ☐ |
| Entity/table exists in uploaded database design or approved schema extension doc | ☐ |
| No new security workflow was invented without source | ☐ |
| Links to relevant product/data/API/backend/frontend docs are included | ☐ |
| Feature is not incorrectly described as basic/MVP | ☐ |

## 2. Authentication checks

| Check | Pass |
|---|---|
| Platform user, tenant staff, and customer identity are not mixed | ☐ |
| Passwords are stored as hashes only | ☐ |
| OTP values are stored as hashes only | ☐ |
| Inactive/suspended identities are rejected | ☐ |
| Customer auth remains tenant-scoped | ☐ |
| Guest customer behavior is explicit | ☐ |

## 3. Authorization checks

| Check | Pass |
|---|---|
| Backend validates permission server-side | ☐ |
| Feature entitlement is checked where feature-gated | ☐ |
| Role feature assignment is checked where applicable | ☐ |
| Runtime feature flag is checked where applicable | ☐ |
| Frontend guard is not treated as final security | ☐ |
| Sensitive actions require proper permission | ☐ |

## 4. Tenant isolation checks

| Check | Pass |
|---|---|
| Every tenant-owned query filters by tenant context | ☐ |
| FK IDs are validated for same tenant | ☐ |
| Outlet IDs belong to same tenant | ☐ |
| Customer data does not cross tenants | ☐ |
| Reports are tenant/outlet/channel scoped | ☐ |
| Offline cache contains only current tenant/outlet data | ☐ |

## 5. POS session and device checks

| Check | Pass |
|---|---|
| POS device belongs to tenant/outlet/till | ☐ |
| Blocked device cannot perform sensitive actions | ☐ |
| Active till/session required where session control applies | ☐ |
| Cash session open/close rules are validated | ☐ |
| Receipt reprint is permission-controlled | ☐ |
| Printer failure does not corrupt sale transaction | ☐ |

## 6. Payment and refund checks

| Check | Pass |
|---|---|
| Provider secrets use `secret_ref`, not config JSON | ☐ |
| Payment idempotency is implemented for retryable operations | ☐ |
| Captured amount cannot exceed amount | ☐ |
| Payment allocations cannot exceed captured amount | ☐ |
| Refund total cannot exceed original captured payment | ☐ |
| Refund action is audited and permission-controlled | ☐ |
| Raw provider payload is not source of financial truth | ☐ |

## 7. Offline sync checks

| Check | Pass |
|---|---|
| Offline queues are treated as staging, not source of truth | ☐ |
| Client transaction IDs are used for dedupe | ☐ |
| Sync validates tenant/outlet/device/session | ☐ |
| Stock conflict creates conflict record, not silent correction | ☐ |
| Blocked devices cannot sync accepted data | ☐ |
| Sync lifecycle creates sync audit logs | ☐ |

## 8. Inventory and stock checks

| Check | Pass |
|---|---|
| Stock changes create `stock_movements` | ☐ |
| Movement quantities are positive; direction comes from movement type | ☐ |
| Required movement references are validated | ☐ |
| Damaged stock is not automatically sellable | ☐ |
| Negative stock behavior follows configured rule | ☐ |
| Stock adjustment/posting is permission-controlled | ☐ |

## 9. Returns, exchanges, discounts checks

| Check | Pass |
|---|---|
| Return references original sale or order where required | ☐ |
| Returned quantity cannot exceed eligible quantity | ☐ |
| Non-returnable/expired policy requires block or override | ☐ |
| Exchange difference uses payment/refund rules | ☐ |
| Discount threshold approval is enforced | ☐ |
| Coupon usage limits are transaction-safe | ☐ |

## 10. Audit checks

| Check | Pass |
|---|---|
| Sensitive action writes audit or specific ledger/log | ☐ |
| Audit does not contain passwords, OTP, provider secrets, or card data | ☐ |
| Actor, tenant, entity, action, and timestamp are captured | ☐ |
| Business audit and technical sync audit are separated | ☐ |
| Audit rows are not edited by normal application users | ☐ |

## 11. API checks

| Check | Pass |
|---|---|
| Request validates tenant/outlet/device context | ☐ |
| Response does not leak secret fields | ☐ |
| Error contract does not expose sensitive internals | ☐ |
| Idempotency is used where duplicate requests are possible | ☐ |
| Invalid status transitions are rejected | ☐ |
| Authorization failure returns controlled error | ☐ |

## 12. Frontend checks

| Check | Pass |
|---|---|
| UI clearly shows tenant/outlet/device/session state | ☐ |
| Feature-disabled states are visible and clear | ☐ |
| Hidden UI actions still rely on backend validation | ☐ |
| Offline indicator and sync status are visible | ☐ |
| Sensitive actions use confirmation/reason/approval UI where required | ☐ |
| Local stores do not mix tenant/outlet data | ☐ |

## 13. DevOps and operations checks

| Check | Pass |
|---|---|
| Payment provider secrets are not committed to repository | ☐ |
| Obsidian/local API secrets are not included in export | ☐ |
| Seed data includes reference/security tables | ☐ |
| Monitoring covers failed payments, sync failures, conflicts, blocked devices | ☐ |
| Backup/recovery considers audit and transaction data | ☐ |
| Support runbooks exist for payment/sync/device incidents | ☐ |

## AI IDE gate

Before AI IDE modifies security-sensitive code, it must read:

- [[README]],
- [[authentication-model]],
- [[authorization-model]],
- [[data-isolation-controls]],
- [[sensitive-actions]],
- relevant data/API/backend/frontend docs.

## Review result

| Result | Meaning |
|---|---|
| Approved | All required checks passed |
| Approved with notes | Minor non-blocking documentation or implementation notes remain |
| Blocked | Security, tenant, payment, sync, audit, or source alignment issue found |

## Related documents

- [[README]]
- [[authentication-model]]
- [[authorization-model]]
- [[data-isolation-controls]]
- [[audit-requirements]]
- [[sensitive-actions]]
- [[10-testing-quality/release-readiness-checklist]]
- [[11-delivery-and-operations/production-go-live-checklist]]
- [[14-ai-ide-rules/production-scope-alignment-rule]]
