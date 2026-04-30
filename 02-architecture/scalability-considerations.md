---
title: Scalability Considerations
owner: Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [architecture, scalability, postgresql, caching]
source: Unified Commerce production scope + database design
---

# Scalability Considerations

## Purpose

This file defines scalability considerations for the production Unified Commerce E-POS + E-Commerce SaaS system.

The system must scale without weakening tenant isolation, financial correctness, stock accuracy, payment safety, offline sync consistency or auditability.

Read with:

- [[02-architecture/system-overview]]
- [[02-architecture/offline-first-architecture]]
- [[03-data/indexing-strategy]]
- [[05-backend/caching-strategy]]
- [[04-api/concurrency-rules]]

---

## Core scalability rules

- Index tenant/date/status-heavy queries.
- Use reporting read models for dashboards.
- Partition/archive audit and stock ledgers when needed.
- Process offline sync and reports asynchronously where appropriate.
- Use idempotency keys for retries.
- Do not introduce generic cache tables.
- Revalidate source tables before critical writes.

---

## PostgreSQL caching and read performance

PostgreSQL-based performance must be handled through:

| Layer | Purpose |
|---|---|
| Indexes | Fast tenant, outlet, date, status, SKU, barcode and idempotency lookups. |
| Query design | Avoid scanning all tenants or wide tables unnecessarily. |
| Existing read models | Dashboard and summary views. |
| Database memory/page cache | PostgreSQL and OS-level repeated read performance. |
| Application-level short-lived cache | Safe master/config data only. |
| Frontend IndexedDB cache | POS offline product/pricing/tax/transaction support. |

Do not create generic PostgreSQL cache tables such as `backend_cache` or `query_cache`.

---

## Scaling POS lookup performance

POS must be fast for cashier workflows.

Priority order:

1. Exact barcode lookup using `tenant_id` + `barcode`.
2. Exact SKU lookup using `tenant_id` + `sku`.
3. Product name search only after barcode/SKU lookup.
4. Cache safe product bootstrap data where appropriate.
5. Revalidate price, tax, stock and session during sale completion.

Related files:

- [[06-frontend/pos-ui-rules]]
- [[06-frontend/offline-frontend-rules]]
- [[05-backend/caching-strategy]]

---

## Scaling e-commerce reads

E-commerce storefront reads may be more list/detail heavy than POS.

Optimize:

| Area | Strategy |
|---|---|
| Product listing | Tenant/channel/status indexes. |
| Product detail | Variant, image, price and tax lookup indexes. |
| Cart validation | Live validation before checkout. |
| Delivery fee | Indexed delivery methods/zones/rates. |
| Customer order history | Tenant/customer/date indexes. |

Cached listing data must not bypass checkout validation.

---

## Scaling reporting

Use existing reporting read models:

- `daily_sales_summaries`
- `daily_payment_summaries`
- `daily_inventory_summaries`
- `daily_discount_return_summaries`

These are acceptable because they have clear reporting grain.

They must not replace source records.

Reports must remain traceable to:

- `sales`
- `payments`
- `refunds`
- `stock_movements`
- `returns`
- `exchanges`

---

## Scaling offline sync

Offline sync can create bursts after reconnect.

Rules:

| Concern | Rule |
|---|---|
| Duplicate submissions | Use client IDs and DB-backed unique checks. |
| Batch processing | Track through `offline_sync_batches`. |
| Item processing | Track through `offline_sync_items`. |
| Sale/payment typed payload | Use typed queue tables as staging only. |
| Conflicts | Store in `offline_sync_conflicts`. |
| Diagnostics | Store in `offline_sync_audit_logs`. |

Do not trust stale product, stock, payment or session cache during sync acceptance.

---

## Scaling payment/refund safety

Payment scaling must not trade away correctness.

Required:

- Tenant-scoped idempotency key checks.
- DB validation of payment status.
- DB validation of refund amount limits.
- Allocation tables for sale/order/exchange/return relationships.
- Audit trail for sensitive actions.

Cache can show payment method configuration, not payment truth.

---

## Scaling stock operations

Stock operations must be ledger-safe.

Use:

- `inventory_balances` for current projection.
- `stock_movements` for immutable history.
- `stock_reservations` for online reservations.
- Proper indexes for tenant/outlet/variant/time.

Do not use long-lived stock cache as final availability.

---

## Risks to avoid

| Risk | Cause |
|---|---|
| Tenant leakage | Cache key missing `tenant_id`. |
| Overselling | Cached stock used as final truth. |
| Wrong totals | Cached stale price/tax used during commit. |
| Unauthorized action | Cached permission not revalidated for sensitive action. |
| Duplicate payment | Idempotency checked in cache instead of DB. |
| Sync corruption | Offline duplicate/conflict checks use cache. |
| Wrong reports | Generic cache treated as accounting data. |

---

## Related files

- [[02-architecture/system-overview]]
- [[02-architecture/offline-first-architecture]]
- [[03-data/indexing-strategy]]
- [[05-backend/caching-strategy]]
- [[04-api/idempotency-rules]]
