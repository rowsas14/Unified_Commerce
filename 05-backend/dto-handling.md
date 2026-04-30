---
title: DTO Handling
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

# DTO Handling

DTOs protect boundaries between API contracts, Application workflows, Domain entities, and Infrastructure persistence.
The uploaded backend architecture includes `ProductDto.cs`, `OrderDto.cs`, `PaymentDto.cs`, and `CustomerDto.cs` in the Application layer, while API modules contain Requests and Responses.

## DTO types

| Type | Layer | Purpose |
|---|---|---|
| Request model | API | HTTP input shape. |
| Response model | API | HTTP output shape. |
| Application DTO | Application | Use-case input/output between API and Application. |
| Domain entity | Domain | Business model, not an API DTO. |
| Persistence entity/config | Infrastructure/Domain depending implementation | EF Core mapped model; must not leak blindly. |

## Basic rule

API request/response objects are not the same as Domain entities.
Application DTOs can bridge API and Application use cases.
Domain objects should remain business-focused and not contain HTTP concerns.

## DTO flow

```text
HTTP Request
  -> API Request model
  -> Application DTO
  -> Domain entity/value/domain service where needed
  -> Repository/Persistence
  -> Application DTO result
  -> API Response model
```

## API request models

Request models should represent what the API accepts.
They may contain:

- primitive input values;
- IDs needed for route/body actions;
- field names aligned with API standards;
- no EF navigation objects;
- no server-calculated trusted totals unless treated as client preview only.

## API response models

Response models should return only what the caller needs.
They should not leak:

- password hashes;
- OTP hashes;
- provider secret references beyond safe identifiers;
- raw payment payloads unless explicitly needed for secure support workflows;
- cross-tenant data;
- internal EF tracking objects.

## Application DTOs

Application DTOs represent use-case data.
They may include data needed by services, validators, and mappers.
They should not contain HTTP-specific response codes or route information.

## Domain entities are not DTOs

Domain entities represent business state and behavior.
Do not bind HTTP requests directly to domain entities.
Do not return domain entities directly from controllers.

## DTO examples by module

| Module | Request/DTO concern |
|---|---|
| Products | Product creation, variant creation, SKU/barcode, channel visibility. |
| Orders | Cart conversion, order item details, address snapshot input. |
| Payments | Method, amount, reference, idempotency key, allocation target. |
| POS Sales | Sale lines, discounts, till session, client transaction ID for offline. |
| Returns | Source sale/order, line quantities, reason, restock action. |
| Offline sync | Batch metadata, item payloads, client entity IDs. |

## Server-calculated values

Client may send displayed totals for comparison, but backend must calculate final values for:

- price;
- tax;
- discount;
- payment balance;
- refund amount;
- exchange difference;
- inventory availability;
- receipt payload.

If request includes totals, treat them as client-side snapshot or validation aid, not authority.

## DTO validation

DTO validation belongs in validators, not random controller code.
See [[05-backend/validation-rules]].

## Naming convention

| Item | Naming pattern |
|---|---|
| API request | `CreateProductRequest`, `CompleteSaleRequest` |
| API response | `ProductResponse`, `SaleResponse` |
| Application DTO | `ProductDto`, `PaymentDto`, `OrderDto` |
| List item DTO | `ProductListItemDto` |
| Detail DTO | `ProductDetailDto` |

## Mapping references

Read [[05-backend/mapping-rules]] for mapping boundaries.

## Checklist

- [ ] Request model is not a domain entity.
- [ ] Response model does not expose secrets or hashes.
- [ ] Application DTO contains use-case data only.
- [ ] Backend recalculates financial totals.
- [ ] DTO fields map to documented database/API rules.
- [ ] DTO does not bypass tenant/access validation.
- [ ] Offline DTO includes client dedupe IDs where required.
