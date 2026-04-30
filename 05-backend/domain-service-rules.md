---
title: Domain Service Rules
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

# Domain Service Rules

Domain services are allowed only for pure business rules that do not belong cleanly to a single entity.
The uploaded backend architecture includes an example `OrderDomainService.cs` under the Domain layer.
This file defines when and how domain services may be used.

## Domain service definition

A domain service:

- lives in `POS.Domain`;
- contains pure business logic;
- does not use EF Core;
- does not call repositories;
- does not know HTTP or API DTOs;
- does not call external providers;
- does not write audit logs directly;
- can be called by Application services.

## When to use a domain service

| Use domain service when | Example in this system |
|---|---|
| Rule spans multiple domain objects | Validate an exchange difference based on returned value and new item value. |
| Rule is pure and reusable | Validate order status transition based on allowed statuses. |
| Rule should not live in controller/service | Calculate whether a return line quantity exceeds sold/returned quantity. |
| Rule has no persistence dependency | Determine payment allocation balance from passed values. |

## When not to use a domain service

| Do not use domain service for | Use instead |
|---|---|
| Loading products/orders from database | Repository through Application service |
| Calling gateway payment provider | Infrastructure service through Application service |
| Reading current tenant/user | Application context service |
| Writing audit logs | Application/audit service |
| Formatting API response | API/Application DTO mapper |
| Frontend calculation preview | Frontend shared kernel only; backend remains final authority |

## Candidate domain service areas

These are candidates because the uploaded scope/database includes rules that span records.
They must still be implemented only when a feature spec requires them.

| Area | Possible domain service | Pure rule examples |
|---|---|---|
| Orders | `OrderDomainService` | Status transition, cancellation eligibility, order total consistency. |
| Payments | `PaymentDomainService` | Allocation cannot exceed captured amount, refund cannot exceed original payment. |
| Inventory | `InventoryDomainService` | Movement direction validation, available quantity calculation from passed values. |
| Discounts | `DiscountDomainService` | Discount amount limit and stacking decision from passed policy. |
| Returns | `ReturnDomainService` | Return quantity eligibility and return window decision when values are passed. |
| Exchanges | `ExchangeDomainService` | Difference direction: collect, refund, none. |
| Offline sync | Usually Application service | Needs persistence and conflict records, so not pure domain except simple validation helpers. |

## Domain service shape

```text
POS.Domain/Modules/Orders/
├── Order.cs
├── OrderItem.cs
└── OrderDomainService.cs
```

## Example responsibility split

| Rule | Domain service? | Reason |
|---|---:|---|
| `Delivered` order cannot move back to `Processing` without controlled correction | Yes, if statuses are passed in. | Pure workflow rule. |
| Load order status history from database | No | Repository responsibility. |
| Create order status history record | No | Application workflow responsibility. |
| Calculate exchange difference direction from old/new totals | Yes | Pure business rule. |
| Issue refund through gateway | No | Infrastructure integration. |

## Domain entity vs domain service

| Rule location | Use when |
|---|---|
| Entity method | Rule only affects one aggregate/entity and its owned data. |
| Domain service | Rule coordinates multiple entities/value inputs but remains pure. |
| Application service | Rule requires repositories, current user, current tenant, feature flags, Unit of Work, or audit. |

## Naming rules

- Use `*DomainService` suffix.
- Place service in owning domain module.
- Keep method names business-readable.
- Do not use infrastructure names such as `Db`, `Repository`, `Http`, or `Gateway` in domain service methods.

## Inputs and outputs

Domain service methods should receive:

- domain entities;
- value objects;
- primitive values that represent business facts;
- collections already loaded by Application layer.

They should return:

- boolean decisions;
- calculated value objects;
- domain result objects;
- domain error/result types if the project uses them.

## Prohibited dependencies

```text
Domain service must not reference:
- Microsoft.EntityFrameworkCore
- ASP.NET Core HTTP abstractions
- API request/response classes
- Infrastructure repositories
- Payment provider SDKs
- Logging implementations
- Current user/session providers
```

## Checklist

- [ ] Rule is pure business logic.
- [ ] Rule does not require database query.
- [ ] Rule does not require current HTTP context.
- [ ] Rule does not require tenant feature lookup.
- [ ] Rule belongs to a real uploaded scope/database workflow.
- [ ] Application service calls the domain service.
- [ ] Domain service has no external dependencies.
