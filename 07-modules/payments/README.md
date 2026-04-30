---
title: Payments Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - payments
---

# Payments Module

## Purpose

Records tenant payment methods, provider config references, payments, gateway events, allocations to sales/orders, and refunds.

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
- Customer
- E-Commerce Operator
- Outlet Manager
- Backend Service
- Payment Provider

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
| `payment_method_types` | `id` | code UNIQUE | Global payment method reference values. |
| `tenant_payment_methods` | `id` | tenant_id -> tenants.id; payment_method_type_id; payment_provider_config_id | Tenant-enabled payment methods. |
| `payment_provider_configs` | `id` | tenant_id -> tenants.id; provider_code; environment | Provider configuration with secret reference. |
| `payments` | `id` | tenant_id -> tenants.id; customer_id; outlet_id; till_session_id; tenant_payment_method_id | Unified payment/payout record. |
| `payment_transactions` | `id` | tenant_id -> tenants.id; payment_id -> payments.id | Gateway/provider event log. |
| `sale_payment_allocations` | `id` | tenant_id -> tenants.id; sale_id -> sales.id; payment_id -> payments.id | Allocates payments to POS sales. |
| `order_payment_allocations` | `id` | tenant_id -> tenants.id; order_id -> orders.id; payment_id -> payments.id | Allocates payments to e-commerce orders. |
| `refunds` | `id` | tenant_id -> tenants.id; original_payment_id -> payments.id; refund_payment_id nullable | Refund header. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/payments/features/payment-method-types/feature-spec|Payment Method Types]] | Platform-owned payment reference values such as cash, card, QR, wallet, bank transfer, and gift card. | `payment_method_types` |
| [[07-modules/payments/features/tenant-payment-methods/feature-spec|Tenant Payment Methods]] | Tenant-enabled payment methods and non-secret method configuration. | `tenant_payment_methods`, `payment_method_types`, `payment_provider_configs` |
| [[07-modules/payments/features/payment-provider-configs/feature-spec|Payment Provider Configs]] | Gateway/provider configuration using secret references instead of plain secrets. | `payment_provider_configs` |
| [[07-modules/payments/features/payments/feature-spec|Payments]] | Unified inbound/outbound payment record for sale, order, refund, or exchange difference. | `payments` |
| [[07-modules/payments/features/payment-transactions/feature-spec|Payment Transactions]] | Gateway/provider event log per payment for auth, capture, void, refund, webhook, or failure. | `payment_transactions`, `payments` |
| [[07-modules/payments/features/payment-allocations/feature-spec|Payment Allocations]] | Allocates captured payments to POS sales or e-commerce orders. | `sale_payment_allocations`, `order_payment_allocations`, `payments`, `sales`, `orders` |
| [[07-modules/payments/features/refunds/feature-spec|Refunds]] | Business refund header linked to original captured payment and optional outbound refund payment. | `refunds`, `payments` |

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
