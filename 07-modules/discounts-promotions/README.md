---
title: Discounts and Promotions Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - discounts-promotions
---

# Discounts and Promotions Module

## Purpose

Controls discount policies, approval requests, coupons, applied discounts, and coupon redemption tracking across POS sales and e-commerce orders.

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

- Cashier
- Outlet Manager
- Tenant Admin
- E-Commerce Operator
- Backend Service

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
| `discount_types` | `id` | code UNIQUE | Discount calculation type reference. |
| `discount_scopes` | `id` | code UNIQUE | Discount scope reference. |
| `discount_policies` | `id` | tenant_id -> tenants.id; outlet_id nullable; channel | Tenant discount approval and stacking rules. |
| `discount_requests` | `id` | tenant_id -> tenants.id; sale_id/order_id; requested_by; approved_by | Approval workflow request. |
| `coupons` | `id` | tenant_id -> tenants.id; code; discount_type_id | Tenant coupon master. |
| `discount_applications` | `id` | tenant_id -> tenants.id; sale/order references; coupon_id/discount_request_id | Actual discount applied. |
| `coupon_redemptions` | `id` | tenant_id -> tenants.id; coupon_id; customer_id; sale_id/order_id | Actual coupon use. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/discounts-promotions/features/discount-policies/feature-spec|Discount Policies]] | Tenant/outlet/channel approval and stacking rules. | `discount_policies` |
| [[07-modules/discounts-promotions/features/discount-requests/feature-spec|Discount Requests]] | Approval workflow request raised against sale/order or line item. | `discount_requests`, `discount_types`, `discount_scopes` |
| [[07-modules/discounts-promotions/features/coupons/feature-spec|Coupons]] | Tenant coupon master with channel, validity, usage limits, and rule payload. | `coupons`, `discount_types` |
| [[07-modules/discounts-promotions/features/discount-applications/feature-spec|Discount Applications]] | Actual discount applied to sale/order or line after approval/coupon/manual action. | `discount_applications` |
| [[07-modules/discounts-promotions/features/coupon-redemptions/feature-spec|Coupon Redemptions]] | Actual coupon use against sale/order with customer tracking where available. | `coupon_redemptions`, `coupons` |

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
