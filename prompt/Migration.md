
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

Task:
Read the approved 2nd Brain data documentation, create Domain entity models for all documented database tables, create EF Core configurations and relationships, add DbSet entries, then generate EF Core migration.

Important:
- This task is for Domain models, EF Core mappings, DbContext DbSet entries, and migration only.
- Do not implement API controllers.
- Do not implement frontend code.
- Do not implement services unless needed only to compile existing architecture.
- Do not add CQRS.
- Do not add MediatR.
- Do not invent requirements.
- Do not invent tables.
- Do not invent columns.
- Do not invent relationships.
- Do not invent indexes.
- Do not invent enum/status values.
- Use only tables, columns, PKs, FKs, indexes, constraints, and relationships already documented in the approved 2nd Brain `03-data/` folder and uploaded database design.

Before coding, read these documents.

Project and architecture:
- 00-start-here/README.md
- 00-start-here/source-document-alignment.md
- 01-product/project-scope.md
- 02-architecture/system-overview.md
- 02-architecture/backend-architecture.md
- 02-architecture/tenancy-architecture.md
- 02-architecture/tenancy-model.md
- 02-architecture/role-permission-capability-model.md
- 02-architecture/offline-first-architecture.md

Data and schema:
- 03-data/README.md
- 03-data/database-overview.md
- 03-data/schema-principles.md
- 03-data/entity-relationship-map.md
- 03-data/tenant-consistency-rules.md
- 03-data/enum-and-reference-data-policy.md
- 03-data/audit-model.md
- 03-data/soft-delete-policy.md
- 03-data/indexing-strategy.md
- 03-data/migration-strategy.md
- 03-data/ef-core-implementation-notes.md
- 03-data/required-schema-extensions.md
- Every Markdown file inside 03-data/entities/

Backend:
- 05-backend/backend-overview.md
- 05-backend/clean-architecture-rules.md
- 05-backend/backend-folder-structure.md
- 05-backend/naming-conventions.md
- 05-backend/domain-service-rules.md
- 05-backend/repository-layer-rules.md
- 05-backend/transaction-boundary-rules.md
- 05-backend/validation-rules.md
- 05-backend/mapping-rules.md
- 05-backend/backend-implementation-checklist.md

Security:
- 09-security-and-compliance/data-isolation-controls.md
- 09-security-and-compliance/audit-requirements.md
- 09-security-and-compliance/authorization-model.md

Schema source rule:
- Treat `03-data/entities/` as the approved table/entity source.
- Treat `03-data/required-schema-extensions.md` as the schema gap control file.
- If a table, column, relationship, index, constraint, enum/status value, or rule is not documented, do not create it.
- If the codebase needs something that is missing from the approved schema documentation, stop and report it as a schema gap.

Implementation order:
Follow the documented dependency order:

1. Tenant foundation
2. Identity, RBAC, and feature access
3. Tenant runtime configuration
4. Catalog, tax, and pricing
5. Inventory and stock control
6. POS devices, sessions, and sales
7. Customer, cart, and e-commerce orders
8. Fulfillment, pickup, and delivery
9. Payments, refunds, and receipts
10. Discounts, coupons, and approvals
11. Returns and exchanges
12. Receipts, audit, and offline sync
13. Reporting read models

Domain model rules:
- Create one Domain entity class for each approved table.
- Place entity classes in the correct Domain module folder.
- Keep Domain entities focused on business state and basic business behavior.
- Do not put EF Core Fluent API configuration inside Domain entity classes.
- Do not put repository logic inside Domain entity classes.
- Do not put DTOs inside Domain.
- Do not put API request/response models inside Domain.
- Do not expose database-specific infrastructure concerns inside Domain.
- Use clear C# naming while mapping to snake_case PostgreSQL table and column names in EF Core configuration.

EF Core configuration rules:
- Create Fluent API configuration classes in Infrastructure/Persistence.
- Configure table names exactly as documented.
- Configure primary keys exactly as documented.
- Configure foreign keys exactly as documented.
- Configure required/nullable fields exactly as documented.
- Configure unique constraints and indexes exactly as documented.
- Configure decimal precision for money, quantity, percentage, and rate fields.
- Configure `citext` where documented for email/normalized email fields.
- Configure `jsonb` where documented for payload/config/snapshot fields.
- Configure `inet` where documented for IP fields.
- Configure generated columns only where documented.
- Configure check constraints where documented and supported by EF Core/PostgreSQL provider.
- Configure relationship delete behavior deliberately; do not cascade-delete financial, audit, stock, payment, sales, order, return, receipt, or sync history unless explicitly documented.

