---
title: Naming Conventions
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

# Naming Conventions

Consistent naming helps developers, architects, QA engineers, product owners, and AI IDE tools understand and modify the backend safely.
Names must align with the production scope, database design, API docs, and Clean Architecture layers.

## Project names

| Layer | Name |
|---|---|
| API | `POS.Api` |
| Application | `POS.Application` |
| Domain | `POS.Domain` |
| Infrastructure | `POS.Infrastructure` |

## Module names

Use business module names, not vague technical names.

| Use | Avoid |
|---|---|
| `SalesPos` | `SalesStuff` |
| `OfflineSync` | `SyncThings` |
| `ReturnsExchanges` | `Returns2` |
| `SettingsConfiguration` | `ConfigsMisc` |
| `IdentityAccess` | `UsersAuthEtc` |

## Class suffixes

| Class type | Suffix |
|---|---|
| Controller | `Controller` |
| Service | `Service` |
| Service interface | `I*Service` |
| Repository | `Repository` |
| Repository interface | `I*Repository` |
| Validator | `Validator` |
| DTO | `Dto` |
| Request | `Request` |
| Response | `Response` |
| Domain service | `DomainService` |

## Method naming

Use business verbs.

| Good | Avoid |
|---|---|
| `CompleteSaleAsync` | `SaveAsync` for full sale workflow |
| `OpenTillSessionAsync` | `CreateSessionRecordAsync` |
| `ApproveDiscountRequestAsync` | `UpdateDiscountAsync` |
| `ProcessOfflineSyncBatchAsync` | `RunSyncAsync` |
| `CreateReturnAsync` | `InsertReturnAsync` |

## Database naming alignment

When referring to tables in docs/code comments, use exact table names from the database design:

- `tenant_feature_entitlements`
- `role_feature_assignments`
- `inventory_balances`
- `stock_movements`
- `offline_sync_batches`
- `offline_sync_items`
- `offline_sale_sync_queue`
- `offline_payment_sync_queue`
- `offline_sync_conflicts`
- `daily_sales_summaries`

## Permission naming

Permission examples in source documents use code style such as `pos.sale.create`.
Keep permission codes stable, lowercase, dot-separated, and module/action based.

## API route naming

API docs own exact route rules.
Backend route names must align with [[04-api/endpoint-design]] and use `/api/v1` versioning.

## Status naming

Use status names from the database design.
Do not invent alternate statuses in services.

## Checklist

- [ ] Name matches module ownership.
- [ ] Class suffix matches responsibility.
- [ ] Method name describes business action.
- [ ] Database table names match uploaded design.
- [ ] Status names are not invented.
- [ ] No CQRS/Mediator naming appears.
