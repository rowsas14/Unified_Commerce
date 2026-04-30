---
title: 07 Modules
type: module-index
status: production-ready
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - modules
  - production-scope
---

# 07 Modules

## Purpose

This folder contains module-level documentation and feature-level documentation for the production-ready Unified Commerce E-POS + E-Commerce SaaS system.

Each module folder defines the module purpose, boundaries, owned entities, feature list, validation expectations, API/backend/frontend touchpoints, security expectations, user-flow references, and QA checklist. Each feature folder contains a `feature-spec.md` and `feature-history.md`.

## Required reading

| Area | Document |
|---|---|
| Start here | [[00-start-here/README|Start Here]] |
| Production scope | [[01-product/project-scope|Project Scope]] |
| Production module catalog | [[01-product/production-module-catalog|Production Module Catalog]] |
| System overview | [[02-architecture/system-overview|System Overview]] |
| Database overview | [[03-data/database-overview|Database Overview]] |
| API overview | [[04-api/api-overview|API Overview]] |
| Backend rules | [[05-backend/backend-overview|Backend Overview]] |
| Frontend rules | [[06-frontend/frontend-overview|Frontend Overview]] |
| Security rules | [[09-security-and-compliance/README|Security and Compliance]] |
| Templates | [[12-templates/feature-spec-template|Feature Spec Template]] |
| AI IDE rules | [[14-ai-ide-rules/README|AI IDE Rules]] |

## Module map

| Module | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/tenant-management/README|Tenant Management]] | Manages the tenant business account, outlets, outlet addresses, and tenant/outlet document numbering foundation for the production Unified Commerce platform. | `tenants`, `outlets`, `outlet_addresses`, `document_sequences` |
| [[07-modules/platform-administration/README|Platform Administration]] | Controls platform users, tenant lifecycle support, and platform-enabled feature availability for tenant accounts. | `platform_users`, `tenant_feature_entitlements`, `platform_features` |
| [[07-modules/identity-access/README|Identity and Access]] | Manages tenant staff identity, roles, permissions, tenant/outlet role assignment, and role-level feature access. | `users`, `roles`, `permissions`, `role_permissions`, `tenant_user_roles`, `outlet_user_roles`, `platform_features`, `tenant_feature_entitlements`, `role_feature_assignments` |
| [[07-modules/settings-configuration/README|Settings and Configuration]] | Stores tenant runtime settings, feature flags, and UI theme tokens by tenant, outlet, channel, or user scope where supported. | `feature_flags`, `tenant_settings`, `ui_themes` |
| [[07-modules/tax/README|Tax]] | Defines tenant tax classes, tax rates, and effective tax-class-to-rate mappings used by catalog, POS, e-commerce orders, returns, refunds, and reports. | `tax_classes`, `tax_rates`, `tax_class_rates` |
| [[07-modules/pricing/README|Pricing]] | Owns named price lists and variant prices by tenant, channel, currency, priority, effective dates, and optional outlet override. | `price_lists`, `price_list_items` |
| [[07-modules/inventory/README|Inventory]] | Tracks stock by tenant, outlet, and product variant using balances, reservations, immutable stock movements, receipts, adjustments, transfers, and stocktakes. | `inventory_balances`, `inventory_channel_allocations`, `stock_movement_types`, `stock_movements`, `stock_reservations`, `purchase_receipts`, `purchase_receipt_lines`, `stock_adjustments`, `stock_adjustment_lines`, `stock_transfers`, `stock_transfer_lines`, `stocktakes`, `stocktake_lines` |
| [[07-modules/pos-devices-hardware/README|POS Devices and Hardware]] | Controls POS terminal identity, till assignment, outlet context, and documented peripheral behavior for scanners, printers, and cash drawer integration. | `tills`, `pos_devices` |
| [[07-modules/sales-pos/README|Sales POS]] | Owns POS cashier session control, scan/add/pay checkout, held sales, void/cancel behavior, cash movements, and sale line recording. | `till_sessions`, `cash_movement_types`, `cash_movements`, `cash_count_denominations`, `sales`, `sale_lines` |
| [[07-modules/payments/README|Payments]] | Records tenant payment methods, provider config references, payments, gateway events, allocations to sales/orders, and refunds. | `payment_method_types`, `tenant_payment_methods`, `payment_provider_configs`, `payments`, `payment_transactions`, `sale_payment_allocations`, `order_payment_allocations`, `refunds` |
| [[07-modules/discounts-promotions/README|Discounts and Promotions]] | Controls discount policies, approval requests, coupons, applied discounts, and coupon redemption tracking across POS sales and e-commerce orders. | `discount_types`, `discount_scopes`, `discount_policies`, `discount_requests`, `coupons`, `discount_applications`, `coupon_redemptions` |
| [[07-modules/customers/README|Customers]] | Manages tenant-scoped POS and e-commerce customer profiles, auth account wrapper, auth identities, and customer addresses. | `customers`, `customer_auth_accounts`, `customer_auth_identities`, `customer_addresses` |
| [[07-modules/ecommerce-orders/README|E-Commerce Orders]] | Owns online carts, cart items, orders, order items, address snapshots, wishlists, and product reviews. | `carts`, `cart_items`, `orders`, `order_items`, `order_addresses`, `wishlists`, `wishlist_items`, `product_reviews` |
| [[07-modules/order-workflow/README|Order Workflow]] | Standardizes allowed status transitions and status history for order, payment, and fulfillment states. | `order_status_transitions`, `order_status_history`, `orders` |
| [[07-modules/fulfillment-logistics/README|Fulfillment and Logistics]] | Supports delivery/pickup methods, delivery zones and fees, delivery/pickup documents, fulfilled order items, and tracking events. | `delivery_methods`, `delivery_zones`, `delivery_zone_rates`, `deliveries`, `delivery_items`, `delivery_tracking` |
| [[07-modules/returns-exchanges/README|Returns and Exchanges]] | Owns return and exchange documents, lines, reason codes, refund allocations, exchange lines, and exchange difference allocations. | `return_reason_codes`, `returns`, `return_lines`, `return_refund_allocations`, `exchanges`, `exchange_lines`, `exchange_payment_allocations`, `exchange_refund_allocations` |
| [[07-modules/receipts/README|Receipts]] | Stores receipt templates, generated receipt payloads, barcode values, print status, and print/reprint/email/download history. | `receipt_templates`, `receipts`, `receipt_print_logs` |
| [[07-modules/offline-sync/README|Offline Sync]] | Owns backend-visible sync batches, sync items, typed sale/payment staging queues, conflicts, and technical sync audit logs for offline-first POS. | `offline_sync_batches`, `offline_sync_items`, `offline_sale_sync_queue`, `offline_payment_sync_queue`, `offline_sync_conflicts`, `offline_sync_audit_logs` |
| [[07-modules/reporting/README|Reporting]] | Provides daily read models for sales, payments, inventory, discounts, returns, and exchanges. These are reporting projections, not financial source-of-truth tables. | `daily_sales_summaries`, `daily_payment_summaries`, `daily_inventory_summaries`, `daily_discount_return_summaries` |
| [[07-modules/loyalty/README|Loyalty]] | Owns tenant loyalty program configuration, tiers, customer memberships, and immutable loyalty points ledger. | `loyalty_programs`, `membership_tiers`, `customer_memberships`, `loyalty_transactions` |
| [[07-modules/otp-auth-security/README|OTP Auth Security]] | Documents OTP channel, purpose, and verification history for staff/admin or customer auth contexts. | `otp_channels`, `otp_purposes`, `otp_verifications` |
| [[07-modules/audit-compliance/README|Audit and Compliance]] | Owns immutable business audit trail expectations for platform and tenant actions, including sensitive actions and configuration changes. | `audit_logs` |
| [[07-modules/data-import-ai/README|Data Import and AI Assisted Onboarding]] | Documents production scope for CSV/Excel import and AI-assisted onboarding while explicitly preserving the current schema gap for import and AI job persistence tables. | Schema gap / no approved owned table |

