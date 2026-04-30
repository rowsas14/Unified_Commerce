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
# Offline Sync API Rules

Offline POS is production-critical. The API must support safe reconnect, duplicate prevention, server validation, conflict creation, and sync audit.
Offline sync must not blindly insert local data into final tables.

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


## Related database tables

| Table | API role |
|---|---|
| `offline_sync_batches` | One reconnect/sync attempt from a POS device. |
| `offline_sync_items` | Generic sync item staging queue. |
| `offline_sale_sync_queue` | Typed sale staging extension. |
| `offline_payment_sync_queue` | Typed payment staging extension. |
| `offline_sync_conflicts` | Conflict records requiring explicit resolution. |
| `offline_sync_audit_logs` | Technical sync lifecycle events. |
| `sales`, `payments`, `stock_movements`, `receipts` | Final accepted source-of-truth tables. |

## Route-family pattern

```http
/api/v1/offline-sync/batches
/api/v1/offline-sync/items
/api/v1/offline-sync/conflicts
```

These are route families. Concrete payloads must be documented in feature specs.

## Sync process

```text
POS reconnects
  → create/submit sync batch
  → submit sync items with local IDs and payloads
  → server validates tenant/outlet/device/session context
  → server accepts, rejects, or marks conflict per item
  → accepted items create final records
  → conflict items create offline_sync_conflicts
  → response updates local queue status
```

## Required payload concepts

| Field/concept | Why needed |
|---|---|
| Tenant/device context | Validate device belongs to tenant/outlet. |
| Client entity ID | Prevent duplicate local item processing. |
| Client transaction ID | Group sale/payment/receipt/stock movement. |
| Entity type | sale, payment, receipt, stock movement, cash movement, return, exchange. |
| Payload | Original offline transaction data. |
| Offline occurred time | Preserve local event timing. |
| App/device version | Useful for troubleshooting where available. |

## Server validation rules

The API must validate:

- registered device is active and belongs to tenant/outlet;
- sync batch tenant/outlet/device match;
- client entity ID was not already processed;
- sale/payment/stock/receipt references are consistent;
- till session is valid for the offline transaction rules;
- products/variants exist and belong to tenant;
- stock and payment state can be accepted or must conflict;
- payload structure matches expected entity type.

## Sync statuses

| Status | Meaning |
|---|---|
| `pending` | Received but not processed. |
| `accepted` | Validated and accepted into final source tables. |
| `rejected` | Invalid and not accepted. |
| `conflict` | Requires explicit conflict resolution. |

## Conflict types

The database design identifies conflict types such as:

- duplicate;
- stock mismatch;
- price changed;
- closed session;
- validation failed.

Do not resolve these silently if business state could be corrupted.

## Response behavior

The sync response must tell the POS client:

- batch status;
- item-level status;
- server entity IDs for accepted items;
- rejection errors for rejected items;
- conflict IDs and conflict type for unresolved items;
- whether retry is allowed.

## Idempotency rules

Use the unique client IDs and transaction IDs documented in [[04-api/idempotency-rules]].
A repeated sync item should return the existing accepted/rejected/conflict state, not create duplicate records.

## Audit/logging rules

Business outcomes belong in source tables and `audit_logs` where needed.
Technical sync lifecycle belongs in `offline_sync_audit_logs`.

## Checklist

- [ ] Sync uses `/api/v1/offline-sync/...`.
- [ ] Registered device is validated.
- [ ] Tenant/outlet/device context is validated.
- [ ] Client IDs support dedupe.
- [ ] Payload is validated by entity type.
- [ ] Accepted items create final source records.
- [ ] Conflicts create conflict records.
- [ ] Response updates client queue deterministically.
- [ ] Technical audit log is written.
