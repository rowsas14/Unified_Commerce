---
title: Caching Strategy
owner: Backend Lead
status: production-ready
last_reviewed: 2026-04-30
tags: [backend, caching, postgresql, performance]
source: Unified Commerce production scope + database design + backend architecture
---

# Caching Strategy

## Purpose

This file defines how backend caching must be handled for the Unified Commerce E-POS + E-Commerce SaaS system.

The decision is clear: **do not create a generic cache table**.

Caching must improve read performance without creating a second source of truth for tenant data, product data, pricing, stock, payments, offline sync, or reports.

This file must be read with:

- [[03-data/indexing-strategy]]
- [[03-data/schema-principles]]
- [[03-data/required-schema-extensions]]
- [[04-api/concurrency-rules]]
- [[04-api/idempotency-rules]]
- [[02-architecture/scalability-considerations]]
- [[02-architecture/offline-first-architecture]]
- [[06-frontend/frontend-caching-rules]]
- [[14-ai-ide-rules/database-alignment-rule]]

---

## Core decision

| Question | Decision |
|---|---|
| Is a new generic cache table required? | No |
| Should `backend_cache`, `query_cache`, `product_cache`, or `tenant_cache` be created? | No |
| Should source tables be duplicated for cache speed? | No |
| Should PostgreSQL indexes and read models be used first? | Yes |
| Should backend cache be final authority? | No |
| Should critical writes revalidate against PostgreSQL? | Always |

Correct statement:

```text
Backend caching is allowed only as a performance layer. PostgreSQL source tables remain the authority.
```

---

## What PostgreSQL already provides

PostgreSQL can support backend performance without a custom cache table through:

| Mechanism | Use |
|---|---|
| Indexes | Fast lookup for tenant, barcode, SKU, status, date and idempotency queries. |
| Query planning | Efficient execution for repeated query shapes. |
| Database memory and OS page cache | Frequently accessed pages can remain memory-resident. |
| Read models | Existing reporting summary tables support dashboard reads. |
| Transaction locks | Required for sequences, payments, coupon usage and stock operations. |
| Constraints | Prevent invalid duplicate or cross-tenant state. |

Do not replace these with a generic cache table.

---

## Data that may be cached

Cache only read-heavy, low-risk, tenant-scoped data.

| Data | Cache allowed | Scope key required | Notes |
|---|---:|---|---|
| `platform_features` | Yes | feature key | Platform feature catalog changes rarely. |
| `tenant_feature_entitlements` | Yes, short-lived | `tenant_id` | Invalidate after platform admin changes. |
| `role_feature_assignments` | Yes, short-lived | `tenant_id`, `role_id` | Stale access is risky; keep short. |
| `permissions` | Yes | permission code | Platform-owned permission catalog. |
| `role_permissions` | Yes, short-lived | `tenant_id`, `role_id` | Recheck before sensitive action. |
| `feature_flags` | Yes | `tenant_id`, scope | Runtime feature config. |
| `tenant_settings` | Yes | `tenant_id`, scope | Tenant/outlet/channel setting reads. |
| `ui_themes` | Yes | `tenant_id` | Safe for UI rendering. |
| `categories` | Yes | `tenant_id` | Catalog navigation. |
| `brands` | Yes | `tenant_id` | Catalog filtering. |
| `product_attributes` | Yes | `tenant_id` | Catalog forms and filters. |
| Product search/list reads | Yes, short-lived | `tenant_id`, channel | Must respect status and channel flags. |
| SKU/barcode lookup | Yes, short-lived | `tenant_id`, barcode/SKU | Useful for POS speed. |
| `price_lists` and `price_list_items` | Yes, careful TTL | `tenant_id`, channel, currency | Must respect effective dates and priority. |
| `tax_classes`, `tax_rates`, `tax_class_rates` | Yes, careful TTL | `tenant_id` | Must invalidate when tax setup changes. |
| `return_policies` | Yes | `tenant_id` | Used for return validation. |
| `payment_method_types` | Yes | method code | Reference data. |
| `tenant_payment_methods` | Yes, short-lived | `tenant_id` | Do not cache secrets. |
| `delivery_methods`, `delivery_zones`, `delivery_zone_rates` | Yes | `tenant_id` | Checkout and fulfillment reads. |
| Reporting read models | Yes | `tenant_id`, date, outlet/channel | Existing read models are for dashboards. |

---

## Data that must not be cached as final truth

The following data can be displayed from a recent read, but must not be trusted from cache during final writes.

