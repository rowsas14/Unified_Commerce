---
title: Indexing Strategy
owner: Data Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [data, indexing, postgresql, performance, caching]
source: Unified Commerce production scope + database design
---

# Indexing Strategy

## Purpose

This file defines indexing rules for the Unified Commerce E-POS + E-Commerce SaaS database.

Indexes are the first database-level performance tool. Do not solve read performance by creating generic cache tables.

Read this with:

- [[03-data/schema-principles]]
- [[03-data/database-overview]]
- [[03-data/required-schema-extensions]]
- [[05-backend/caching-strategy]]
- [[02-architecture/scalability-considerations]]

---

## Core indexing principles

| Principle | Rule |
|---|---|
| Tenant-first | Tenant-owned query indexes should usually start with `tenant_id`. |
| Outlet-aware | POS/inventory queries often need `tenant_id`, `outlet_id`. |
| Channel-aware | Pricing, settings and product visibility often need channel. |
| Time-aware | Sales, payments, reports and audit queries need date/time indexes. |
| Dedupe-aware | Offline and payment idempotency require unique/filtered indexes. |
| Status-aware | Order, sale, delivery and sync screens commonly filter by status. |
| No generic cache table | Improve queries using indexes/read models first. |

---

## Cache-safe query optimization

Caching must not hide missing indexes.

Before adding application cache, confirm the related PostgreSQL query has the right index.

| Query area | Required index direction |
|---|---|
| Product lookup | `products(tenant_id, status)` and `product_variants(tenant_id, sku/barcode)` |
| POS barcode scan | `product_variants(tenant_id, barcode)` partial where barcode is not null |
| POS SKU search | `product_variants(tenant_id, sku)` |
| Category browse | `categories(tenant_id, parent_id, sort_order)` |
| Product channel listing | `products(tenant_id, is_sellable_pos/is_sellable_online, status)` |
| Price resolution | `price_lists(tenant_id, channel, currency, is_active)` plus `price_list_items(tenant_id, price_list_id, variant_id)` |
| Tax calculation | `tax_rates(tenant_id, code, starts_at)` and `tax_class_rates(tenant_id, tax_class_id, starts_at)` |
| Feature config | `feature_flags(tenant_id, feature_id, scope)` |
| Tenant settings | `tenant_settings(tenant_id, setting_key, scope)` |
| Inventory lookup | `inventory_balances(tenant_id, outlet_id, variant_id)` |
| Payment idempotency | `payments(tenant_id, idempotency_key)` partial where key is not null |
| Offline dedupe | `offline_sync_items(tenant_id, device_id, entity_type, client_entity_id)` |

---

## Tenant and outlet indexes

Most production queries are tenant-scoped.

Recommended index shapes:

```text
(tenant_id, status)
(tenant_id, created_at)
(tenant_id, business_date)
(tenant_id, outlet_id, business_date)
(tenant_id, outlet_id, status)
```

Use these for:

- `sales`
- `orders`
- `payments`
- `refunds`
- `returns`
- `exchanges`
- `deliveries`
- `till_sessions`
- `stock_movements`

---

## Catalog indexes

Catalog must support both POS speed and e-commerce browsing.

| Table | Index need |
|---|---|
| `products` | Tenant + status + channel sellable flags. |
| `product_variants` | Tenant + SKU, tenant + barcode. |
| `categories` | Tenant + parent + sort order. |
| `brands` | Tenant + name/code uniqueness. |
| `product_attributes` | Tenant + code. |
| `attribute_values` | Tenant + attribute + value code. |
| `product_images` | Tenant + product + variant + sort order. |
| `return_policies` | Tenant + code. |

POS barcode scans must not perform wide text search before exact barcode/SKU lookup.

---

## Pricing and tax indexes

Pricing and tax are cacheable only with careful invalidation, so indexes must still support live validation.

| Table | Index need |
|---|---|
| `price_lists` | `tenant_id`, `channel`, `currency`, `is_active`, effective date. |
| `price_list_items` | `tenant_id`, `price_list_id`, `variant_id`, nullable `outlet_id`. |
| `tax_classes` | `tenant_id`, `code`. |
| `tax_rates` | `tenant_id`, `code`, `starts_at`, `ends_at`. |
| `tax_class_rates` | `tenant_id`, `tax_class_id`, `starts_at`, `ends_at`. |

Price list overlap rules must be enforced by service logic or database constraints where practical.

---

## Inventory indexes

Inventory is sensitive. Avoid long-lived cache; use strong indexes.

| Table | Index need |
|---|---|
| `inventory_balances` | Unique `tenant_id`, `outlet_id`, `variant_id`. |
| `inventory_channel_allocations` | `tenant_id`, `outlet_id`, `variant_id`, `channel`. |
| `stock_movements` | `tenant_id`, `outlet_id`, `variant_id`, `occurred_at`. |
| `stock_reservations` | `tenant_id`, `order_id`, `variant_id`, `status`. |
| `stock_transfers` | `tenant_id`, source/destination outlets, status. |
| `stocktakes` | `tenant_id`, `outlet_id`, `status`. |

Stock movement queries can grow quickly; date and variant indexes matter.

---

## Payment and sync indexes

Payment and offline sync must not depend on cache for correctness.

| Table | Index need |
|---|---|
| `payments` | Tenant idempotency key, tenant/device/client payment id. |
| `payment_transactions` | Tenant + payment id + provider transaction id. |
| `refunds` | Tenant + original payment id. |
| `offline_sync_batches` | Tenant + device + status + started time. |
| `offline_sync_items` | Tenant + device + entity type + client entity id. |
| `offline_sale_sync_queue` | Tenant + device + client sale id. |
| `offline_payment_sync_queue` | Tenant + device + client payment id. |
| `offline_sync_conflicts` | Tenant + device + resolution status. |

Duplicate prevention must be DB-backed using unique constraints or indexed lookup.

---

## Reporting indexes

Existing read models must support dashboard filters.

| Read model | Index need |
|---|---|
| `daily_sales_summaries` | Tenant + scope + channel + business date. |
| `daily_payment_summaries` | Tenant + scope + business date + payment method. |
| `daily_inventory_summaries` | Tenant + outlet + variant + business date. |
| `daily_discount_return_summaries` | Tenant + scope + channel + business date. |

Read models are optimized summaries, not cache tables.

---

## Anti-patterns

Do not add indexes blindly for every column.

Do not create generic cache tables to avoid indexing work.

Do not index sensitive JSONB payloads unless a documented query requires it.

Do not use application cache to hide missing tenant filters.

Do not rely on cached stock, payment or sync status for final transaction decisions.

---

## Review checklist

Before a production query is accepted:

- [ ] Query has tenant filter where tenant-owned.
- [ ] Query uses outlet/channel/user context where required.
- [ ] Query has supporting index or documented reason not to.
- [ ] Query does not scan all tenants.
- [ ] Query does not depend on stale cache for correctness.
- [ ] Query supports expected POS speed where used in cashier flow.
- [ ] Query supports e-commerce list/detail/checkout behavior where used online.
- [ ] Query does not bypass source-of-truth tables.
