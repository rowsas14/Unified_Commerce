---
title: Database Alignment Rule
owner: Architecture + AI IDE
status: production-ready
last_reviewed: 2026-04-30
tags: [ai-ide, database, caching]
source: Unified Commerce production scope + database design
---

# Database Alignment Rule

## Purpose

This file tells AI IDE tools how to stay aligned with the approved Unified Commerce database design.

AI IDE tools must not create entities, migrations, repositories, APIs or frontend assumptions that are not supported by the approved schema or documented schema extension rules.

Read with:

- [[03-data/database-overview]]
- [[03-data/schema-principles]]
- [[03-data/required-schema-extensions]]
- [[03-data/indexing-strategy]]
- [[05-backend/caching-strategy]]

---

## Required behavior

- Check database overview and entity docs before coding.
- Respect source-of-truth tables.
- Respect tenant isolation.
- Respect PK/FK relationships.
- Respect unique constraints and idempotency rules.
- Do not create ad hoc tables without schema extension approval.
- Do not invent cache tables.

---

## Do not invent cache tables

AI IDE tools must not add tables like:

```text
cache
backend_cache
query_cache
product_cache
tenant_cache
cached_permissions
cached_stock
cached_payments
```

Reason:

- The approved database design already defines source-of-truth tables.
- Reporting read models already exist for dashboard reads.
- Performance should first be handled through indexes and query design.
- Backend cache does not require a PostgreSQL cache table.
- Generic cache tables can leak tenant data or create stale business decisions.

Correct action:

```text
Read [[05-backend/caching-strategy]] and [[03-data/indexing-strategy]] before adding any cache-related backend change.
```

---

## Source-of-truth rule

AI must preserve source-of-truth ownership.

| Area | Source of truth |
|---|---|
| Sale | `sales`, `sale_lines` |
| Order | `orders`, `order_items` |
| Payment | `payments`, `payment_transactions`, allocation tables |
| Refund | `refunds`, refund allocation tables |
| Stock | `inventory_balances`, `stock_movements`, `stock_reservations` |
| Return | `returns`, `return_lines` |
| Exchange | `exchanges`, `exchange_lines` |
| Receipt | `receipts`, `receipt_print_logs` |
| Offline sync | `offline_sync_batches`, `offline_sync_items`, typed queues/conflicts/audit logs |
| Audit | `audit_logs` |
| Feature access | `platform_features`, `tenant_feature_entitlements`, `role_feature_assignments`, `feature_flags` |

Do not duplicate these tables as cache versions.

---

## What to do when performance seems slow

AI IDE must follow this order:

1. Check whether the query is tenant-scoped.
2. Check [[03-data/indexing-strategy]].
3. Check if an existing read model exists.
4. Check if the backend can use safe short-lived cache.
5. Check if frontend cache or IndexedDB already covers the UX need.
6. Only then suggest a documented schema extension.

Do not jump directly to creating a table.

---

## Required schema extension process

If a new table is genuinely needed:

- [ ] Update [[03-data/required-schema-extensions]].
- [ ] Update the owning module docs under [[07-modules/README]].
- [ ] Update entity reference docs.
- [ ] Update API docs if endpoints are affected.
- [ ] Update backend docs and implementation checklist.
- [ ] Update tests.
- [ ] Get approval before migration.

---

## Cache-safe implementation reminders

When AI implements backend caching:

- Cache keys must include `tenant_id`.
- Outlet/channel/user context must be included when required.
- Payment, stock, refund, coupon, offline sync and document sequence checks must be DB-backed.
- Backend must revalidate before critical writes.
- Cache must not store secrets or OTP values.
- Cache must not bypass audit.

---

## Forbidden AI IDE output

Do not generate code or migrations that introduce:

```text
DbSet<BackendCache>
CacheEntity
ProductCacheEntity
TenantCacheEntity
CachedStockEntity
CachedPaymentStatusEntity
```

Do not generate repositories such as:

```text
ICacheRepository
BackendCacheRepository
QueryCacheRepository
```

unless the database design is officially updated to include such a table. Current decision: **not required**.

---

## Final rule

```text
Database performance improvements must align with approved schema, indexes, read models and backend caching strategy. AI must not create generic cache tables.
```
