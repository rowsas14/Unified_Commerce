---
title: Data Dictionary Index
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [data-dictionary, index]
---

# Data Dictionary Index

Use this file to locate the correct entity reference before creating or changing database, backend, API, frontend, or test code.

---

## Table ownership index

| Entity area | File |
|---|---|
| Platform users, tenants, outlets, document numbers | [[entities/platform-tenant-entities]] |
| Staff users, roles, permissions, feature access | [[entities/identity-access-entities]] |
| Feature flags, tenant settings, UI themes | [[entities/configuration-entities]] |
| Brands, suppliers, categories, products, variants, attributes, images | [[entities/catalog-entities]] |
| Tax classes/rates and price lists/items | [[entities/pricing-tax-entities]] |
| Stock balances, movements, reservations, receiving, adjustments, transfers, stocktakes | [[entities/inventory-entities]] |
| Tills, POS devices, sessions, cash movements, sales | [[entities/pos-device-sales-entities]] |
| Customers, auth, OTP, wishlist, reviews, loyalty, carts, orders | [[entities/customer-ecommerce-entities]] |
| Delivery methods, zones, rates, deliveries, tracking | [[entities/fulfillment-entities]] |
| Payment methods, provider configs, payments, transactions, allocations, refunds | [[entities/payments-entities]] |
| Discount policies, requests, coupons, applications, redemptions | [[entities/discounts-coupons-entities]] |
| Returns, return lines, exchanges, allocations | [[entities/returns-exchanges-entities]] |
| Receipts, print logs, audit logs, offline sync | [[entities/receipts-audit-offline-entities]] |
| Daily reporting read models | [[entities/reporting-entities]] |
| Data import and AI onboarding schema gap | [[entities/data-import-ai-entities]] |

---

## Usage rules

- Feature specs must reference the relevant entity file.
- API specs must reference source tables and read models separately.
- Backend services must not update derived read models as if they are source data.
- Frontend state must not invent fields not present in the source entities or API contract.
- Tests must include tenant consistency and invalid FK relationship cases.

Related: [[database-overview]], [[entity-relationship-map]], [[../12-templates/entity-reference-template]].
