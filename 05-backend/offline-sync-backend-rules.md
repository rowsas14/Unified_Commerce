---
title: Offline Sync Backend Rules
folder: 05-backend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
stack: .NET Web API, Clean Architecture, PostgreSQL, EF Core
patterns: Service Pattern, Repository Pattern, Unit of Work
cqrs: not-used
---

# Offline Sync Backend Rules

Offline POS is a production requirement. The backend must safely accept, validate, deduplicate, process, reject, or conflict offline-created records.

## Database source

| Table | Backend purpose |
|---|---|
| `offline_sync_batches` | One reconnect/sync attempt from a POS device. |
| `offline_sync_items` | Generic sync item queue for offline-created records. |
| `offline_sale_sync_queue` | Typed sale staging linked one-to-one with sync item. |
| `offline_payment_sync_queue` | Typed payment staging linked one-to-one with sync item. |
| `offline_sync_conflicts` | Conflict record when item cannot be accepted cleanly. |
| `offline_sync_audit_logs` | Technical lifecycle log for sync processing. |

## Source-of-truth rule

Offline queues are staging records only.
Accepted data must be written to source-of-truth tables such as `sales`, `sale_lines`, `payments`, `sale_payment_allocations`, `stock_movements`, and `receipts`.

## Sync flow

```text
Device reconnects
  -> create sync batch
  -> store sync items
  -> create typed queue rows where applicable
  -> deduplicate by tenant + device + entity type + client entity id
  -> validate tenant/outlet/device/session
  -> process valid records into source-of-truth tables
  -> create conflict records for invalid/conflicting items
  -> write sync audit logs
  -> return item-level result
```

## Required validation

| Validation | Rule |
|---|---|
| Tenant | Device and payload tenant must match authenticated/sync context. |
| Outlet | Device outlet and payload outlet must match. |
| Device | Device must be registered and active. |
| Client IDs | Client entity IDs must be unique per tenant/device/entity type. |
| Session | Till session must be valid or conflict must be created. |
| Stock | Stock mismatch must create conflict, not silent corruption. |
| Payment | Offline payment must dedupe by client payment ID. |

## Conflict types

The database design lists conflict types including:

- `duplicate`
- `stock_mismatch`
- `price_changed`
- `closed_session`
- `validation_failed`

Backend must use these conflict meanings consistently.

## Processing order

Offline items in the same transaction group should be processed in business-safe order:

1. sale header/lines;
2. payment records;
3. payment allocations;
4. stock movements;
5. receipt records;
6. audit/reporting updates where applicable.

## Idempotency

Use database uniqueness rules and service checks:

- `offline_sync_items`: tenant + device + entity type + client entity ID;
- `sales`: tenant + source device + client transaction ID;
- `payments`: tenant + source device + client payment ID;
- `stock_movements`: tenant + source device + client movement ID.

## Status handling

| Status | Meaning |
|---|---|
| `pending` | Received but not processed. |
| `accepted` / `processed` | Valid item written to source-of-truth tables. |
| `rejected` | Invalid item rejected with error. |
| `conflict` | Requires explicit resolution. |

## Audit rule

Business effects use `audit_logs` where sensitive.
Technical sync events use `offline_sync_audit_logs`.

## Backend must not

- silently modify offline payloads to force success;
- accept duplicate client transactions;
- skip stock conflict detection;
- treat typed offline queues as source of truth;
- trust offline totals without server validation;
- process offline items without tenant/device validation.

## Checklist

- [ ] Batch created.
- [ ] Items stored.
- [ ] Dedupe applied.
- [ ] Tenant/outlet/device validated.
- [ ] Typed queues linked one-to-one where used.
- [ ] Source-of-truth records created only after validation.
- [ ] Conflicts created explicitly.
- [ ] Sync audit logs written.
