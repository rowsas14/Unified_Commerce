---
title: E-Commerce Orders Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - ecommerce-orders
---

# E-Commerce Orders Module

## Purpose

Owns online carts, cart items, orders, order items, address snapshots, wishlists, and product reviews.

This module is part of the production-ready Unified Commerce E-POS + E-Commerce SaaS system. It must follow tenant isolation, role/permission validation, feature access validation, API rules, backend Clean Architecture rules, frontend implementation rules, audit requirements, and testing expectations defined in the 2nd Brain.

## Required reading

| Area | Document |
|---|---|
| Product scope | [[01-product/project-scope|Project Scope]] |
| Module catalog | [[01-product/production-module-catalog|Production Module Catalog]] |
| System architecture | [[02-architecture/system-overview|System Overview]] |
| Tenancy model | [[02-architecture/tenancy-model|Tenancy Model]] |
| RBAC / feature access | [[02-architecture/role-permission-capability-model|Role Permission Capability Model]] |
| Database overview | [[03-data/database-overview|Database Overview]] |
| API rules | [[04-api/api-overview|API Overview]] |
| Backend rules | [[05-backend/backend-overview|Backend Overview]] |
| Frontend rules | [[06-frontend/frontend-overview|Frontend Overview]] |
| Security rules | [[09-security-and-compliance/README|Security and Compliance]] |
| AI IDE rules | [[14-ai-ide-rules/README|AI IDE Rules]] |

## Actors

- Customer
- Guest Customer
- E-Commerce Operator
- Backend Service
- Frontend Storefront

## Module scope

### In scope

- Maintain only the responsibilities explicitly described in this module and its feature specs.
- Use only database tables listed in this module or referenced by linked modules.
- Enforce tenant isolation for every tenant-owned operation.
- Apply backend authorization and feature access checks before changing data.
- Update audit logs or workflow history where the uploaded database design provides the required table.
- Keep frontend screens operational and workflow-focused, especially for POS-facing behavior.

### Out of scope

- Do not own responsibilities of unrelated modules.
- Do not create new tables that are not in the uploaded database design.
- Do not add CQRS, mediator, event sourcing, or undocumented architecture patterns.
- Do not bypass the API/backend/security rules documented in the 2nd Brain.


## Owned or primary tables

| Table | PK | Important FK / attribute references | Purpose |
| --- | --- | --- | --- |
| `carts` | `id` | tenant_id -> tenants.id; customer_id nullable; guest_token | Shopping cart header. |
| `cart_items` | `id` | tenant_id -> tenants.id; cart_id -> carts.id; variant_id -> product_variants.id | Cart line items. |
| `orders` | `id` | tenant_id -> tenants.id; customer_id; source_cart_id; fulfillment_outlet_id | E-commerce order header. |
| `order_items` | `id` | tenant_id -> tenants.id; order_id -> orders.id; variant_id -> product_variants.id | Order line items. |
| `order_addresses` | `id` | tenant_id -> tenants.id; order_id -> orders.id | Immutable order address snapshots. |
| `wishlists` | `id` | tenant_id -> tenants.id; customer_id -> customers.id | Customer wishlist header. |
| `wishlist_items` | `id` | tenant_id -> tenants.id; wishlist_id -> wishlists.id; variant_id -> product_variants.id | Wishlist items. |
| `product_reviews` | `id` | tenant_id -> tenants.id; customer_id; product_id; order_id nullable | Moderated product reviews. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/ecommerce-orders/features/carts/feature-spec|Carts]] | Shopping cart header for logged-in or guest customer context. | `carts`, `customers` |
| [[07-modules/ecommerce-orders/features/cart-items/feature-spec|Cart Items]] | Variant line items inside cart with pricing snapshot. | `cart_items`, `carts`, `product_variants` |
| [[07-modules/ecommerce-orders/features/orders/feature-spec|Orders]] | E-commerce order header with order/payment/fulfillment status fields. | `orders`, `customers`, `carts`, `outlets` |
| [[07-modules/ecommerce-orders/features/order-items/feature-spec|Order Items]] | Frozen order line items with reserved, fulfilled, and returned quantities. | `order_items`, `orders`, `product_variants` |
| [[07-modules/ecommerce-orders/features/order-addresses/feature-spec|Order Addresses]] | Immutable billing/shipping address snapshots under an order. | `order_addresses`, `orders` |
| [[07-modules/ecommerce-orders/features/wishlists/feature-spec|Wishlists]] | Customer wishlist header and items for e-commerce saved variants. | `wishlists`, `wishlist_items`, `customers`, `product_variants` |
| [[07-modules/ecommerce-orders/features/product-reviews/feature-spec|Product Reviews]] | Moderated customer product reviews linked to product and optional order. | `product_reviews`, `customers`, `products`, `orders` |

## Dependency map

```text
Tenant context
  -> permissions and feature access
  -> module validation
  -> API request handling
  -> service layer workflow
  -> repository/data access
  -> audit/reporting where applicable
```

## Business rule summary

- Tenant-owned rows must never be shared across tenants.
- Status changes must follow documented status rules and allowed transitions where available.
- Monetary, stock, payment, refund, discount, receipt, offline sync, and audit-related changes must be validated by the backend.
- Frontend may guide the user and protect the workflow, but backend remains the final authority.
- Read-model/reporting tables are not source-of-truth transaction tables.
- Offline staging tables are not source-of-truth transaction tables.

## API reference policy

This module must follow [[04-api/endpoint-design|Endpoint Design]] and [[04-api/api-overview|API Overview]]. Do not invent endpoint names in implementation. Final API routes must be documented in the API folder before code is written.

## Backend implementation policy

- Use Clean Architecture with Service Pattern and Repository Pattern only.
- Controllers must remain thin.
- Application services orchestrate workflows.
- Domain entities/domain services hold pure business rules where applicable.
- Repositories handle data access.
- Unit of Work/transaction boundaries must protect multi-table writes.

## Frontend implementation policy

- Use React + TypeScript + Tailwind CSS.
- Use TanStack Query for server state.
- Use Zustand only for client workflow state.
- Never trust UI hiding as authorization.
- Keep POS-facing screens touchscreen-first, fast, and operational.

## QA checklist

- [ ] Tenant isolation tested.
- [ ] Permission and feature access tested.
- [ ] Required entities and FK relationships verified.
- [ ] API request/response validation tested.
- [ ] Backend service workflow tested.
- [ ] Frontend loading/error/empty states tested.
- [ ] Audit or history behavior tested where applicable.
- [ ] Offline behavior tested if the feature can operate offline.
- [ ] Reporting impact reviewed where applicable.

## Implementation notes for AI IDE tools

- Read this module README before editing any feature in this module.
- Read the target feature `feature-spec.md` and `feature-history.md` before coding.
- Do not implement schema gaps as new tables unless the database design is officially updated.
- Update feature history after implementation, bug fixes, or rule changes.
- Cross-reference related modules instead of duplicating rules.
