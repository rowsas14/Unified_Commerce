---
title: EF Core Implementation Notes
folder: 03-data
status: production-ready
owner: Backend / Data
tags: [ef-core, dotnet, postgres]
---

# EF Core Implementation Notes

These notes align the PostgreSQL database design with the .NET Clean Architecture backend.

---

## Project placement

| Concern | Correct layer |
|---|---|
| Entities and value behavior | `POS.Domain` |
| DTOs, validators, services, interfaces | `POS.Application` |
| DbContext, mappings, repositories, Unit of Work | `POS.Infrastructure` / Persistence |
| Controllers, requests, responses, middleware | `POS.Api` |

The backend architecture document shows Clean Architecture layers. In the final implementation, the API, Application, Domain, and Infrastructure projects should be separate projects/layers, not nested as folders inside the API project.

---

## PostgreSQL mapping rules

| Database feature | EF Core note |
|---|---|
| `uuid` PKs | Use `Guid` and database/client generation consistently. |
| `citext` | Enable PostgreSQL extension and map normalized email fields correctly. |
| `jsonb` | Use owned value objects or typed JSON payload classes where stable. |
| `numeric(12,2)` | Use `decimal`, never `double`, for money. |
| `numeric(14,3)` | Use `decimal` for stock quantities. |
| Generated column | Map `available_qty` as computed or service-maintained according to implementation. |
| Partial unique index | Configure with `HasFilter(...)` in EF Core. |
| Check constraints | Add with migration SQL or provider-supported constraints. |
| `inet` | Use suitable PostgreSQL mapping or string wrapper if needed. |

---

## Modeling rules

- Do not expose EF entities directly as API responses.
- Keep API request/response models in API layer and application DTOs in Application layer if both are used.
- Repository interfaces should live in Application abstractions when application services depend on them.
- Domain services must not access DbContext, HTTP, cache, or external providers.
- Transaction boundaries should wrap sale completion, payment allocation, stock movement, receipt generation, and sync acceptance where required.

---

## EF query rules

- Always filter by tenant for tenant-owned queries.
- Use projection DTOs for read screens and reports.
- Avoid loading full object graphs for POS scan/search.
- Use explicit includes only when necessary.
- Do not rely only on global query filters for security-sensitive tenant checks.

Related: [[tenant-consistency-rules]], [[indexing-strategy]], [[../05-backend/backend-architecture]].
