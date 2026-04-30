---
title: Repository Layer Rules
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

# Repository Layer Rules

Repositories are the persistence access boundary for the backend.
The uploaded backend architecture places repository implementations in Infrastructure and shows examples such as `ProductRepository`, `OrderRepository`, `CustomerRepository`, and `PaymentRepository`.

This project uses Repository Pattern with Service Pattern.
Repositories must support Clean Architecture without becoming a business logic layer.

## Repository purpose

| Repository responsibility | Explanation |
|---|---|
| Query source tables | Use EF Core to read/write database tables from the database design. |
| Apply tenant filters | Every tenant-owned query must be tenant-scoped. |
| Encapsulate persistence details | Hide EF Core configuration details from services. |
| Support transaction workflows | Participate in Unit of Work, not commit independently. |
| Return domain/application models | Do not return API request/response classes. |

## Repository must not do

| Not allowed | Reason |
|---|---|
| Check HTTP permissions directly | Authorization belongs in service/access layer. |
| Decide business workflow | Services/domain rules own workflow. |
| Commit database transaction independently | Unit of Work owns transaction commit. |
| Return `IQueryable` freely to controllers | Leaks persistence and tenant filtering risk. |
| Generate receipts or payment decisions | Business/application responsibility. |
| Write audit records without service intent | Audit must be tied to workflow meaning. |

## Interface location

Repository contracts should be visible to the Application layer and implemented by Infrastructure.

```text
POS.Application/Common/Interfaces/
  IProductRepository.cs
  IOrderRepository.cs
  IPaymentRepository.cs
  IUnitOfWork.cs

POS.Infrastructure/Repositories/
  Products/ProductRepository.cs
  Orders/OrderRepository.cs
  Payments/PaymentRepository.cs
```

## Tenant filtering rule

Every repository method for tenant-owned data must require tenant context or receive a tenant-scoped specification.

| Good method shape | Problem method shape |
|---|---|
| `GetByIdAsync(Guid tenantId, Guid id)` | `GetByIdAsync(Guid id)` for tenant-owned table |
| `FindBySkuAsync(Guid tenantId, string sku)` | `FindBySkuAsync(string sku)` |
| `GetOpenTillSessionAsync(Guid tenantId, Guid tillId)` | `GetOpenTillSessionAsync(Guid tillId)` |
| `GetOrderWithItemsAsync(Guid tenantId, Guid orderId)` | `GetOrderWithItemsAsync(Guid orderId)` |

## Repository families

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


## Read query rules

- Use tenant id in every tenant-owned query.
- Include outlet id where outlet ownership matters.
- Use pagination for list endpoints.
- Use indexes documented in [[03-data/indexing-strategy]].
- Avoid loading unnecessary navigation graphs.
- Prefer explicit includes/projections for response DTOs.
- Do not return cross-tenant data for convenience.

## Write command rules

- Do not call `SaveChanges` inside individual repositories when Unit of Work is used.
- Do not create side effects outside the intended aggregate/workflow.
- Use database constraints as final guards, but validate before hitting the database where possible.
- Keep immutable ledger/history rows append-only through repository methods.

## Special repository rules

| Area | Rule |
|---|---|
| Stock movements | Repository may insert movements, but service/domain decides movement type and reference. |
| Payments | Repository may store payment records, but service validates allocation and refund limits. |
| Offline sync | Repository stores sync items/conflicts; service decides accept/reject/conflict. |
| Audit logs | Repository writes audit entries requested by application workflow. |
| Reporting | Repository reads read models; read models must derive from source transactions. |

## Repository naming

| Entity area | Repository name |
|---|---|
| Products and variants | `IProductRepository`, `ProductRepository` |
| Orders and order items | `IOrderRepository`, `OrderRepository` |
| Payments/refunds | `IPaymentRepository`, `PaymentRepository` |
| Inventory | `IInventoryRepository`, `InventoryRepository` |
| POS sessions/sales | `ISalesPosRepository`, `SalesPosRepository` |
| Offline sync | `IOfflineSyncRepository`, `OfflineSyncRepository` |
| Feature access | `IFeatureAccessRepository`, `FeatureAccessRepository` |

## Review checklist

- [ ] Repository methods are tenant-scoped.
- [ ] Repository does not contain permission logic.
- [ ] Repository does not commit its own transaction.
- [ ] Repository does not expose unsafe `IQueryable` to API.
- [ ] Repository methods match database source tables.
- [ ] Repository read methods support pagination where needed.
- [ ] Ledger/history inserts are append-only.
