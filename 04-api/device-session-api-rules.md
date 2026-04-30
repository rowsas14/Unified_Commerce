---
title: {title}
owner: API Architecture / Backend Architecture
status: production-ready
last_reviewed: {DATE}
tags: [{tags}]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
---
# Device and Session API Rules

POS APIs must understand device, outlet, till, and session context.
This is required because POS sales, stock deduction, receipts, cash movements, and offline sync depend on the correct terminal context.

## Required reading

- [[00-start-here/README|Start Here]]
- [[01-product/project-scope|Production Project Scope]]
- [[01-product/production-module-catalog|Production Module Catalog]]
- [[02-architecture/system-overview|System Overview]]
- [[02-architecture/tenancy-architecture|Tenancy Architecture]]
- [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
- [[03-data/database-overview|Database Overview]]
- [[03-data/tenant-consistency-rules|Tenant Consistency Rules]]
- [[09-security-and-compliance/authorization-model|Authorization Model]]
- [[09-security-and-compliance/audit-requirements|Audit Requirements]]


## Related tables

| Table | Purpose |
|---|---|
| `tills` | Cash register/till master per tenant/outlet. |
| `pos_devices` | Registered POS terminal/browser/device. |
| `till_sessions` | Open/close session per till. |
| `cash_movement_types` | Reference values for cash in/out/safe drop/etc. |
| `cash_movements` | Non-sale cash movement within a session. |
| `cash_count_denominations` | Denomination-level till close count. |
| `sales` | POS sale header linked to till session and device. |
| `payments` | Payment records linked to outlet/till session where applicable. |
| `receipts` | Receipt rows linked to source device where applicable. |

## Device context rules

| Rule | API implication |
|---|---|
| Device belongs to tenant | Validate before POS workflows. |
| Device belongs to outlet | Use correct stock and session context. |
| Device belongs to till | Validate session and cash drawer context. |
| Device has status | Block inactive/blocked devices. |
| Device has fingerprint/code | Prevent unknown terminal access. |
| Device tracks app version/last seen | Useful for sync/support. |

## Till session rules

| Rule | API implication |
|---|---|
| One open session per till | Enforce before opening. |
| Session required for POS billing where enabled | Block sale completion without session. |
| Opening float required | Validate non-negative amount. |
| Cash movements belong to session | Validate session is open where required. |
| Close requires counted cash where configured | Calculate expected/count/variance. |
| Variance may require approval | Permission and audit required. |

## POS sale context

A POS sale API must validate:

- tenant context;
- outlet context;
- active user;
- registered active POS device;
- active till session;
- products/variants belong to tenant;
- stock belongs to outlet;
- payment allocation belongs to same tenant/session where relevant;
- receipt is generated for correct outlet/device context.

## Cash movement context

Cash movement APIs must validate:

- active till session;
- cash movement type;
- positive amount;
- reason;
- performed by user;
- manager approval where required;
- source device where applicable;
- offline client cash movement ID where applicable.

## Offline device sync context

Offline sync APIs must validate that:

- device is registered;
- device is active;
- device belongs to tenant/outlet;
- sync batch outlet/device match;
- offline transaction references same device context;
- duplicate local IDs are handled.

## API response requirements

POS clients need clear state responses:

| State | Needed by frontend |
|---|---|
| Device registered/blocked | Allow or block POS screen. |
| Outlet assigned | Use correct stock and receipt context. |
| Till assigned | Show session controls. |
| Session open/closed | Lock/unlock billing. |
| Offline status | Control sync queue visibility. |
| Printer/scanner status | UI support where configured. |

## Checklist

- [ ] Device belongs to tenant.
- [ ] Device belongs to outlet and till.
- [ ] Device status is active.
- [ ] User has outlet access.
- [ ] Till session state is valid.
- [ ] Cash movement/session rules are enforced.
- [ ] Offline IDs are deduped.
- [ ] Sensitive device/session actions are audited.