## Module documentation rules

- Every module README must state purpose, scope, actors, primary tables, feature map, dependency notes, backend/frontend policies, QA checklist, and AI IDE notes.
- Every feature folder must contain `feature-spec.md` and `feature-history.md`.
- Feature specs must use approved source documents only.
- Schema gaps must be explicit. Do not hide them or implement around them without a database update.
- Avoid duplicate rules across features. Put shared module behavior in the module README and cross-reference it.
- Use Obsidian-style wiki links when referencing other documentation.

## Dependency order for implementation

```text
Tenant foundation
  -> platform feature entitlement
  -> identity/RBAC
  -> settings/configuration
  -> catalog/tax/pricing
  -> inventory
  -> POS devices/session
  -> POS sales/payments/receipts
  -> e-commerce carts/orders/fulfillment
  -> returns/exchanges/refunds
  -> offline sync/reporting/audit
```

## Source of truth rules

| Concern | Source of truth |
|---|---|
| Product scope | `01-product` |
| System architecture | `02-architecture` |
| Database tables and relationships | `03-data` |
| API design rules | `04-api` |
| Backend rules | `05-backend` |
| Frontend rules | `06-frontend` |
| Module-specific implementation rules | `07-modules` |
| User workflows | `08-user-flows` |
| Security/compliance | `09-security-and-compliance` |
| Testing | `10-testing-quality` |
| AI IDE behavior | `14-ai-ide-rules` |

## AI IDE usage

AI IDE tools must not start implementation from code alone. They must read:

1. This module index.
2. Target module `README.md`.
3. Target feature `feature-spec.md`.
4. Target feature `feature-history.md`.
5. Related API/backend/frontend/security/user-flow docs.

If a feature references a schema gap, AI IDE tools must stop and ask for schema confirmation instead of inventing tables.