| Data | Why not cache as truth |
|---|---|
| `sales`, `sale_lines` | Completed sale creation must be transactional. |
| `orders`, `order_items` | Status and payment/fulfillment state must be current. |
| `payments`, `payment_transactions` | Payment state and idempotency must be exact. |
| `refunds` | Refund limits must check captured payment state. |
| `inventory_balances` | Stale stock can cause overselling. |
| `stock_movements` | Immutable stock ledger. |
| `stock_reservations` | Reservation expiry and available stock are time-sensitive. |
| `till_sessions` | Cashier session must be current. |
| `cash_movements` | Cash reconciliation must be exact. |
| `discount_requests` | Approval state must not be stale. |
| `coupons.used_count` and `coupon_redemptions` | Usage limits require transactional control. |
| `otp_verifications` | Security-sensitive and expiry-sensitive. |
| `document_sequences` | Must use row-level locking; never cache next number. |
| `offline_sync_batches`, `offline_sync_items`, `offline_sync_conflicts` | Sync dedupe/conflict checks must be DB-backed. |
| `audit_logs` | Immutable audit source. |

---

## Required cache key rules

Every tenant-owned backend cache key must include tenant context.

Minimum cache key shape:

```text
{tenant_id}:{module}:{resource}:{parameters}
```

Examples:

```text
tenant-123:catalog:barcode:8901234567890
tenant-123:settings:outlet:outlet-7:pos
tenant-123:feature-access:role:manager
tenant-123:pricing:pos:LKR:variant-55
```

When applicable, also include:

| Context | Required when |
|---|---|
| `outlet_id` | POS, stock, outlet settings, till/session operations. |
| `channel` | POS vs e-commerce pricing, settings and visibility. |
| `user_id` | User-scoped settings or feature flags. |
| `role_id` | Role permission/feature checks. |
| `currency` | Price list and payment displays. |

Never reuse cache across tenants.

---

## Required invalidation rules

Invalidate or refresh cache after writes to these areas:

| Write area | Cache to invalidate |
|---|---|
| Product or variant update | Product search, barcode lookup, channel listing. |
| Category/brand/attribute update | Catalog navigation/filter caches. |
| Price list update | Pricing cache for affected tenant/channel/currency. |
| Tax class/rate update | Tax calculation cache. |
| Tenant setting update | Settings cache for affected tenant/outlet/channel/user scope. |
| Feature flag update | Feature config cache. |
| Tenant entitlement update | Tenant feature access cache. |
| Role permission update | User/role permission cache. |
| Role feature assignment update | Feature access cache. |
| Payment method update | Payment method cache. |
| Return policy update | Return validation cache. |
| Delivery zone/rate update | Checkout/delivery fee cache. |

If invalidation is not implemented, use a short TTL and force DB validation before critical operations.

---

## Transaction safety rule

Cache may support display and lookup speed, but these operations must validate from PostgreSQL during commit:

- POS sale completion
- Order placement
- Payment capture or refund
- Coupon redemption
- Discount approval/application
- Return creation
- Exchange completion
- Stock adjustment
- Stock reservation
- Offline sync acceptance
- Receipt generation
- Till session open/close
- Document sequence allocation

Backend service methods must treat cached data as advisory.

---

## Recommended implementation approach

| Step | Rule |
|---|---|
| 1 | Add proper indexes first. |
| 2 | Use existing reporting read models for dashboard reads. |
| 3 | Add short-lived application-level cache only for safe read-heavy data. |
| 4 | Include tenant/outlet/channel/user context in cache keys. |
| 5 | Invalidate after writes where possible. |
| 6 | Revalidate source tables before critical writes. |
| 7 | Never store secrets or payment-sensitive values in cache. |
| 8 | Never add generic cache tables without approved schema change. |

---

## Backend service checklist

Before adding cache to a service, confirm:

- [ ] The data is read-heavy.
- [ ] The data is not a financial or stock source of truth.
- [ ] The cache key includes `tenant_id`.
- [ ] Outlet/channel/user context is included when needed.
- [ ] The write path invalidates or bypasses the cache.
- [ ] Sensitive operations still validate from PostgreSQL.
- [ ] The cache does not store secrets, OTPs, raw payment payloads or private keys.
- [ ] Tests cover stale cache behavior.

---

## Anti-patterns

Do not implement:

```text
backend_cache
query_cache
product_cache
tenant_cache
cache_entries
cached_permissions
cached_stock
cached_payments
```

Do not use cache to:

- Allocate document numbers.
- Confirm final stock availability.
- Confirm payment idempotency.
- Confirm offline sync duplicates.
- Confirm coupon usage limits.
- Override tenant isolation.
- Avoid writing audit logs.

---

## Related files

- [[03-data/indexing-strategy]]
- [[03-data/schema-principles]]
- [[03-data/required-schema-extensions]]
- [[04-api/concurrency-rules]]
- [[04-api/idempotency-rules]]
- [[06-frontend/frontend-caching-rules]]
- [[14-ai-ide-rules/database-alignment-rule]]
