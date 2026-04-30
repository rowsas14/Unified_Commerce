---
title: Backend Documentation Index
folder: 05-backend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
stack: .NET Web API, Clean Architecture, PostgreSQL, EF Core
patterns: Service Pattern, Repository Pattern, Unit of Work
cqrs: not-used
---

# Backend Documentation Index

This folder defines backend implementation rules for the production-ready Unified Commerce E-POS + E-Commerce SaaS system.
It is written for backend developers, architects, QA engineers, product owners, DevOps engineers, and AI IDE tools.

The backend must follow the uploaded backend architecture source: a .NET Web API solution with Clean Architecture layers, feature-based modules, services, DTOs, validators, repositories, external integration services, and Unit of Work.
The backend must also remain aligned with the production scope and production database design.

## Non-negotiable implementation boundary

This backend documentation uses **Clean Architecture with Service Pattern and Repository Pattern only**.

| Do | Do not |
|---|---|
| Use services for use-case orchestration | Do not introduce CQRS command/query handlers |
| Use repositories for persistence access | Do not introduce Mediator/MediatR pipeline guidance |
| Use validators for request/application validation | Do not move business logic into controllers |
| Use Unit of Work for transactional workflows | Do not bypass tenant and feature access checks |
| Use domain services only for pure domain rules | Do not call EF Core from the Domain layer |


## Required reading order

| Order | File | Why it matters |
|---:|---|---|
| 1 | [[05-backend/backend-overview]] | Gives the backend purpose, source alignment, and production module map. |
| 2 | [[05-backend/clean-architecture-rules]] | Defines what belongs in API, Application, Domain, and Infrastructure. |
| 3 | [[05-backend/backend-folder-structure]] | Converts the uploaded backend structure into a clean production solution structure. |
| 4 | [[05-backend/authentication-authorization]] | Defines platform user, tenant staff, customer, role, and permission handling. |
| 5 | [[05-backend/feature-access-handling]] | Explains tenant entitlement, role feature assignment, and feature flags. |
| 6 | [[05-backend/validation-rules]] | Defines validation responsibilities across API, Application, Domain, and database. |
| 7 | [[05-backend/transaction-boundary-rules]] | Defines safe transaction boundaries for sales, payments, inventory, returns, sync. |
| 8 | [[05-backend/offline-sync-backend-rules]] | Defines backend handling for offline POS sync queues and conflicts. |
| 9 | [[05-backend/backend-implementation-checklist]] | Final implementation gate for any backend feature. |

## Folder file map

| File | Primary audience | Main topic |
|---|---|---|
| [[05-backend/backend-overview]] | All backend contributors | Backend role in the full system |
| [[05-backend/clean-architecture-rules]] | Architects, backend developers | Layer responsibilities |
| [[05-backend/backend-folder-structure]] | Backend developers, AI IDE tools | Solution and project structure |
| [[05-backend/domain-service-rules]] | Architects, senior backend developers | When domain services are allowed |
| [[05-backend/dto-handling]] | Backend/API developers | Request, response, application DTO rules |
| [[05-backend/mapping-rules]] | Backend developers | Entity/DTO mapping boundaries |
| [[05-backend/validation-rules]] | Backend developers, QA | Input and business validation |
| [[05-backend/exception-handling]] | Backend developers, API consumers | Controlled backend errors |
| [[05-backend/authentication-authorization]] | Backend/security developers | Login, identity, RBAC |
| [[05-backend/feature-access-handling]] | Backend/security developers | Entitlements and feature access |
| [[05-backend/offline-sync-backend-rules]] | Backend/POS developers | Offline sync processing |
| [[05-backend/payment-gateway-integration-rules]] | Backend/payment developers | Manual and gateway-backed payment records |
| [[05-backend/transaction-boundary-rules]] | Backend developers | Unit of Work and transaction safety |
| [[05-backend/repository-layer-rules]] | Backend developers | Repository responsibilities |
| [[05-backend/service-layer-rules]] | Backend developers | Service orchestration rules |
| [[05-backend/background-job-rules]] | Backend/DevOps | Deferred processing rules from current scope |
| [[05-backend/outbox-event-rules]] | Architects | Controlled note: outbox not defined in uploaded source |
| [[05-backend/caching-strategy]] | Backend/frontend/platform | Safe backend caching boundaries |
| [[05-backend/naming-conventions]] | All developers | Names for modules, files, classes, methods |
| [[05-backend/backend-implementation-checklist]] | Developers, AI IDE tools, reviewers | Final backend gate |

## Source alignment

