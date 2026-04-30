---
title: Unified Commerce 2nd Brain
owner: Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [unified-commerce, 2nd-brain, production]
source: Unified Commerce production scope + database design
---

# Unified Commerce 2nd Brain

This vault is the production documentation baseline for a **multi-tenant Unified Commerce SaaS platform** covering E-POS, E-Commerce, offline POS, inventory, payments, refunds, returns, exchanges, fulfillment, loyalty, reporting, audit, tenant configuration and AI-assisted onboarding.

This is not a basic POS, MVP, demo, or small e-commerce project. Treat every workflow as production business software: tenant-isolated, auditable, permission-controlled and recoverable.

## Start here

1. Read [00-start-here/README.md](00-start-here/README.md).
2. Confirm the production scope in [01-product/project-scope.md](01-product/project-scope.md).
3. Understand the system shape in [02-architecture/system-overview.md](02-architecture/system-overview.md).
4. Check the data baseline in [03-data/database-overview.md](03-data/database-overview.md).
5. Before coding, follow [14-ai-ide-rules/fullstack-feature-implementation-rule.md](14-ai-ide-rules/fullstack-feature-implementation-rule.md).

## Production module map

| Area | Primary folder | Purpose |
| --- | --- | --- |
| Platform Administration | 07-modules/platform-administration/ | Manage platform-side users, tenant lifecycle decisions, feature entitlements and support controls. |
| Tenant Management | 07-modules/tenant-management/ | Maintain tenant profile, outlets, addresses, operating mode and document numbering. |
| Identity, RBAC and Feature Access | 07-modules/identity-access/ | Control staff identity, roles, permissions, outlet assignments and feature-level access. |
| Settings, Feature Flags and Themes | 07-modules/settings-configuration/ | Configure runtime settings, feature flags, UI themes and scoped tenant behavior without code changes. |
| Data Import and AI-Assisted Onboarding | 07-modules/data-import-ai/ | Import products/customers from files and use AI extraction only through reviewable, auditable staging. |
| Product and Catalog Management | 07-modules/catalog/ | Own product master data, variants, attributes, brands, suppliers, images and channel visibility. |
| Pricing | 07-modules/pricing/ | Own tenant price lists, channel pricing, outlet overrides and deterministic price selection. |
| Tax and Calculation Policy | 07-modules/tax/ | Define tax classes, tax rates, inclusive/exclusive pricing behavior, rounding and refund tax reversal rules. |
| Inventory and Stock Control | 07-modules/inventory/ | Maintain outlet-wise stock, stock ledger, reservations, receiving, adjustments, transfers and stocktakes. |
| POS Devices, Terminals and Hardware | 07-modules/pos-devices-hardware/ | Register POS terminals and configure tills, receipt printers, scanners and peripheral assignment by outlet. |
| POS Sales and Checkout | 07-modules/sales-pos/ | Run fast barcode-first POS checkout with active session, cart, hold/recall, payment trigger, stock deduction and receipt trigger. |
| Payments, Refunds and Allocations | 07-modules/payments/ | Record cash/manual/gateway payments, payment transactions, split allocations and refunds. |
| Discounts, Coupons and Approvals | 07-modules/discounts-promotions/ | Control manual discounts, coupons, approval thresholds, stacking rules and redemption history. |
| Returns and Exchanges | 07-modules/returns-exchanges/ | Process POS and e-commerce returns/exchanges with policy validation, refund/collection, stock restoration and audit. |
| Customer Management | 07-modules/customers/ | Maintain tenant-scoped customer profiles, addresses, online auth accounts and duplicate detection. |
| E-Commerce Storefront, Cart, Checkout and Orders | 07-modules/ecommerce-orders/ | Support online product browsing, cart, checkout, order creation, customer tracking, wishlist and reviews. |
| Order, Payment and Fulfillment Workflow | 07-modules/order-workflow/ | Standardize status transitions across order, payment and fulfillment lifecycles. |
| Fulfillment, Pickup and Delivery | 07-modules/fulfillment-logistics/ | Manage delivery methods, zones, rates, delivery documents, pickup collection and delivery tracking. |
| Loyalty and Membership | 07-modules/loyalty/ | Define loyalty programs, tiers, customer memberships and immutable point ledger transactions. |
| OTP and Customer Authentication Security | 07-modules/otp-auth-security/ | Issue, verify and rate-limit OTP flows for login, signup, reset and verification use cases. |
| Receipts and Print Logs | 07-modules/receipts/ | Generate frozen receipt outputs, manage templates and audit print/reprint/email/download actions. |
| Offline POS Sync | 07-modules/offline-sync/ | Receive offline-created sales/payments/receipts/stock movements, validate them and manage conflicts safely. |
| Reporting and Dashboards | 07-modules/reporting/ | Expose daily sales, payment, inventory, discount, return, tax, cash and offline sync reporting. |
| Audit and Compliance | 07-modules/audit-compliance/ | Provide immutable auditability for sensitive business, configuration, payment, stock and offline sync actions. |

## Non-negotiable rules

- Backend is the final authority for tenant, outlet, permission, feature and workflow validation.
- Every tenant-owned business record must be tenant-scoped directly or through a tenant-scoped parent.
- Financial, stock, payment, refund, receipt, offline sync and sensitive configuration actions must be traceable.
- Offline POS queues are staging records; accepted transactions must be written to the source-of-truth tables.
- Feature access is not fixed by role name. Effective access is tenant-specific: feature entitlement, feature flag, role feature assignment and permissions must all be considered.

## Clean export notes

This ZIP intentionally excludes `.git` history and Obsidian Local REST API secret configuration. Do not add API keys, local private keys, `.env` values or local workspace state to this repository.
