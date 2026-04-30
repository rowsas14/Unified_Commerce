---
title: Clean Architecture Rules
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

# Clean Architecture Rules

This file defines the backend layer rules for the production Unified Commerce system.
The uploaded backend architecture shows a layered design with API, Application, Domain, Infrastructure, repositories, services, validators, DTOs, and Unit of Work.
These rules convert that source into enforceable implementation guidance.

## Non-negotiable implementation boundary

This backend documentation uses **Clean Architecture with Service Pattern and Repository Pattern only**.

| Do | Do not |
|---|---|
| Use services for use-case orchestration | Do not introduce CQRS command/query handlers |
| Use repositories for persistence access | Do not introduce Mediator/MediatR pipeline guidance |
| Use validators for request/application validation | Do not move business logic into controllers |
| Use Unit of Work for transactional workflows | Do not bypass tenant and feature access checks |
| Use domain services only for pure domain rules | Do not call EF Core from the Domain layer |


## Correct project boundary

The uploaded structure shows the layers under one tree, but production documentation must treat them as separate solution projects or separate top-level layer folders.
Do not document `POS.Application`, `POS.Domain`, and `POS.Infrastructure` as subfolders of `POS.Api`.

```text
src/
├── POS.Api/
├── POS.Application/
├── POS.Domain/
├── POS.Infrastructure/
└── POS.Tests/          # test project if/when test structure is created
```

## Layer dependency rule

```text
POS.Api            -> POS.Application
POS.Application    -> POS.Domain
POS.Infrastructure -> POS.Application + POS.Domain
POS.Domain         -> no project dependency on API/Application/Infrastructure
```

| Layer | May depend on | Must not depend on |
|---|---|---|
| API | Application contracts and services | Infrastructure repositories directly, EF Core DbContext directly |
| Application | Domain, application interfaces | API controllers, HTTP request classes, EF Core concrete DbContext |
| Domain | Nothing outside domain/common primitives | Application services, repositories, EF Core, gateways |
| Infrastructure | Application interfaces, Domain entities | API controller logic, frontend concerns |

## API layer rules

The API layer owns HTTP-facing concerns only.

| API concern | Allowed |
|---|---|
| Controllers | Route requests to application services. |
| Requests | API request models for HTTP input. |
| Responses | API response models or wrappers. |
| Middlewares | Exception handling, auth context, tenant context, request logging where applicable. |
| Filters | Request/action policies if needed. |
| Extensions | Service registration and application setup. |

API controllers must not:

- calculate final POS totals;
- decide stock movement rules;
- perform EF Core queries directly;
- create payment/refund records directly;
- bypass feature access checks;
- contain large business workflows.

## Application layer rules

The Application layer owns use-case orchestration.

| Application concern | Example |
|---|---|
| Service | `ProductService`, `PaymentService`, `InventoryService`, `AuthService` |
| DTO | Product, order, payment, customer, inventory data transfer models |
| Validator | Required fields, state checks, input range rules |
| Interface | Application contracts consumed by API or implemented by Infrastructure |
| Strategy | Only where uploaded architecture shows behavior switching, such as order/payment behavior |
| Factory | Strategy selection where required by documented workflow |

Application services must:

- receive tenant/user/outlet/device context;
- call validators;
- coordinate repositories;
- coordinate Unit of Work;
- call domain services when pure domain logic is needed;
- produce DTOs/responses for API layer;
- avoid HTTP-specific behavior.

## Domain layer rules

The Domain layer owns pure business meaning.

| Domain item | Use |
|---|---|
| Entity | Represents business state such as product, order, payment, customer, stock item. |
| Value object | Represents immutable business values where needed. |
| Domain service | Handles pure business rules that do not naturally belong to one entity. |
| Enum/value constants | Represents valid domain states when not database reference tables. |

Domain must not:

- know about HTTP;
- know about JSON payloads;
- know about EF Core migrations;
- call payment gateways;
- read current user context;
- write audit tables directly.

## Infrastructure layer rules

Infrastructure owns persistence and external systems.

| Infrastructure concern | Backend rule |
|---|---|
| Persistence | EF Core DbContext, entity configurations, migrations. |
| Repositories | Implement application repository interfaces. |
| Unit of Work | Commit multi-table changes atomically. |
| External services | Payment providers, printer bridge APIs if backend-integrated, messaging providers if configured. |
| Secret access | Use references/configuration; do not store secrets in plain JSON fields. |

## Feature module organization

Feature-based grouping is allowed inside each layer.

```text
POS.Application/
└── Modules/
    ├── Products/
    ├── Orders/
    ├── Payments/
    ├── Customers/
    ├── Inventory/
    ├── SalesPos/
    ├── ReturnsExchanges/
    ├── OfflineSync/
    └── SettingsConfiguration/
```

## Cross-cutting rules

| Cross-cutting concern | Where handled |
|---|---|
| Authentication | API middleware + Application auth service |
| Authorization | Application services using permission/feature context |
| Tenant context | API middleware/context provider; Application validates usage |
| Validation | API model validation + Application validators + Domain/database constraints |
| Exception handling | API middleware maps application/domain exceptions to API error contract |
| Transactions | Application service starts/coordinates Unit of Work |
| Auditing | Application service triggers audit writes through repository/service |

## Source-of-truth rule

The database design defines the source-of-truth tables.
The backend must not create hidden alternative stores for sales, payments, inventory, receipts, refunds, orders, offline sync, or audit.

| Data area | Source of truth |
|---|---|
| Sales | `sales`, `sale_lines` |
| Orders | `orders`, `order_items`, `order_status_history` |
| Payments | `payments`, `payment_transactions`, allocation tables |
| Stock | `stock_movements`, `inventory_balances`, `stock_reservations` |
| Returns | `returns`, `return_lines`, refund allocations |
| Exchanges | `exchanges`, `exchange_lines`, payment/refund allocations |
| Offline sync | `offline_sync_batches`, `offline_sync_items`, typed queues, conflict tables |
| Receipts | `receipt_templates`, `receipts`, `receipt_print_logs` |
| Audit | `audit_logs`, plus `offline_sync_audit_logs` for sync diagnostics |

## Review checklist

- [ ] No controller contains business workflow logic.
- [ ] No Domain class depends on EF Core or HTTP.
- [ ] Application service has a clear use case boundary.
- [ ] Repository interface is owned by Application or appropriate abstraction layer.
- [ ] Infrastructure implements repository contracts.
- [ ] Unit of Work wraps multi-table operations.
- [ ] Tenant context is passed and checked.
- [ ] Feature access and permission checks are not skipped.
- [ ] No CQRS or Mediator implementation has been introduced.
