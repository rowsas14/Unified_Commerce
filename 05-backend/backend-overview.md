---
title: Backend Overview
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

# Backend Overview

The backend is the final authority for the Unified Commerce E-POS + E-Commerce SaaS system.
It must enforce tenant isolation, role and permission access, feature access, workflow status rules, validation, transaction safety, offline sync acceptance, financial integrity, and auditability.

This project is a production-ready system, not a basic POS or MVP.
The backend must support POS, e-commerce, shared catalog, outlet inventory, tax/pricing, payments/refunds, returns/exchanges, fulfillment, cash sessions, receipt generation, feature flags, reporting, and offline sync.

## Architecture baseline

The uploaded backend architecture defines a Clean Architecture design with:

- `POS.Api` for controllers, request/response contracts, middleware, filters, and extension registration.
- `POS.Application` for services, DTOs, validators, interfaces, strategies, and business orchestration.
- `POS.Domain` for pure domain models, entities, value objects, and domain services.
- `POS.Infrastructure` for persistence, repositories, external integrations, and Unit of Work.

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


## Backend responsibilities

| Responsibility | Backend requirement |
|---|---|
| Authentication | Validate platform user, tenant staff, and customer access flows. |
| Authorization | Check roles, permissions, outlet assignment, and sensitive action permission. |
| Feature access | Check tenant entitlement, role feature assignment, and runtime feature flags where applicable. |
| Tenant isolation | Enforce tenant boundaries for every tenant-owned table and query. |
| Catalog integrity | Protect product, variant, SKU, barcode, tax, price, return policy, and channel visibility rules. |
| Inventory integrity | Write stock movements and balances through controlled services only. |
| POS integrity | Validate till session, device, outlet, sale state, payment allocation, stock deduction, and receipt generation. |
| Payment integrity | Keep payment records, gateway events, allocations, refunds, and idempotency consistent. |
| Offline sync | Accept only valid offline payloads through sync batches/items and conflict records. |
| Reporting | Keep read models derived from source transactions, not manually edited totals. |
| Audit | Record sensitive actions and preserve traceability. |

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


## Main backend flows

### POS sale completion

```text
POS API request
  -> authenticate user and terminal context
  -> validate tenant, outlet, till session, feature access, permission
  -> validate products, variants, price, tax, discounts
  -> create sale and sale lines
  -> create payment and sale payment allocations
  -> create stock movements
  -> create receipt record
  -> update reporting projections when required
  -> commit through Unit of Work
```

### E-commerce order placement

```text
Storefront API request
  -> validate customer or guest identity
  -> validate tenant and channel context
  -> validate cart, variants, price, tax, stock, delivery method
  -> create order and order items
  -> reserve stock where required
  -> record payment status and allocation
  -> record order status history
  -> commit through Unit of Work
```

### Offline POS sync

```text
Sync batch received
  -> validate device, tenant, outlet
  -> store offline_sync_batch and offline_sync_items
  -> deduplicate by device + client ids
  -> process typed sale/payment queue records
  -> accept valid records into source-of-truth tables
  -> create conflict records for invalid/conflicting records
  -> write sync audit logs
```

## Layer ownership summary

| Layer | Owns | Does not own |
|---|---|---|
| API | HTTP contract, authentication middleware, controller routing, request binding | Business workflow, EF Core persistence, financial calculations as final authority |
| Application | Use-case orchestration, validation coordination, services, DTOs, transaction boundaries | HTTP-specific logic, SQL-specific details inside controllers, UI state |
| Domain | Pure rules, entities, value objects, domain services | Repositories, EF Core, external gateways, current user provider |
| Infrastructure | DbContext, repositories, external providers, Unit of Work | API contracts, UI workflows, domain business decisions |

## Required cross-folder references

| Need | Read |
|---|---|
| Product/module boundary | [[01-product/production-module-catalog]] |
| Architecture | [[02-architecture/backend-architecture]] |
| Entity rules | [[03-data/database-overview]] |
| API rules | [[04-api/api-overview]] |
| Security | [[09-security-and-compliance/authorization-model]] |
| Templates | [[12-templates/feature-spec-template]] |

## Backend non-goals

| Not allowed | Reason |
|---|---|
| CQRS handlers | Uploaded backend architecture uses service-based orchestration. |
| Mediator pipeline guidance | Not part of the provided architecture baseline. |
| Controller-heavy business logic | Violates Clean Architecture and makes testing harder. |
| Frontend-only validation | Backend must be final authority. |
| Direct table patching for totals | Financial and inventory records must remain traceable. |

## Done checklist for backend docs

- [ ] Feature has a module owner.
- [ ] Feature has database source tables identified.
- [ ] API request/response contract is documented.
- [ ] Service orchestration is defined.
- [ ] Repository access is scoped and tenant-filtered.
- [ ] Validation rules are documented.
- [ ] Permission and feature checks are documented.
- [ ] Transaction boundary is clear.
- [ ] Audit or history records are identified.
- [ ] Tests can be derived from the rules.
