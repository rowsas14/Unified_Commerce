---
title: Entity Reference Index
folder: 03-data/entities
status: production-ready
owner: Architecture / Backend / Data
tags: [entities, index]
---

# Entity Reference Index

This folder contains table-level entity references grouped by production module area.

Use these files before writing migrations, EF Core mappings, API specs, backend services, frontend types, test cases, or AI IDE implementation prompts.

---

## Entity files

| File | Scope |
|---|---|
| [[platform-tenant-entities]] | These tables define the platform-side user boundary, tenant business account, outlet/location model, outlet address, and business document number generation. |
| [[identity-access-entities]] | These tables implement tenant-bound staff identity, tenant and outlet role assignment, platform permission catalog, platform feature entitlements, and role-to-feature access. |
| [[configuration-entities]] | These tables store tenant-side runtime configuration, scoped settings, feature flags, and UI theme tokens. |
| [[catalog-entities]] | These tables define the shared POS and E-Commerce product catalog, variant model, attributes, brands, suppliers, images, and return policy classification. |
| [[pricing-tax-entities]] | These tables define tax classes, effective tax rates, class-rate mappings, channel price lists, and variant price rows. |
| [[inventory-entities]] | These tables implement outlet-wise stock balances, stock ledger movements, reservations, supplier receiving, adjustments, transfers, and stocktakes. |
| [[pos-device-sales-entities]] | These tables define tills, registered POS devices, cashier/till sessions, cash movements, denomination counts, POS sales, and POS sale lines. |
| [[customer-ecommerce-entities]] | These tables define tenant-scoped customers, customer online authentication, OTP, addresses, wishlists, reviews, loyalty, carts, orders, order lines, address snapshots, and order status history. |
| [[fulfillment-entities]] | These tables support delivery and pickup methods, delivery zones, zone rates, fulfillment headers, delivery items, and tracking events. |
| [[payments-entities]] | These tables define payment method references, tenant payment methods, gateway configuration, unified payment records, provider event logs, allocations to sales/orders, and refunds. |
| [[discounts-coupons-entities]] | These tables define discount type/scope references, tenant policies, approval requests, coupons, actual applied discounts, and coupon redemptions. |
| [[returns-exchanges-entities]] | These tables support POS and E-Commerce returns, return lines, refund allocation, exchange headers, exchange lines, and exchange payment/refund allocations. |
| [[receipts-audit-offline-entities]] | These tables store receipt templates and outputs, print logs, immutable business audit logs, offline sync batches/items, typed sale/payment staging queues, conflicts, and technical sync audit logs. |
| [[reporting-entities]] | These tables are pre-aggregated read models for daily sales, payment, inventory, discount, return, and exchange reporting. |
| [[data-import-ai-entities]] | The production scope includes CSV/Excel import and AI-assisted onboarding, but the uploaded database design does not define dedicated import or AI staging tables. This file prevents accidental invention of schema during implementation. |

Related: [[../data-dictionary-index]], [[../database-overview]].
