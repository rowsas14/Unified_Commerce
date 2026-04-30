---
title: {title}
owner: API Architecture / Backend Architecture
status: production-ready
last_reviewed: {DATE}
tags: [{tags}]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
---
# Module Endpoint Map

This file maps production modules to API route families under `/api/v1`.
The map is intentionally module-based, because the uploaded backend architecture uses feature/module grouping and Clean Architecture service boundaries.

## Required reading

- [[00-start-here/README|Start Here]]
- [[01-product/project-scope|Production Project Scope]]
- [[01-product/production-module-catalog|Production Module Catalog]]
- [[02-architecture/system-overview|System Overview]]
- [[02-architecture/tenancy-architecture|Tenancy Architecture]]
- [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
- [[03-data/database-overview|Database Overview]]
- [[03-data/tenant-consistency-rules|Tenant Consistency Rules]]
- [[09-security-and-compliance/authorization-model|Authorization Model]]
- [[09-security-and-compliance/audit-requirements|Audit Requirements]]


> **API contract rule**  
> This folder defines production API design rules and route-family conventions.  
> A final endpoint contract must still be documented in a feature API spec using [[12-templates/api-spec-template]].  
> Do not implement undocumented endpoints only because a table exists.


## Route-family naming rules

| Rule | Decision |
|---|---|
| Version prefix | Always `/api/v1`. |
| Resource naming | Use plural nouns for resource collections. |
| Module ownership | Every route family must map to a production module. |
| Workflow actions | Use action subroutes only when state transition rules are required. |
| Platform routes | Platform-admin-only APIs use a platform route family. |
| Tenant routes | Tenant-user APIs resolve tenant through auth/context. |
| POS routes | POS routes validate outlet, device, till, and session. |

## Route-family map

| Production module | Route family | Main database concepts |
|---|---|---|
| Platform Administration | `/api/v1/platform/...` | `platform_users`, `platform_features`, tenant entitlements |
| Tenant Management | `/api/v1/tenants`, `/api/v1/outlets` | `tenants`, `outlets`, `outlet_addresses`, `document_sequences` |
| Identity Access | `/api/v1/auth`, `/api/v1/users`, `/api/v1/roles`, `/api/v1/permissions` | users, roles, permissions, assignments |
| Feature Access | `/api/v1/features`, `/api/v1/feature-access` | features, entitlements, role-feature assignments, flags |
| Settings Configuration | `/api/v1/settings`, `/api/v1/themes` | tenant settings, UI themes, feature flags |
| Catalog | `/api/v1/catalog/...` | brands, suppliers, categories, products, variants, attributes, images |
| Tax | `/api/v1/tax/...` | tax classes, rates, class-rate mapping |
| Pricing | `/api/v1/pricing/...` | price lists and price list items |
| Inventory | `/api/v1/inventory/...` | balances, allocations, movements, reservations, receiving, adjustments, transfers, stocktakes |
| POS Devices | `/api/v1/pos/devices`, `/api/v1/pos/tills` | POS devices and tills |
| Till Sessions | `/api/v1/pos/sessions` | till sessions, cash movements, cash counts |
| POS Sales | `/api/v1/pos/sales` | sales and sale lines |
| Payments | `/api/v1/payments` | payment methods, provider configs, payments, transactions, allocations |
| Refunds | `/api/v1/refunds` | refunds and refund allocations |
| Discounts/Coupons | `/api/v1/discounts`, `/api/v1/coupons` | discount policies, requests, applications, coupon redemptions |
| Returns | `/api/v1/returns` | returns, return lines, refund allocations |
| Exchanges | `/api/v1/exchanges` | exchanges, exchange lines, payment/refund allocations |
| Receipts | `/api/v1/receipts` | templates, receipts, print logs |
| Customers | `/api/v1/customers` | customers, addresses, auth accounts |
| OTP/Auth Security | `/api/v1/otp`, `/api/v1/customer-auth` | OTP channels, purposes, verification rows, identities |
| Wishlist | `/api/v1/wishlists` | wishlist headers and items |
| Product Reviews | `/api/v1/reviews` | product review moderation |
| Loyalty/Membership | `/api/v1/loyalty/...` | loyalty programs, tiers, memberships, transactions |
| Carts | `/api/v1/carts` | carts and cart items |
| Orders | `/api/v1/orders` | orders, order items, order addresses |
| Order Workflow | `/api/v1/order-workflow` | transitions and status history |
| Fulfillment | `/api/v1/fulfillment/...` | delivery methods, zones, rates, deliveries, items, tracking |
| Offline Sync | `/api/v1/offline-sync/...` | batches, items, typed queues, conflicts, audit logs |
| Reporting | `/api/v1/reports/...` | daily sales, payment, inventory, discount/return summaries |
| Audit | `/api/v1/audit-logs` | business audit logs and sync audit diagnostics |

## Route-family dependency map

```text
POS sale API
  → tenant + outlet + user + role + feature access
  → registered POS device + till session
  → catalog + tax + pricing
  → inventory stock movement
  → payment allocation
  → receipt generation
  → audit/reporting side effects

E-Commerce order API
  → tenant + customer/cart
  → catalog + pricing + tax
  → stock reservation
  → payment recording/provider transaction
  → order workflow
  → fulfillment assignment
  → reporting/audit side effects
```

## Platform versus tenant API separation

| Route type | Caller | Notes |
|---|---|---|
| `/api/v1/platform/...` | Platform user | Tenant lifecycle, feature catalog, tenant entitlements. |
| `/api/v1/tenants/...` | Platform admin or tenant admin where allowed | Tenant-scoped configuration and outlet setup. |
| `/api/v1/pos/...` | Tenant staff on registered POS device | Requires outlet/device/till/session context. |
| `/api/v1/catalog/...` | Tenant users and storefront readers | Write access requires tenant staff permissions. |
| `/api/v1/orders/...` | Customer or operations staff | Response fields must be caller-specific. |
| `/api/v1/reports/...` | Manager/admin/reporting users | Must enforce report permissions. |

## Schema gap rule

If a scope area exists but the database design does not define the required source table, do not create final CRUD APIs yet.
Document the gap in [[03-data/required-schema-extensions]] and wait for schema approval.

## Completion checklist

- [ ] Route family is listed here.
- [ ] Concrete endpoints are documented in feature API specs.
- [ ] Module ownership is clear.
- [ ] Related tables are documented.
- [ ] Auth/tenant/feature/permission rules are clear.
- [ ] Unsafe workflows include idempotency/concurrency rules.
