---
title: Mapping Rules
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

# Mapping Rules

Mapping converts data between API requests, Application DTOs, Domain entities, and response models.
This file prevents accidental leakage of persistence models and keeps backend boundaries clean.

## Mapping flow

```text
API Request
  -> Application DTO
  -> Domain entity / existing persisted entity update
  -> Application DTO result
  -> API Response
```

## Mapping ownership

| Mapping | Owner |
|---|---|
| API request to Application DTO | API layer or thin mapper. |
| Application DTO to Domain/Persistence model | Application service/mapper. |
| Domain/Persistence model to Application DTO | Application mapper/service. |
| Application DTO to API response | API layer or response mapper. |

## Mapping must not bypass validation

Mapping only transforms shape.
It does not prove that the request is valid.
Validation must happen through validators and business rules.

## Entity update rule

For updates, load the tenant-scoped entity from repository first, then apply allowed changes.
Do not create a new entity from request and attach it blindly.

```text
Load existing tenant-scoped entity
  -> validate status and permissions
  -> map allowed fields
  -> save through Unit of Work
```

## Sensitive field mapping

| Field type | Mapping rule |
|---|---|
| Password | Hash before persistence; never map plain password to response. |
| OTP | Store hash only; never return OTP hash. |
| Payment secret | Use `secret_ref`; do not map secrets into response. |
| Raw provider payload | Keep internal unless secure support use case exists. |
| Audit old/new values | Avoid storing secrets in audit snapshots. |

## Tenant field mapping

Tenant identity must come from authenticated/context data, not from untrusted body fields unless the endpoint is platform-admin tenant management.

| Scenario | Tenant source |
|---|---|
| Tenant staff product creation | Current tenant context. |
| POS sale | Current tenant + device/outlet/till session context. |
| Customer order | Storefront tenant context. |
| Platform tenant creation | Platform admin request body may define new tenant. |

## ID mapping rules

- Route ID must match body ID if both exist.
- Tenant-owned ID must be validated with tenant filter.
- Outlet ID must belong to tenant.
- Device ID must belong to tenant and outlet where relevant.
- Client IDs from offline mode are not server IDs.

## Financial mapping rule

Do not trust client-calculated monetary values as final.

| Client value | Backend action |
|---|---|
| Cart subtotal | Recalculate. |
| POS tax total | Recalculate or validate against backend rules. |
| Discount amount | Validate against policy/approval. |
| Payment amount | Validate against payable/refund/difference balance. |
| Exchange difference | Recalculate from return/new items. |

## Offline payload mapping

Offline payloads should first map into sync records:

```text
Client offline payload
  -> offline_sync_items.payload
  -> typed queue payload where applicable
  -> validated source-of-truth records only after acceptance
```

Do not directly trust offline payloads into `sales`, `payments`, or `stock_movements` without server validation.

## Response mapping rules

Responses should be useful but controlled.

| Response type | Include |
|---|---|
| Product list | id, name, SKU/barcode where relevant, status, price summary, stock summary if allowed. |
| Sale response | sale number, status, totals, payment summary, receipt reference. |
| Order response | order number, order/payment/fulfillment statuses, totals, items, address snapshot summary. |
| Payment response | status, method, amount, reference, allocation target. |
| Offline sync response | batch status, item statuses, conflict references. |

## Mapper placement

Keep mappers near the module that owns the data.
Avoid one huge global mapper file.

```text
POS.Application/Modules/Products/ProductMapper.cs
POS.Application/Modules/Payments/PaymentMapper.cs
POS.Application/Modules/OfflineSync/OfflineSyncMapper.cs
```

Exact file names are implementation details, but module ownership must remain clear.

## Checklist

- [ ] Mapping does not perform authorization.
- [ ] Mapping does not skip validation.
- [ ] Tenant ID is context-controlled.
- [ ] Sensitive fields are excluded from responses.
- [ ] Existing records are loaded before update mapping.
- [ ] Financial values are recalculated server-side.
- [ ] Offline payloads map through sync staging first.
