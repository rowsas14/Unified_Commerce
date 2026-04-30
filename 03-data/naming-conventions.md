---
title: Data Naming Conventions
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [naming, conventions]
---

# Data Naming Conventions

Naming consistency prevents confusion across database, EF Core, API contracts, and frontend state.

---

## Database naming rules

| Item | Rule | Example |
|---|---|---|
| Tables | plural snake_case | `product_variants` |
| Primary key | `id` unless reference table differs | `id uuid` |
| Foreign keys | singular parent name + `_id` | `tenant_id`, `product_id` |
| Timestamps | snake_case with `_at` | `created_at`, `updated_at` |
| Status columns | descriptive status name | `payment_status`, `fulfillment_status` |
| Codes | stable business/reference key | `code`, `feature_key` |
| Snapshots | suffix `_snapshot` or payload | `pricing_snapshot`, `payload` |

---

## Important naming boundaries

| Do not confuse | Correct distinction |
|---|---|
| `products` vs `product_variants` | Product is master; variant is sellable SKU/barcode. |
| `payments` vs allocations | Payment records money; allocation links money to sale/order. |
| `returns` vs `refunds` | Return is merchandise/document; refund is money. |
| `receipts` vs receipt templates | Template defines layout; receipt stores frozen output. |
| `feature_flags` vs entitlements | Entitlement enables tenant access; flag configures runtime behavior. |
| `offline_sync_items` vs source tables | Sync items are staging; accepted records go to sales/payments/etc. |

---

## EF/API mapping note

- Database uses snake_case.
- C# entities may use PascalCase property names mapped by EF Core.
- API JSON may use camelCase.
- Do not change database names to match frontend terminology.

Related: [[ef-core-implementation-notes]], [[schema-principles]].
