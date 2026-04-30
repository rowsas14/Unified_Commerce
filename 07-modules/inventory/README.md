---
title: Inventory Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - inventory
---

# Inventory Module

## Purpose

Tracks stock by tenant, outlet, and product variant using balances, reservations, immutable stock movements, receipts, adjustments, transfers, and stocktakes.

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

- Inventory Staff
- Outlet Manager
- Cashier
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
| `inventory_balances` | `id` | tenant_id -> tenants.id; outlet_id -> outlets.id; variant_id -> product_variants.id | Current stock projection. |
| `inventory_channel_allocations` | `id` | tenant_id -> tenants.id; outlet_id; variant_id; channel | Optional channel allocation. |
| `stock_movement_types` | `id` | code UNIQUE; direction | Reference movement types. |
| `stock_movements` | `id` | tenant_id -> tenants.id; outlet_id; variant_id; movement_type_id; reference FKs | Immutable stock ledger. |
| `stock_reservations` | `id` | tenant_id -> tenants.id; outlet_id; variant_id; order_id; order_item_id | E-commerce reservation rows. |
| `purchase_receipts` | `id` | tenant_id -> tenants.id; supplier_id; outlet_id | Supplier receiving document. |
| `purchase_receipt_lines` | `id` | tenant_id -> tenants.id; purchase_receipt_id; variant_id | Received stock lines. |
| `stock_adjustments` | `id` | tenant_id -> tenants.id; outlet_id; created_by; approved_by | Manual inventory adjustment header. |
| `stock_adjustment_lines` | `id` | tenant_id -> tenants.id; stock_adjustment_id; variant_id | Manual adjustment lines. |
| `stock_transfers` | `id` | tenant_id -> tenants.id; from_outlet_id; to_outlet_id | Stock transfer header. |
| `stock_transfer_lines` | `id` | tenant_id -> tenants.id; transfer_id; variant_id | Transfer lines. |
| `stocktakes` | `id` | tenant_id -> tenants.id; outlet_id; created_by | Stock count session header. |
| `stocktake_lines` | `id` | tenant_id -> tenants.id; stocktake_id; variant_id | Stock count results. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/inventory/features/inventory-balances/feature-spec|Inventory Balances]] | Current stock projection by outlet and variant. | `inventory_balances`, `outlets`, `product_variants` |
| [[07-modules/inventory/features/channel-allocations/feature-spec|Inventory Channel Allocations]] | Optional allocation of outlet stock between POS and e-commerce channels. | `inventory_channel_allocations`, `inventory_balances` |
| [[07-modules/inventory/features/stock-ledger/feature-spec|Stock Ledger]] | Immutable stock movement history with movement type, reference document, source channel, and offline dedupe. | `stock_movement_types`, `stock_movements` |
| [[07-modules/inventory/features/stock-reservations/feature-spec|Stock Reservations]] | E-commerce stock reservations before fulfillment and release/commit/expiry handling. | `stock_reservations`, `orders`, `order_items` |
| [[07-modules/inventory/features/purchase-receipts/feature-spec|Purchase Receipts]] | Supplier stock receiving headers and line items that create purchase receipt stock movements when posted. | `purchase_receipts`, `purchase_receipt_lines`, `suppliers` |
| [[07-modules/inventory/features/stock-adjustments/feature-spec|Stock Adjustments]] | Manual inventory adjustment documents with reason, approval, and posted adjustment movements. | `stock_adjustments`, `stock_adjustment_lines` |
| [[07-modules/inventory/features/stock-transfers/feature-spec|Stock Transfers]] | Stock movement between outlets through transfer header and line quantities. | `stock_transfers`, `stock_transfer_lines` |
| [[07-modules/inventory/features/stocktakes/feature-spec|Stocktakes]] | Stock count sessions and counted lines that create stocktake gain/loss movements. | `stocktakes`, `stocktake_lines` |

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