| Source document | Backend impact |
|---|---|
| Production scope | Defines required backend modules and workflows: tenant, RBAC, catalog, tax, inventory, POS, payments, discounts, returns, e-commerce, fulfillment, offline sync, reporting, configuration, security. |
| Database design | Defines the source-of-truth tables, relationships, tenant boundaries, offline sync queues, audit tables, and reporting read models. |
| Backend architecture | Defines Clean Architecture with API, Application, Domain, Infrastructure, services, DTOs, validators, repositories, external services, and Unit of Work. |
| Frontend architecture | Defines API consumers and frontend needs: POS shell, offline queue, peripherals, Zustand state, TanStack Query, and shared client-side calculation helpers. |

## Backend solution expectation

```text
Client / POS / Admin / E-Commerce
        |
        v
POS.Api
  Controllers, Requests, Responses, Middlewares, Filters
        |
        v
POS.Application
  Services, DTOs, Validators, Interfaces, Strategies when needed
        |
        v
POS.Domain
  Entities, value objects, domain rules, domain services
        |
        v
POS.Infrastructure
  EF Core persistence, repositories, external provider adapters, Unit of Work
        |
        v
PostgreSQL + external providers
```


## Production module map

| Production area | Backend ownership | Key database families |
|---|---|---|
| Platform and tenant foundation | Tenant lifecycle, outlet context, document numbers | `platform_users`, `tenants`, `outlets`, `outlet_addresses`, `document_sequences` |
| Identity and access | Staff auth, tenant roles, outlet roles, permission checks | `users`, `roles`, `permissions`, `role_permissions`, `tenant_user_roles`, `outlet_user_roles` |
| Feature access | Tenant feature entitlement, role feature assignment, runtime flags | `platform_features`, `tenant_feature_entitlements`, `role_feature_assignments`, `feature_flags` |
| Catalog, tax, pricing | Product master, variants, attributes, tax classes, price lists | `products`, `product_variants`, `tax_classes`, `tax_rates`, `price_lists`, `price_list_items` |
| Inventory | Balance projection, immutable stock movement ledger, transfers, stocktakes | `inventory_balances`, `stock_movements`, `stock_reservations`, `stock_transfers`, `stocktakes` |
| POS | Till/device/session, sale headers, sale lines, cash movements | `tills`, `pos_devices`, `till_sessions`, `sales`, `sale_lines`, `cash_movements` |
| Payments and refunds | Payment capture records, allocations, gateway trace, refunds | `payments`, `payment_transactions`, `sale_payment_allocations`, `order_payment_allocations`, `refunds` |
| Orders and fulfillment | Cart/order creation, status history, delivery/pickup records | `carts`, `orders`, `order_items`, `order_status_history`, `deliveries`, `delivery_items` |
| Returns and exchanges | Return documents, exchange documents, allocations, stock impact | `returns`, `return_lines`, `exchanges`, `exchange_lines` |
| Receipts and audit | Receipt generation records, print logs, business audit | `receipt_templates`, `receipts`, `receipt_print_logs`, `audit_logs` |
| Offline sync | Sync batches, sync items, typed queues, conflicts, sync audit | `offline_sync_batches`, `offline_sync_items`, `offline_sale_sync_queue`, `offline_payment_sync_queue`, `offline_sync_conflicts` |
| Reporting | Read models and operational summaries | `daily_sales_summaries`, `daily_payment_summaries`, `daily_inventory_summaries`, `daily_discount_return_summaries` |


## Backend design rules

| Rule | Required behavior |
|---|---|
| Tenant isolation | Every tenant-owned operation must validate tenant context before reading or writing data. |
| Backend authority | Frontend guards and UI hiding are not security. Backend must recheck auth, tenant, outlet, permission, feature access, and business state. |
| Service orchestration | Controllers call application services. Services coordinate validation, repositories, Unit of Work, and domain rules. |
| Repository access | Repositories encapsulate EF Core queries and persistence. Controllers must not call repositories directly. |
| Domain purity | Domain entities/services must not depend on EF Core, HTTP, API DTOs, external gateways, or logging infrastructure. |
| Auditability | Sensitive actions must create audit records or link to audit-capable transaction records. |
| Offline safety | Offline payloads must be deduplicated, validated, and processed through server-side sync records. |
| Financial integrity | Sales, payments, refunds, discounts, tax, receipts, and reporting totals must not be manually patched outside controlled services. |

## Feature implementation entry point

Before implementing any backend feature:

- Read [[01-product/project-scope]].
- Read the related entity file under [[03-data/README]].
- Read the related API rule under [[04-api/README]].
- Read this folder's relevant backend rule files.
- Read the related module feature spec under `07-modules/`.
- Use [[05-backend/backend-implementation-checklist]] as the final gate.

## AI IDE rule

AI IDE tools must not create backend code from this folder alone.
They must cross-check product scope, database design, API rules, backend rules, security rules, and feature specs before editing code.