DbContext rules:
- Add DbSet entries only for approved entities.
- Do not add DbSet for tables not documented.
- Keep DbContext inside Infrastructure/Persistence according to backend folder structure.
- Apply all entity configurations through configuration scanning or explicit registration according to project convention.
- Do not put business logic in DbContext.

PostgreSQL naming rules:
- Domain class names use PascalCase.
- C# properties use PascalCase.
- PostgreSQL tables use documented snake_case table names.
- PostgreSQL columns use documented snake_case column names.
- Join tables and allocation tables must keep documented names.
- Do not rename tables because of C# naming preference.

Tenant isolation rules:
- Every tenant-owned entity must include `TenantId` when documented.
- Every tenant-owned FK relationship must be configured so tenant consistency can be validated in service/repository logic.
- Never create cross-tenant relationships.
- Do not remove `tenant_id` from any documented tenant-owned table.
- Platform-owned reference tables must not receive `tenant_id` unless documented.

Relationship rules:
- Configure all documented one-to-many, many-to-one, and one-to-one relationships.
- Configure nullable FKs exactly as documented.
- Configure required FKs exactly as documented.
- Configure composite unique constraints where documented.
- Configure filtered/partial unique indexes where documented and supported.
- If EF Core cannot express a PostgreSQL-specific constraint cleanly, document it in the migration review notes.

Status/check rules:
- Use documented string/check status values.
- Do not create extra status enum values.
- Do not convert a documented varchar/check status column into a separate lookup table unless the database design already defines it.
- Seed only documented reference data where the data documentation requires seeded reference tables.

Source-of-truth rules:
- Do not create cache tables.
- Do not duplicate transactional source-of-truth data.
- Reporting read models are allowed only where documented.
- Offline sync queue/staging tables are not financial source of truth.
- Financial and inventory source-of-truth tables must be preserved exactly as documented.

Critical tables rule:
For these areas, do not simplify the model:
- sales and sale_lines
- orders and order_items
- payments and payment_transactions
- refunds and allocations
- stock_movements and inventory_balances
- returns and return_lines
- exchanges and exchange_lines
- receipt_templates, receipts, and receipt_print_logs
- audit_logs
- offline_sync_batches, offline_sync_items, offline sale/payment queues, conflicts, and sync audit logs
- reporting read models

Migration rules:
- After Domain entities, EF Core configurations, relationships, and DbSet entries are created, generate an EF Core migration.
- Migration name should be clear, for example:
  `InitialUnifiedCommerceProductionSchema`
- Review the generated migration before applying it.
- The migration must match the approved database design.
- Do not apply migration automatically until the generated migration is reviewed.
- Do not manually add unrelated SQL.
- If PostgreSQL-specific items require manual migration SQL, add only what is documented and explain why.
- Do not create generic cache tables in the migration.
- Do not create unapproved AI/import/peripheral tables unless they are documented in the approved schema.

Required output before coding:
Before making code changes, provide:

1. List of documents read.
2. List of entity documentation files found under `03-data/entities/`.
3. Planned Domain module folders.
4. Planned entity class count.
5. Planned EF Core configuration class count.
6. Planned DbSet entries.
7. Migration dependency order.
8. Any schema gaps or ambiguities found.

If there are schema gaps:
- Stop before coding the missing area.
- Report the gap clearly.
- Do not invent a solution.

Required output after coding:
After implementation, provide:

1. Domain entity classes created.
2. EF Core configuration classes created.
3. DbSet entries added.
4. Relationships configured.
5. Indexes and unique constraints configured.
6. Check constraints configured.
7. JSONB/citext/inet/generated-column mappings configured.
8. Migration file generated.
9. Any migration review notes.
10. Any schema gaps not implemented.
11. Any tables intentionally skipped and why.
12. Build/compile result.
13. Migration command used.
14. Whether migration was generated only or also applied.

Do not:
- Do not create API endpoints.
- Do not create frontend files.
- Do not implement feature services.
- Do not implement CQRS.
- Do not use MediatR.
- Do not create new requirements.
- Do not create unapproved schema.
- Do not remove important existing code.
- Do not modify unrelated modules.
- Do not use generic names like Entity1, BaseLookup, CommonCache, or DynamicTable for approved business tables.