---
title: 03 Data Documentation Index
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [data, database, postgres, ef-core]
---

# 03 Data Documentation Index

This folder is the data authority for the production-ready Unified Commerce E-POS + E-Commerce SaaS system.

It translates the uploaded production scope and database design into developer-readable database rules, entity references, migration guidance, EF Core implementation notes, indexing guidance, offline sync design, and tenant isolation rules.

---

## Start here

| Step | File | Use it for |
|---:|---|---|
| 1 | [[database-overview]] | Understand schema groups and source-of-truth boundaries. |
| 2 | [[schema-principles]] | Read non-negotiable schema design rules. |
| 3 | [[tenant-consistency-rules]] | Understand tenant, outlet, user, device, and customer isolation. |
| 4 | [[entity-relationship-map]] | See cross-module table relationships. |
| 5 | [[data-dictionary-index]] | Find the owning entity reference file. |
| 6 | `entities/*` | Read PK/FK/important-attribute details for every table group. |
| 7 | [[indexing-strategy]] | Apply unique/index rules. |
| 8 | [[migration-strategy]] | Order migrations safely. |
| 9 | [[ef-core-implementation-notes]] | Implement PostgreSQL and EF Core mappings. |

---

## Entity reference map

| Group | Entity file | Table scope |
|---|---|---|
| Platform and Tenant Foundation | [[entities/platform-tenant-entities]] | platform_users, tenants, outlets, outlet_addresses, document_sequences |
| Identity, RBAC, and Feature Access | [[entities/identity-access-entities]] | users, roles, permissions, role_permissions, tenant_user_roles, outlet_user_roles, platform_features, tenant_feature_entitlements, role_feature_assignments |
| Tenant Runtime Configuration | [[entities/configuration-entities]] | feature_flags, tenant_settings, ui_themes |
| Catalog | [[entities/catalog-entities]] | brands, suppliers, supplier_addresses, categories, return_policies, products, product_variants, attributes, attribute templates, supplier and image mappings |
| Pricing and Tax | [[entities/pricing-tax-entities]] | tax_classes, tax_rates, tax_class_rates, price_lists, price_list_items |
| Inventory and Stock Control | [[entities/inventory-entities]] | inventory_balances, stock_movements, reservations, purchase receipts, adjustments, transfers, stocktakes |
| POS Device, Session, and Sales | [[entities/pos-device-sales-entities]] | tills, pos_devices, till_sessions, cash movements, sales, sale_lines |
| Customer and E-Commerce | [[entities/customer-ecommerce-entities]] | customers, auth, OTP, addresses, wishlists, reviews, loyalty, carts, orders, order history |
| Fulfillment, Pickup, and Delivery | [[entities/fulfillment-entities]] | delivery_methods, delivery_zones, rates, deliveries, delivery_items, tracking |
| Payments, Refunds, and Allocations | [[entities/payments-entities]] | payment method types, tenant methods, provider configs, payments, transactions, allocations, refunds |
| Discounts and Coupons | [[entities/discounts-coupons-entities]] | discount types/scopes, policies, requests, coupons, applications, redemptions |
| Returns and Exchanges | [[entities/returns-exchanges-entities]] | return reasons, returns, return lines, refund allocations, exchanges, exchange lines/allocations |
| Receipts, Audit, and Offline Sync | [[entities/receipts-audit-offline-entities]] | receipt templates, receipts, print logs, audit logs, offline sync batches/items/queues/conflicts |
| Reporting Read Models | [[entities/reporting-entities]] | daily_sales_summaries, daily_payment_summaries, daily_inventory_summaries, daily_discount_return_summaries |
| Data Import and AI-Assisted Onboarding | [[entities/data-import-ai-entities]] | scope exists; dedicated tables are not in uploaded database design |


---

## Non-negotiable data rules

- Tenant isolation is mandatory for tenant-owned data.
- Variant is the sellable unit for POS, E-Commerce, stock, pricing, returns, and exchanges.
- Inventory changes are ledgered through `stock_movements`.
- Payments are unified records and allocated to sales, orders, returns, or exchanges.
- Returns and exchanges are separate documents, not negative sales rows.
- Receipts are frozen generated outputs.
- Offline sync queues are staging only; accepted records land in source tables.
- Reporting tables are read models and must be rebuildable from source transactions.
- Backend is final authority for pricing, tax, discount, payment, stock, and access validation.

---

## Related folders

- [[00-start-here/README]]
- [[01-product/project-scope]]
- [[02-architecture/system-overview]]
- [[04-api/api-overview]]
- [[05-backend/backend-overview]]
- [[06-frontend/frontend-overview]]
- [[07-modules/README]]
- [[12-templates/entity-reference-template]]
