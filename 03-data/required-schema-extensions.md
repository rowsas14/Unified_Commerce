---
title: Required Schema Extensions
owner: Data Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [database, schema, extensions, caching]
source: Unified Commerce production scope + database design
---

# Required Schema Extensions

## Purpose

This file lists schema extensions required to align the production scope with the uploaded production database design.

It also records a clear negative decision: **generic cache tables are not required**.

Read this with:

- [[03-data/database-overview]]
- [[03-data/schema-principles]]
- [[03-data/indexing-strategy]]
- [[05-backend/caching-strategy]]
- [[14-ai-ide-rules/database-alignment-rule]]

---

## Extension decision rules

A new table may be added only when:

- The uploaded scope requires a durable business workflow that the current schema does not support.
- The table has a clear owner module.
- The table is not duplicating an existing source-of-truth table.
- The related feature spec and entity reference are updated.
- The migration order is understood.
- Tenant isolation is defined where the table is tenant-owned.

A new table must not be added only because a query is slow.

Slow reads should first be handled using:

- Proper indexes.
- Query design.
- Existing reporting read models.
- Short-lived application cache where safe.
- Frontend offline cache where already defined for POS.

---

## Required additions from scope/schema alignment

The uploaded database design is strong, but the production scope requires a few additional table groups to avoid weak JSON-only or undocumented behavior.

| Area | Tables | Why |
|---|---|---|
| Data import | `data_import_jobs`, `data_import_files`, `data_import_records`, `data_import_validation_errors`, `data_import_duplicate_candidates` | Product/customer imports need reviewable staging, validation and duplicate checks. |
| AI extraction | `ai_extraction_jobs`, `ai_extraction_results`, `ai_review_decisions` | AI output must be reviewed before saving source records. |
| POS peripherals | `pos_peripherals`, `pos_device_peripheral_assignments`, `printer_profiles`, `printer_test_logs`, `cash_drawer_devices` | Scope requires terminal, printer, scanner and drawer assignment. |
| Tax policy | `tax_calculation_policies` | Inclusive/exclusive pricing, discount-tax behavior and rounding must be consistent. |
| Role templates | `role_templates`, `role_template_permissions`, `role_template_feature_assignments` | Tenant onboarding needs configurable default role setup. |
| Reporting | `daily_tax_summaries`, `daily_product_sales_summaries`, `daily_category_sales_summaries`, `daily_cash_session_summaries`, `offline_sync_daily_summaries`, `low_stock_snapshots` | Production dashboards need fast, explainable read models. |
| Product publishing | `product_channel_publication`, `product_seo_metadata` | Storefront publishing and SEO need stronger state than a simple boolean. |

These additions must be implemented only through approved migrations after module and entity documentation are updated.

---

## Not required: generic cache tables

Do not add cache tables such as:

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

- The uploaded database design already has source-of-truth tables.
- Reporting read models already exist for dashboard-style reads.
- PostgreSQL indexing should handle lookup performance.
- Backend application cache does not need a database table.
- A generic cache table can create stale data and tenant leakage.

Correct approach:

```text
Use source tables + indexes + read models + safe application cache.
```

---

## Existing read models are not generic cache tables

The following database tables are valid reporting read models:

- `daily_sales_summaries`
- `daily_payment_summaries`
- `daily_inventory_summaries`
- `daily_discount_return_summaries`

They are allowed because they have clear reporting purpose and defined grain.

They are not final financial source of truth.

Source-of-truth remains in:

- `sales`
- `sale_lines`
- `orders`
- `order_items`
- `payments`
- `refunds`
- `stock_movements`
- `returns`
- `exchanges`

---

## Cache-related schema rule

If a developer thinks a new table is needed for caching, follow this order:

1. Check [[05-backend/caching-strategy]].
2. Check [[03-data/indexing-strategy]].
3. Check whether an existing source table already owns the data.
4. Check whether an existing reporting read model can support the report.
5. Check whether a safer index or query change solves the issue.
6. Raise a schema decision only if the table is a true business/read model, not generic cache.

---

## Examples of allowed performance tables

Allowed only when documented as read models:

| Table type | Allowed? | Reason |
|---|---:|---|
| Daily sales summary | Yes | Clear reporting grain. |
| Daily payment summary | Yes | Clear reporting grain. |
| Daily tax summary | Yes, if approved | Clear report requirement. |
| Low stock snapshot | Yes, if approved | Operational read model. |
| Generic query cache | No | No business meaning. |
| Cached stock balance duplicate | No | Duplicates `inventory_balances`. |
| Cached payment status | No | Duplicates sensitive payment state. |

---

## Migration rule

Any required schema extension must include:

- [ ] Module owner.
- [ ] Table purpose.
- [ ] PK and FK rules.
- [ ] Tenant isolation rule.
- [ ] Indexes.
- [ ] Source-of-truth relationship.
- [ ] API/backend impact.
- [ ] Security/audit impact.
- [ ] Feature spec update.
- [ ] Test case update.

---

## Final caching schema decision

```text
No generic cache table is required for PostgreSQL-based backend caching.
```

This decision must remain unless the database design is formally changed.
