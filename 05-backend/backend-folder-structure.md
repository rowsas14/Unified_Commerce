---
title: Backend Folder Structure
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

# Backend Folder Structure

This file documents the production backend folder structure for the Unified Commerce system.
It is based on the uploaded backend architecture, corrected for Clean Architecture separation.

The uploaded source uses feature-based grouping in API and Application layers and separates API, Application, Domain, and Infrastructure responsibilities.
The final backend structure must keep that idea while avoiding layer nesting mistakes.

## Correct top-level solution structure

```text
src/
├── POS.Api/
├── POS.Application/
├── POS.Domain/
├── POS.Infrastructure/
└── POS.Tests/                  # optional test project structure when tests are added
```

## POS.Api structure

```text
POS.Api/
├── Modules/
│   ├── Auth/
│   │   ├── Controllers/
│   │   ├── Requests/
│   │   └── Responses/
│   ├── Products/
│   ├── Orders/
│   ├── Payments/
│   ├── Customers/
│   ├── Inventory/
│   ├── SalesPos/
│   ├── ReturnsExchanges/
│   ├── Fulfillment/
│   ├── OfflineSync/
│   ├── Reporting/
│   └── SettingsConfiguration/
├── Middlewares/
├── Filters/
├── Extensions/
├── Responses/
└── Program.cs
```

## API module contents

| Folder | Purpose |
|---|---|
| `Controllers/` | Thin HTTP controllers that call application services. |
| `Requests/` | HTTP request models, not domain entities. |
| `Responses/` | HTTP response models or API response wrappers. |
| `Middlewares/` | Exception handling, auth/tenant context, request pipeline concerns. |
| `Filters/` | Optional API-level filters. |
| `Extensions/` | Service registration and pipeline extension methods. |

## POS.Application structure

```text
POS.Application/
├── Modules/
│   ├── Products/
│   │   ├── ProductService.cs
│   │   ├── ProductDto.cs
│   │   ├── ProductValidator.cs
│   │   └── Interfaces/
│   ├── Orders/
│   │   ├── OrderService.cs
│   │   ├── OrderDto.cs
│   │   ├── OrderValidator.cs
│   │   ├── Strategies/
│   │   ├── Factories/
│   │   └── Interfaces/
│   ├── Payments/
│   ├── Customers/
│   ├── Inventory/
│   ├── Auth/
│   ├── SalesPos/
│   ├── DiscountsPromotions/
│   ├── ReturnsExchanges/
│   ├── Fulfillment/
│   ├── Receipts/
│   ├── OfflineSync/
│   └── Reporting/
└── Common/
    ├── Interfaces/
    ├── Responses/
    ├── Exceptions/
    └── Context/
```

## Application module contents

| Item | Rule |
|---|---|
| `*Service.cs` | Orchestrates one feature/use case family. |
| `*Dto.cs` | Application-facing data transfer model. |
| `*Validator.cs` | Validates command/request data before persistence. |
| `Interfaces/` | Service/repository/unit-of-work/context contracts. |
| `Strategies/` | Allowed only where behavior switching exists in source architecture, such as orders/payments. |
| `Factories/` | Used only to select documented strategies. |

## POS.Domain structure

```text
POS.Domain/
├── Modules/
│   ├── Products/
│   ├── Orders/
│   ├── Payments/
│   ├── Customers/
│   ├── Inventory/
│   ├── SalesPos/
│   ├── ReturnsExchanges/
│   └── CommonBusinessRules/
└── Common/
    ├── BaseEntity.cs
    └── ValueObjects/
```

## Domain structure rule

The uploaded backend source lists examples such as `Product.cs`, `Order.cs`, `OrderItem.cs`, `OrderDomainService.cs`, `Payment.cs`, `PaymentStatus.cs`, `Customer.cs`, `StockItem.cs`, and `StockStatus.cs`.
The production backend may expand module names to match the production scope, but must keep the same principle: pure business model only.

## POS.Infrastructure structure

```text
POS.Infrastructure/
├── Persistence/
│   ├── PosDbContext.cs
│   ├── Configurations/
│   └── Migrations/
├── Repositories/
│   ├── Products/
│   ├── Orders/
│   ├── Customers/
│   ├── Payments/
│   ├── Inventory/
│   ├── SalesPos/
│   └── OfflineSync/
├── Services/
│   ├── Payments/
│   ├── Messaging/
│   └── Files/
├── UnitOfWork/
└── Extensions/
```

## Infrastructure contents

| Folder | Purpose |
|---|---|
| `Persistence/` | EF Core DbContext, configurations, migrations. |
| `Repositories/` | Concrete repository implementations. |
| `Services/` | External provider integrations where the scope/database supports them. |
| `UnitOfWork/` | Transaction commit boundary. |
| `Extensions/` | Infrastructure service registration. |

## Module naming alignment

| Product module | Backend module name suggestion |
|---|---|
| Platform and Tenant Management | `TenantManagement` |
| Authentication, RBAC, Feature Access | `IdentityAccess` |
| Product and Catalog Management | `Catalog` or `Products` with clear ownership |
| Tax and Pricing Rules | `Tax`, `Pricing` |
| Inventory and Stock Management | `Inventory` |
| POS Device and Hardware | `PosDevicesHardware` |
| Cash Drawer, Shift, Session | `TillSessions` or `SalesPos` submodule |
| POS Sales and Checkout | `SalesPos` |
| Payments, Refunds, Receipts | `Payments`, `Receipts` |
| Discounts and Coupons | `DiscountsPromotions` |
| Returns and Exchanges | `ReturnsExchanges` |
| E-Commerce Orders | `Orders` or `EcommerceOrders` |
| Fulfillment | `Fulfillment` |
| Offline Sync | `OfflineSync` |
| Reporting and Audit | `Reporting`, `Audit` |

## Folder structure rules

- Keep API contracts out of Domain.
- Keep EF Core configurations out of Application and Domain.
- Keep API request/response classes out of Application unless intentionally reused as DTOs; prefer separate DTOs when workflows need them.
- Keep tenant context abstractions in Common/Application, implementation in API/Infrastructure.
- Keep repository interfaces near Application contracts, implementation in Infrastructure.
- Do not create CQRS folders such as `Commands`, `Queries`, `Handlers`, or `Mediators`.
- Do not create generic `Services` dumping grounds without module ownership.

## Review checklist

- [ ] Top-level layers are separate.
- [ ] Module folder names match product/database modules.
- [ ] API controllers are feature-grouped.
- [ ] Application services are feature-grouped.
- [ ] Domain models remain pure.
- [ ] Infrastructure owns persistence and external systems.
- [ ] No CQRS/Mediator folders exist.
- [ ] Unit of Work is available for multi-table operations.
