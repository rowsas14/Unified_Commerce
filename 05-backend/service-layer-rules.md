---
title: Service Layer Rules
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

# Service Layer Rules

The Application service layer is the main backend orchestration layer for this system.
The uploaded backend architecture explicitly includes services such as `ProductService`, `OrderService`, `PaymentService`, `CustomerService`, `InventoryService`, and `AuthService`.
This file defines how those services must behave in the production Unified Commerce backend.

## Service purpose

Application services coordinate use cases.
They do not act as simple pass-through wrappers around repositories.
They enforce the business workflow described by the product scope, database design, API rules, and security rules.

## Service responsibilities

| Responsibility | Required behavior |
|---|---|
| Context validation | Validate tenant, user, outlet, device, till session, and channel context where relevant. |
| Access checks | Check permission, role access, tenant feature entitlement, role feature assignment, and feature flags. |
| Input validation | Call the correct validator before business action. |
| Workflow orchestration | Coordinate repositories, domain services, Unit of Work, and external integrations. |
| Transaction management | Start or use Unit of Work for multi-table changes. |
| Audit trigger | Create or request audit records for sensitive actions. |
| Result shaping | Return application DTOs, not EF Core entities. |

## Service must not do

| Not allowed | Reason |
|---|---|
| Directly expose EF entities to API | Leaks persistence model. |
| Ignore tenant filters | Breaks multi-tenant isolation. |
| Put SQL/EF-heavy query details in service | Belongs in repository. |
| Put pure domain calculations in service when a domain service is needed | Weakens domain model. |
| Perform HTTP-specific response formatting | Belongs in API layer. |
| Use CQRS handlers | Not part of this project architecture. |

## Core service flow template

```text
Service method receives DTO + context
  -> validate authentication/session context
  -> validate tenant/outlet/channel/device context
  -> check permission and feature access
  -> run input validator
  -> load required source records using repositories
  -> run business validation and domain rules
  -> make state changes
  -> create audit/history/sync/ledger records where required
  -> commit Unit of Work
  -> return DTO/result
```

## POS sale service behavior

A POS sale completion service must coordinate:

- tenant and outlet validation;
- POS device and till session validation;
- product variant validation;
- price, tax, discount, and payment checks;
- `sales` and `sale_lines` creation;
- `payments` and `sale_payment_allocations` creation;
- stock movement creation;
- receipt generation record;
- audit for sensitive actions such as void or override.

## Order service behavior

An e-commerce order service must coordinate:

- customer or guest identity;
- cart validation;
- product visibility and variant availability;
- price/tax validation;
- stock reservation where required;
- payment status handling;
- order status history;
- fulfillment readiness.

## Payment service behavior

A payment service must coordinate:

- payment method validation;
- manual reference or provider configuration validation;
- idempotency key handling;
- payment record creation;
- provider transaction log creation where applicable;
- allocation to sale/order/exchange/refund records;
- refund limit validation.

## Inventory service behavior

An inventory service must coordinate:

- outlet and variant context;
- balance lookup;
- stock movement type validation;
- reservation hold/release;
- transfer, adjustment, stocktake, return, and sale stock effects;
- conflict creation for offline stock mismatch.

## Offline sync service behavior

Offline sync service must:

- create or update `offline_sync_batches`;
- store `offline_sync_items`;
- use typed queues for sale/payment records where appropriate;
- deduplicate by tenant/device/client entity IDs;
- process valid items into source-of-truth tables;
- create `offline_sync_conflicts` for invalid or conflicting items;
- write `offline_sync_audit_logs`.

## Service interface naming

| Service | Interface |
|---|---|
| `ProductService` | `IProductService` |
| `OrderService` | `IOrderService` |
| `PaymentService` | `IPaymentService` |
| `InventoryService` | `IInventoryService` |
| `AuthService` | `IAuthService` |
| `OfflineSyncService` | `IOfflineSyncService` |

## Service method naming

Use action names that match business behavior.

| Good | Avoid |
|---|---|
| `CompleteSaleAsync` | `SaveSaleAsync` when it hides payment/stock effects |
| `OpenTillSessionAsync` | `UpdateSessionAsync` for opening workflow |
| `ApproveDiscountRequestAsync` | `SetDiscountStatusAsync` without permission/audit meaning |
| `CreateReturnAsync` | `InsertReturnAsync` |
| `ProcessOfflineSyncBatchAsync` | `DoSyncAsync` |

## Checklist

- [ ] Service method has one clear business purpose.
- [ ] Service receives required context.
- [ ] Access checks happen before state changes.
- [ ] Validators are called.
- [ ] Repositories are used for data access.
- [ ] Unit of Work is used for multi-table changes.
- [ ] Sensitive actions are audited.
- [ ] Result is returned as DTO.
