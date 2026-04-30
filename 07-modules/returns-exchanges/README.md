---
title: Returns and Exchanges Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - returns-exchanges
---

# Returns and Exchanges Module

## Purpose

Owns return and exchange documents, lines, reason codes, refund allocations, exchange lines, and exchange difference allocations.

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
- Customer
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
| `return_reason_codes` | `id` | tenant_id -> tenants.id; code | Tenant-owned return reasons. |
| `returns` | `id` | tenant_id -> tenants.id; source_sale_id/source_order_id; original_outlet_id; return_outlet_id | Return header. |
| `return_lines` | `id` | tenant_id -> tenants.id; return_id; source_sale_line_id/source_order_item_id; variant_id | Returned line items. |
| `return_refund_allocations` | `id` | tenant_id -> tenants.id; return_id; refund_id | Allocates refunds to returns. |
| `exchanges` | `id` | tenant_id -> tenants.id; source_return_id | Exchange header. |
| `exchange_lines` | `id` | tenant_id -> tenants.id; exchange_id; variant_id | New items issued in exchange. |
| `exchange_payment_allocations` | `id` | tenant_id -> tenants.id; exchange_id; payment_id | Additional collection for exchange difference. |
| `exchange_refund_allocations` | `id` | tenant_id -> tenants.id; exchange_id; refund_id | Refund for exchange difference. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/returns-exchanges/features/return-reason-codes/feature-spec|Return Reason Codes]] | Tenant-owned return reason reference values. | `return_reason_codes` |
| [[07-modules/returns-exchanges/features/returns/feature-spec|Returns]] | Return header for POS sale or e-commerce order with original and processing outlet context. | `returns`, `sales`, `orders`, `customers` |
| [[07-modules/returns-exchanges/features/return-lines/feature-spec|Return Lines]] | Returned line items linked to source sale line or order item. | `return_lines`, `returns`, `sale_lines`, `order_items`, `product_variants` |
| [[07-modules/returns-exchanges/features/return-refund-allocations/feature-spec|Return Refund Allocations]] | Allocates refund records to return documents. | `return_refund_allocations`, `returns`, `refunds` |
| [[07-modules/returns-exchanges/features/exchanges/feature-spec|Exchanges]] | Exchange header created from a return with old/new values and difference direction. | `exchanges`, `returns` |
| [[07-modules/returns-exchanges/features/exchange-lines/feature-spec|Exchange Lines]] | New items issued in exchange with pricing snapshot. | `exchange_lines`, `exchanges`, `product_variants` |
| [[07-modules/returns-exchanges/features/exchange-payment-refund-allocations/feature-spec|Exchange Payment and Refund Allocations]] | Allocates payment/refund records for exchange differences. | `exchange_payment_allocations`, `exchange_refund_allocations`, `payments`, `refunds` |

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
