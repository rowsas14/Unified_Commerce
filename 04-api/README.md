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
# API Documentation Index

This folder defines the API documentation rules for the production Unified Commerce E-POS + E-Commerce SaaS system.
The API layer is the controlled boundary between frontend clients and the backend application services.
It must protect tenant isolation, enforce feature access, validate workflow rules, and preserve transaction integrity.

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


## API baseline

| Area | Decision |
|---|---|
| API version prefix | `/api/v1` |
| Architecture style | .NET Web API with Clean Architecture, Service Pattern, Repository Pattern |
| API grouping | Feature/module-based route families |
| Backend authority | Backend validates tenant, feature, permission, price, tax, stock, payment, refund, and workflow state |
| Frontend role | Frontend improves UX and offline continuity but cannot be final authority |
| Tenant model | Tenant context is mandatory for tenant-owned business records |
| POS model | POS APIs must understand outlet, device, till, and session context |
| Offline model | Offline POS sync uses batch/item staging and server-side validation |
| Financial model | Payments, refunds, discounts, and cash movements require idempotency and audit where relevant |
| Reporting model | Reports read from transaction data and reporting read models, not manual totals |

## Files in this folder

| File | Purpose |
|---|---|
| [[04-api/api-overview|api-overview]] | Explains the overall API boundary, client types, module ownership, and `/api/v1` usage. |
| [[04-api/module-endpoint-map|module-endpoint-map]] | Maps production modules to route families and related database concepts. |
| [[04-api/endpoint-design|endpoint-design]] | Defines naming, verbs, resource routes, workflow routes, and status-changing route rules. |
| [[04-api/request-response-standard|request-response-standard]] | Defines request body, response envelope, metadata, and validation response conventions. |
| [[04-api/tenant-context-api-rules|tenant-context-api-rules]] | Defines tenant, outlet, device, till, channel, and platform-admin context rules. |
| [[04-api/auth-and-authorization|auth-and-authorization]] | Defines authentication and authorization behavior for API calls. |
| [[04-api/feature-access-api-rules|feature-access-api-rules]] | Defines platform entitlement, tenant runtime flag, role feature assignment, and permission checks. |
| [[04-api/idempotency-rules|idempotency-rules]] | Defines duplicate protection for payments, orders, sales, refunds, and offline sync. |
| [[04-api/concurrency-rules|concurrency-rules]] | Defines race-condition handling for stock, coupons, document sequences, payments, and sync. |
| [[04-api/error-contract|error-contract]] | Defines error response categories, structure, and module-specific error handling. |
| [[04-api/pagination-filtering-sorting|pagination-filtering-sorting]] | Defines list endpoint conventions for filters, sorting, search, and paging. |
| [[04-api/offline-sync-api-rules|offline-sync-api-rules]] | Defines offline sync batch/item/conflict route behavior. |
| [[04-api/payment-refund-api-rules|payment-refund-api-rules]] | Defines payment, allocation, provider transaction, and refund API behavior. |
| [[04-api/order-workflow-api-rules|order-workflow-api-rules]] | Defines order/payment/fulfillment transition API behavior. |
| [[04-api/device-session-api-rules|device-session-api-rules]] | Defines POS device, till, session, and cash context behavior. |
| [[04-api/api-change-checklist|api-change-checklist]] | Checklist to follow before changing or adding API behavior. |

## API clients

| Client | Main API needs |
|---|---|
| POS terminal | Auth, device/session context, product lookup, cart/sale, payment, receipt, cash close, offline sync. |
| Admin portal | Tenant, outlets, staff, roles, feature access, catalog, inventory, tax, pricing, reporting, configuration. |
| E-Commerce storefront | Product listing, cart, checkout, customer auth, orders, wishlist, reviews. |
| E-Commerce operations | Order processing, fulfillment, delivery, refund handling, customer support. |
| Platform admin | Tenant lifecycle, platform features, tenant entitlements, support-level visibility. |
| AI IDE tools | Must read this folder before generating or editing API/backend/frontend code. |

## Source alignment

| Source area | API documentation responsibility |
|---|---|
| Scope document | Defines the modules and workflows that APIs must support. |
| Database design | Defines table ownership, FK relationships, status values, and data constraints. |
| Backend architecture | Defines controller/application service/domain/infrastructure separation. |
| Frontend architecture | Defines consuming route guards, API client, POS shells, offline queue, and peripheral clients. |
| Security docs | Define authentication, authorization, tenant isolation, and audit requirements. |

## Global API rules

1. Every production route must use `/api/v1`.
2. Every endpoint must belong to a documented production module.
3. Backend must enforce tenant isolation even if frontend sends hidden or disabled UI.
4. Backend must reject records that cross tenant boundaries.
5. POS APIs must validate outlet, device, till, and session context.
6. Feature-controlled APIs must check entitlement, runtime flag, role-feature assignment, and permission.
7. Payment/refund/order/sale/offline sync APIs must be idempotent where duplicate submission is possible.
8. Status changes must use documented transitions and audit rules.
9. Reporting APIs must not mutate source transaction data.
10. API docs must not invent tables, endpoints, or workflows outside the uploaded source documents.

## Documentation checklist

- [ ] Route family is listed in [[04-api/module-endpoint-map]].
- [ ] Concrete endpoint uses `/api/v1`.
- [ ] Related module README exists under `07-modules`.
- [ ] Related feature spec exists.
- [ ] Related database tables are documented under [[03-data/README|03-data]].
- [ ] Auth requirement is documented.
- [ ] Tenant/outlet/device/channel context is documented.
- [ ] Feature access requirement is documented where applicable.
- [ ] Request and response shapes follow [[04-api/request-response-standard]].
- [ ] Errors follow [[04-api/error-contract]].
- [ ] Idempotency and concurrency behavior are documented where relevant.
- [ ] Tests are planned under [[10-testing-quality/test-strategy]].

## Do not do this

- Do not add unversioned APIs.
- Do not trust frontend-calculated totals.
- Do not trust body-only `tenant_id` values.
- Do not create generic CRUD APIs for sensitive workflows.
- Do not update inventory balances without stock movement traceability.
- Do not implement offline sync as direct blind inserts into final tables.
- Do not expose provider raw payloads, audit payloads, or sensitive logs to normal users.
- Do not create endpoints for schema gaps without approved database design updates.
