---
title: Enum and Reference Data Policy
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [enums, reference-data]
---

# Enum and Reference Data Policy

The uploaded database design uses a mix of seeded reference tables and checked string status columns. Follow this policy to avoid inconsistent enum handling.

---

## Seeded reference tables

| Table | Purpose |
|---|---|
| `permissions` | Platform permission catalog. |
| `platform_features` | Platform feature catalog. |
| `stock_movement_types` | Inventory movement behavior and direction. |
| `cash_movement_types` | Non-sale cash movement types. |
| `payment_method_types` | Cash, card, QR, wallet, bank transfer, gift card. |
| `discount_types` | Percentage, fixed amount, price override. |
| `discount_scopes` | Line, sale, order. |
| `otp_channels` | Email, SMS, WhatsApp. |
| `otp_purposes` | Login, signup, reset password, verification, MFA. |

---

## Checked string status columns

| Area | Example columns |
|---|---|
| Tenant/user state | `tenants.status`, `users.status` |
| Product/catalog state | `products.status`, `product_variants.status` |
| POS workflow | `sales.status`, `till_sessions.status` |
| E-Commerce workflow | `orders.order_status`, `payment_status`, `fulfillment_status` |
| Payments/refunds | `payments.payment_status`, `refunds.refund_status` |
| Returns/exchanges | `returns.status`, `exchanges.status` |
| Offline sync | `offline_sync_batches.status`, `offline_sync_items.sync_status` |

---

## Rules

- Use reference tables when values have behavior, seed ownership, UI labels, or relationships.
- Use checked strings when values are simple workflow states defined by the database design.
- Do not create separate status tables unless the approved database design changes.
- Keep C# enums aligned with database check values.
- Status transition rules belong in application services and workflow docs, not only enum definitions.

Related: [[seed-data-strategy]], [[entities/customer-ecommerce-entities]], [[entities/payments-entities]].
