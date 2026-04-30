---
title: Migration Strategy
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [migrations, ef-core, postgres]
---

# Migration Strategy

Migrations must respect table dependencies, seed data, tenant boundaries, and production safety.

---

## Dependency order

| Order | Tables / area |
|---:|---|
| 1 | Platform and tenant foundation: `platform_users`, `tenants`, `outlets`, `outlet_addresses`, `document_sequences`. |
| 2 | Identity and access: `users`, `roles`, `permissions`, role mappings, features, entitlements. |
| 3 | Configuration: `feature_flags`, `tenant_settings`, `ui_themes`. |
| 4 | Catalog and tax/pricing reference: brands, suppliers, categories, tax, return policies, products, variants, attributes, prices. |
| 5 | Inventory: balances, movement types, movements, reservations, receiving, adjustments, transfers, stocktakes. |
| 6 | POS: tills, devices, sessions, cash, sales, sale lines. |
| 7 | Customers and e-commerce: customers, auth, OTP, addresses, wishlist, reviews, loyalty, carts, orders. |
| 8 | Fulfillment: delivery methods, zones, rates, deliveries, items, tracking. |
| 9 | Payments/refunds: methods, provider configs, payments, transactions, allocations, refunds. |
| 10 | Discounts, returns, exchanges, receipts, audit, offline sync. |
| 11 | Reporting read models. |

---

## Migration safety checklist

- [ ] Migration is reversible where practical.
- [ ] Data backfill is separated from schema change when large data is affected.
- [ ] New not-null columns have defaults or staged backfill.
- [ ] Tenant consistency constraints are validated before enforcement.
- [ ] Unique indexes are checked against existing data before deployment.
- [ ] Seed data migration is idempotent.
- [ ] Downtime risk is documented.
- [ ] API/backend/frontend compatibility is considered.

---

## Production warning

Do not add import/AI, peripheral assignment, or extra reporting tables directly from scope notes. Those require an approved database design update because the uploaded database design does not define their full schema.

Related: [[required-schema-extensions]], [[indexing-strategy]], [[ef-core-implementation-notes]].
