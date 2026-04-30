---
title: Schema Principles
owner: Data Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [data, schema, source-of-truth, caching]
source: Unified Commerce production scope + database design
---

# Schema Principles

## Purpose

This file defines schema-level principles for the Unified Commerce E-POS + E-Commerce SaaS database.

The database must support production tenant isolation, POS speed, e-commerce ordering, payments, stock movements, returns, exchanges, offline sync, reporting and audit.

Read with:

- [[03-data/database-overview]]
- [[03-data/entity-relationship-map]]
- [[03-data/indexing-strategy]]
- [[03-data/tenant-consistency-rules]]
- [[05-backend/caching-strategy]]

---

## Core principles

- Use UUID primary keys for business entities.
- Use `tenant_id` on tenant-owned tables.
- Use immutable ledgers for stock and loyalty points.
- Use checked strings or reference tables deliberately.
- Use JSONB for configurable settings, not core transaction facts.
- Use partial unique indexes for nullable scoped uniqueness.
- Use source-of-truth tables for transactions.
- Use read models for reporting, not financial truth.
- Do not create generic cache tables.

---

## Cache is not source of truth

Caching is a performance concern, not a schema ownership pattern.

Do not create duplicate cache tables for:

```text
products
prices
stock
payments
orders
sales
permissions
tenant settings
offline sync state
```

Correct approach:

```text
source table -> indexed query/read model -> optional short-lived application cache
```

Incorrect approach:

```text
source table -> copied cache table -> business logic reads cache as truth
```

---

## Source-of-truth examples

| Business area | Source-of-truth tables |
|---|---|
| Tenant setup | `tenants`, `outlets`, `outlet_addresses`, `document_sequences` |
| Staff access | `users`, `roles`, `permissions`, `role_permissions`, `tenant_user_roles`, `outlet_user_roles` |
| Feature access | `platform_features`, `tenant_feature_entitlements`, `role_feature_assignments`, `feature_flags` |
| Catalog | `products`, `product_variants`, `categories`, `brands`, `product_attributes`, `attribute_values` |
| Pricing | `price_lists`, `price_list_items` |
| Tax | `tax_classes`, `tax_rates`, `tax_class_rates` |
| Inventory | `inventory_balances`, `stock_movements`, `stock_reservations` |
| POS sale | `sales`, `sale_lines` |
| Payment | `payments`, `payment_transactions`, allocation tables |
| Refund | `refunds`, refund allocation tables |
| Order | `orders`, `order_items`, `order_status_history` |
| Return/exchange | `returns`, `return_lines`, `exchanges`, `exchange_lines` |
| Receipt | `receipts`, `receipt_print_logs` |
| Offline sync | `offline_sync_batches`, `offline_sync_items`, typed queue/conflict/audit tables |
| Audit | `audit_logs` |

---

## Read models are allowed

Reporting read models are allowed when they have clear grain and purpose.

Existing reporting read models:

- `daily_sales_summaries`
- `daily_payment_summaries`
- `daily_inventory_summaries`
- `daily_discount_return_summaries`

These are not generic cache tables.

They are derived summaries used for dashboard/reporting reads.

They must be traceable back to transaction data.

---

## Offline queues are not cache tables

Offline sync tables are staging and reconciliation records.

They must not be treated as cache.

| Table | Purpose |
|---|---|
| `offline_sync_batches` | One reconnect/sync attempt from POS device. |
| `offline_sync_items` | Generic payload item received from device. |
| `offline_sale_sync_queue` | Typed sale staging extension. |
| `offline_payment_sync_queue` | Typed payment staging extension. |
| `offline_sync_conflicts` | Conflict records requiring resolution. |
| `offline_sync_audit_logs` | Technical sync diagnostics. |

Accepted offline transactions must land in source tables such as `sales`, `payments`, `stock_movements`, and `receipts`.

---

## JSONB rule

JSONB is allowed for:

- Tenant settings payloads.
- Feature flag configuration.
- UI theme tokens.
- Pricing snapshots.
- Receipt payload snapshots.
- Provider raw payloads.
- Offline sync payloads.

JSONB must not replace relational tables for:

- Sales lines.
- Order items.
- Payment allocation.
- Stock movements.
- Return lines.
- Exchange lines.
- Tenant/user/role relationships.

---

## Tenant consistency rule

Every FK relationship across tenant-owned tables must maintain the same tenant context.

Examples:

- `sale_lines.sale_id` and `sale_lines.variant_id` must belong to same tenant.
- `order_items.order_id` and `order_items.variant_id` must belong to same tenant.
- `price_list_items.price_list_id` and `variant_id` must belong to same tenant.
- `wishlist_items.wishlist_id` and `variant_id` must belong to same tenant.
- Offline sync device, outlet and batch must belong to same tenant.

See [[03-data/tenant-consistency-rules]].

---

## Schema change checklist

Before adding a new table:

- [ ] It is required by scope or approved schema extension.
- [ ] It is not a generic cache table.
- [ ] It has a clear owning module.
- [ ] It has PK and FK rules.
- [ ] It defines tenant isolation.
- [ ] It defines source-of-truth relationship.
- [ ] It defines indexes.
- [ ] It defines audit/security implications.
- [ ] It is added to entity reference docs.
- [ ] It is included in migration strategy.

---

## Related files

- [[03-data/database-overview]]
- [[03-data/required-schema-extensions]]
- [[03-data/indexing-strategy]]
- [[05-backend/caching-strategy]]
- [[14-ai-ide-rules/database-alignment-rule]]
