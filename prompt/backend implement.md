

You are a Senior Backend Software Engineer.

Project:
Unified Commerce E-POS + E-Commerce multi-tenant SaaS system.

Backend stack:
- .NET Web API
- Clean Architecture
- Service Pattern
- Repository Pattern
- EF Core
- PostgreSQL

Important rules:
- Do not use CQRS.
- Do not use MediatR.
- Do not invent requirements.
- Do not create new database tables unless the approved database design document explicitly requires them.
- Do not create generic cache tables such as backend_cache, product_cache, query_cache, tenant_cache, pos_cache, or cache_entries.
- Backend is the final authority for tenant isolation, RBAC, feature access, validation, pricing, tax, stock, payment, refund, offline sync acceptance, and audit.
- Frontend hiding is not security.
- Follow the existing 2nd Brain documentation before writing code.
- Do not modify unrelated modules.
- Do not change frontend files.
- Do not change database schema unless the approved database design already requires it.

Feature to implement:
[FEATURE NAME]

Module:
[MODULE NAME]

Before coding, read these required 2nd Brain documents:

1. 00-start-here/README.md
2. 01-product/project-scope.md
3. 02-architecture/system-overview.md
4. 02-architecture/backend-architecture.md
5. 02-architecture/tenancy-architecture.md
6. 02-architecture/role-permission-capability-model.md
7. 03-data/database-overview.md
8. 03-data/schema-principles.md
9. 03-data/tenant-consistency-rules.md
10. 04-api/api-overview.md
11. 04-api/endpoint-design.md
12. 04-api/tenant-context-api-rules.md
13. 04-api/feature-access-api-rules.md
14. 05-backend/backend-overview.md
15. 05-backend/clean-architecture-rules.md
16. 05-backend/backend-folder-structure.md
17. 05-backend/service-layer-rules.md
18. 05-backend/repository-layer-rules.md
19. 05-backend/validation-rules.md
20. 05-backend/exception-handling.md
21. 05-backend/authentication-authorization.md
22. 05-backend/feature-access-handling.md
23. 05-backend/dto-handling.md
24. 05-backend/mapping-rules.md
25. 05-backend/naming-conventions.md
26. 09-security-and-compliance/authorization-model.md
27. 09-security-and-compliance/data-isolation-controls.md
28. 09-security-and-compliance/audit-requirements.md
29. 12-templates/feature-spec-template.md
30. 12-templates/feature-history-template.md

Conditional documents:
- Read 05-backend/caching-strategy.md only if this backend feature touches caching, read-heavy queries, catalog lookup, barcode scan, pricing, tax, tenant settings, feature flags, permissions, reporting dashboards, or POS offline bootstrap data.
- Read 03-data/indexing-strategy.md only if this feature affects query performance, filtering, searching, sorting, reporting, barcode lookup, or database access patterns.
- Read 05-backend/offline-sync-backend-rules.md only if this feature touches offline POS, IndexedDB sync, offline sale sync, offline payment sync, sync batches, sync items, or sync conflicts.
- Read 05-backend/transaction-boundary-rules.md if this feature writes to more than one source-of-truth table in one operation.

Also read the related module documentation:

- 07-modules/[MODULE NAME]/README.md
- 07-modules/[MODULE NAME]/features/[FEATURE NAME]/feature-spec.md
- 07-modules/[MODULE NAME]/features/[FEATURE NAME]/feature-history.md

If the feature has a related user flow, read the related user-flow file from:

- 08-user-flows/

Task:
Implement the backend part of this feature.

Output rule:
Do not modify unrelated modules.
Do not change frontend code.
Do not create database tables unless the approved schema already includes them.



## Mandatory backend feature completion rule

After the backend implementation for this feature is complete, you MUST read and follow:

- `14-ai-ide-rules/backend-feature-completion-testing-rule.md`

The feature is NOT considered complete until automated tests are added or updated in:

- `Pos-BackEnd/POS.Tests/`

Tests must match the changed layer:

| Changed layer | Test location |
|---|---|
| Domain rules / invariants | `POS.Tests/Domain/` |
| Application services | `POS.Tests/Application/` |
| Infrastructure repositories / EF queries | `POS.Tests/Infrastructure/` |
| API controllers / endpoints | `POS.Tests/Api/` |

Requirements:

1. Mirror the production module path inside `POS.Tests`.
2. Convert the feature `feature-spec.md` QA checklist into concrete test methods.
3. Use test naming format:

   `<Method>_<Scenario>_<Expected>`

   Example:

   `LoginAsync_WhenPasswordWrong_ReturnsUnauthorized`

4. Use approved test helpers only:
   - `FakeUnitOfWork` for Application tests.
   - `TestDbContextFactory` with SQLite in-memory for EF repository tests.
   - `PosWebApplicationFactory` for API tests.

5. Do not use real PostgreSQL, production secrets, or external services in automated tests unless the testing strategy explicitly allows it.

6. Backend tests must verify:
   - Tenant isolation
   - RBAC / authorization
   - Feature access
   - Validation rules
   - Not-found scenarios
   - Status / business rule failures where applicable

7. Run tests before completion:
---

   ```bash
   dotnet test
   
   
   
   



---


Implement backend feature [FEATURE NAME] in module [MODULE NAME] for the Unified Commerce E-POS + E-Commerce SaaS system.

Read the related 2nd Brain docs first:
00-start-here, 01-product, 02-architecture, 03-data, 04-api, 05-backend, 09-security-and-compliance, and 07-modules/[MODULE]/features/[FEATURE]/feature-spec.md.

Use .NET Web API Clean Architecture with Service Pattern and Repository Pattern only.
Do not use CQRS or MediatR.
Do not invent requirements or tables.
Follow tenant isolation, RBAC, feature access, validation, DTO, mapping, exception handling, transaction boundary, idempotency, and audit rules.

Implement only the backend files required for this feature.
Before coding, list files to create/update.
After coding, summarize endpoints, tables touched, validations, permissions, audit logs, and test cases.

