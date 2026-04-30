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
# API Overview

The Unified Commerce API supports the production E-POS + E-Commerce SaaS platform.
It connects POS terminals, admin screens, storefront screens, e-commerce operations, platform admin tools, and offline sync flows to the backend application layer.

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


## API base path

```http
/api/v1
```

All route examples and route families in this folder use this version prefix.
Do not create production routes outside `/api/v1`.

## Architecture boundary

```text
Frontend client
  ↓
/api/v1 controller or endpoint adapter
  ↓
Authentication + tenant context + feature/permission validation
  ↓
Application service
  ↓
Domain model/domain service where business rules require it
  ↓
Repository + Unit of Work
  ↓
PostgreSQL tables and read models
```

## Layer responsibility

| Layer | API-facing responsibility |
|---|---|
| API layer | Routes, request models, response models, filters, middleware, auth context, controller orchestration. |
| Application layer | Use-case services, DTOs, validators, workflow orchestration, transaction boundaries. |
| Domain layer | Pure business rules for aggregates and domain services. |
| Infrastructure layer | Repository implementation, EF Core, external providers, printer/payment integration adapters. |
| Frontend | Calls documented APIs through API client and TanStack Query/offline queue where applicable. |

## Production API areas

| Area | API behavior |
|---|---|
| Tenant foundation | Tenant, outlet, document sequence, runtime configuration. |
| Identity/access | Platform users, tenant staff, roles, permissions, outlet assignments. |
| Feature access | Platform feature catalog, tenant entitlements, role feature assignments, feature flags. |
| Catalog/pricing/tax | Products, variants, categories, attributes, return policy, tax classes, price lists. |
| Inventory | Balances, stock movement ledger, reservations, adjustments, transfers, stocktakes. |
| POS | Devices, tills, sessions, cash movements, sales, sale lines, receipts. |
| E-Commerce | Customers, auth, carts, orders, order items, addresses, wishlist, reviews. |
| Fulfillment | Delivery methods, zones, deliveries, delivery items, tracking. |
| Payments/refunds | Tenant payment methods, payments, provider transactions, allocations, refunds. |
| Discounts/coupons | Policies, discount requests, coupon redemptions, discount applications. |
| Returns/exchanges | Return/exchange documents, lines, refund/payment allocations. |
| Offline sync | Sync batches, items, typed sale/payment queues, conflicts, sync audit logs. |
| Reporting/audit | Daily summaries, audit logs, operational reports. |

## API authority rules

| Data or decision | Final authority |
|---|---|
| Tenant access | Backend middleware/service validation. |
| Feature availability | Backend feature access service. |
| Role/permission access | Backend authorization layer. |
| Product availability | Backend catalog and channel rules. |
| Price/tax/discount totals | Backend pricing/tax/discount services. |
| Stock availability | Backend inventory and reservation services. |
| Payment/refund status | Backend payment services and provider transaction records. |
| Receipt payload | Backend receipt generation service. |
| Offline sync acceptance | Backend sync processor. |
| Status transitions | Backend workflow validation. |

## Client-specific API constraints

| Client | Constraint |
|---|---|
| POS terminal | Must include valid device/session context for POS workflows. |
| Offline POS | Must use local IDs and sync batch/item model for queued transactions. |
| Storefront | Must not receive tenant-internal admin fields. |
| Customer account | Must not access another customer's tenant-scoped data. |
| Admin portal | Must enforce role/permission/feature access per module. |
| Platform admin | Must separate platform operations from tenant staff operations. |

## `/api/v1` compatibility rule

POS terminals may remain offline and sync later.
Therefore `/api/v1` sync, payment, receipt, sale, and status behavior must remain stable once released.
Breaking changes require deliberate versioning and migration handling.

## API source-of-truth links

| Concern | Source link |
|---|---|
| Product scope | [[01-product/project-scope]] |
| Module list | [[01-product/production-module-catalog]] |
| System architecture | [[02-architecture/system-overview]] |
| Data model | [[03-data/database-overview]] |
| Security | [[09-security-and-compliance/README]] |
| Backend implementation | [[05-backend/backend-overview]] |
| Frontend API usage | [[06-frontend/api-client-and-query-rules]] |
| API template | [[12-templates/api-spec-template]] |
