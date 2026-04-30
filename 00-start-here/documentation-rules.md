---
title: Documentation Rules
owner: Documentation Lead
status: production-ready
last_reviewed: 2026-04-30
tags:
  - documentation-rules
  - writing-standard
  - ai-ide
---

# Documentation Rules

This file defines how Markdown content must be written in the Unified Commerce 2nd Brain.

Every file must be source-aligned, practical, readable, and safe for production implementation.

---

## 1. Source-of-truth rule

| Source | Controls |
|---|---|
| Scope document | Production modules, actors, workflows, operational behavior, system boundaries. |
| Database design document | Table names, relationships, constraints, source-of-truth rules, offline sync, reporting models. |
| Backend architecture document | Clean Architecture layers, services, validators, DTOs, repositories, strategies, Unit of Work. |
| Frontend architecture document | React structure, guards, providers, layouts, features, shells, state, offline, peripherals. |
| Current 2nd Brain structure | Folder ownership and internal navigation. |

If a detail is missing, mark it as an open question or schema gap. Do not invent it as confirmed scope.

Related: [[source-document-alignment]]

---

## 2. Production language rule

Use:

- Production-ready Unified Commerce SaaS.
- Multi-tenant E-POS + E-Commerce platform.
- Offline-capable POS and online commerce platform.
- Tenant-scoped retail operations platform.

Avoid:

- Basic POS.
- Simple billing system.
- MVP-only project.
- Demo app.

If sequencing is needed, use “release wave”, “implementation priority”, “go-live required”, or “post-go-live enhancement”.

---

## 3. Front matter rule

Every Markdown file should start with:

```yaml
---
title: File Title
owner: Documentation Lead
status: draft | in-review | production-ready | superseded
last_reviewed: YYYY-MM-DD
tags:
  - tag-one
---
```

Use `production-ready` only when source-aligned and linked.

Update `last_reviewed` only when content changes.

---

## 4. Length rule

| File type | Normal length |
|---|---:|
| Folder README | 80–160 lines |
| Rule file | 120–220 lines |
| Feature spec | 150–220 lines |
| User flow | 100–180 lines |
| Test file | 120–220 lines |
| Runbook | 120–220 lines |

Shorter is acceptable for narrow files.

Longer is acceptable when genuinely required.

Do not add filler.

---

## 5. Naming and linking rule

Use lowercase kebab-case:

```text
payment-refund-api-rules.md
offline-sync-test-cases.md
feature-access-handling.md
```

Use Obsidian wiki links:

```text
See [[03-data/database-overview]].
See [[04-api/idempotency-rules|Idempotency Rules]].
```

Do not leave broken links.

Do not link archived files as active implementation rules.

---

## 6. Folder ownership rule

| Content | Folder |
|---|---|
| Product scope | [[01-product]] |
| System design | [[02-architecture]] |
| Database/entity rules | [[03-data]] |
| API rules | [[04-api]] |
| Backend implementation | [[05-backend]] |
| Frontend implementation | [[06-frontend]] |
| Feature ownership | [[07-modules]] |
| User workflow | [[08-user-flows]] |
| Security/compliance | [[09-security-and-compliance]] |
| Test cases | [[10-testing-quality]] |
| Delivery/support | [[11-delivery-and-operations]] |
| Templates | [[12-templates]] |
| Change history | [[13-project-history]] |
| AI IDE rules | [[14-ai-ide-rules]] |
| Old content | [[99-archive]] |

---

## 7. Module documentation rule

Each module under [[07-modules]] should contain:

```text
README.md
features/[feature-name]/feature-spec.md
features/[feature-name]/feature-history.md
```

A module README must include:

- Purpose.
- Scope.
- Owned features.
- Tables owned or touched.
- API ownership.
- Backend ownership.
- Frontend ownership.
- User flows.
- Permissions and feature access.
- Audit impact.
- Offline impact, if relevant.
- Related modules.
- What the module must not own.

---

## 8. Feature spec rule

Every `feature-spec.md` must include:

| Section | Meaning |
|---|---|
| Purpose | Why the feature exists. |
| Scope | Included and excluded behavior. |
| Actors | Users or systems involved. |
| Permissions | Role, permission, entitlement, flag. |
| Business rules | Production behavior. |
| Validation rules | Required fields, limits, blocked states. |
| Status rules | State transitions, if relevant. |
| Database impact | Tables read/written. |
| API impact | Endpoint expectations. |
| Backend impact | Services, validators, transactions, domain rules. |
| Frontend impact | Pages, components, stores, UI states. |
| Offline impact | Cache, queue, sync, conflict, or not applicable. |
| Audit impact | Sensitive action logging. |
| Reporting impact | Reports affected. |
| Acceptance criteria | Checkable completion rules. |
| Open questions | Unresolved items only. |

Placeholder feature specs are not implementation-ready.

---

## 9. Database rule

Use exact table names from the database design.

Examples:

- `tenants`
- `outlets`
- `document_sequences`
- `platform_features`
- `tenant_feature_entitlements`
- `role_feature_assignments`
- `products`
- `product_variants`
- `inventory_balances`
- `stock_movements`
- `sales`
- `sale_lines`
- `orders`
- `order_items`
- `payments`
- `refunds`
- `returns`
- `exchanges`
- `receipts`
- `offline_sync_batches`
- `offline_sync_items`

If a needed table is absent, document it in [[03-data/required-schema-extensions]].

---

## 10. Tenant isolation rule

Any file involving business data must answer:

- Is the record tenant-owned?
- Does it carry `tenant_id` directly?
- Does it inherit tenant context from a parent?
- Is outlet context required?
- Is channel context required?
- What backend check prevents cross-tenant access?

Frontend hiding is not tenant isolation.

Backend validation is required.

---

## 11. RBAC and feature access rule

Document feature access as:

```text
Tenant entitlement + runtime feature flag + role-feature assignment + permission + tenant/outlet context
```

The system supports tenant-specific configurable RBAC and feature permissions.

Related:

- [[02-architecture/role-permission-capability-model]]
- [[04-api/feature-access-api-rules]]
- [[05-backend/feature-access-handling]]
- [[06-frontend/feature-access-ui-rules]]

---

## 12. Backend rule

Backend docs must follow:

```text
POS.Api
POS.Application
POS.Domain
POS.Infrastructure
```

Rules:

- Controllers stay thin.
- Application services orchestrate use cases.
- Domain entities/services hold pure business rules.
- Validators handle input validation.
- Repositories are accessed through interfaces.
- Infrastructure implements database and external integrations.
- Unit of Work controls transactions.

Do not document application/domain/infrastructure as folders inside `POS.Api`.

---

## 13. Frontend rule

Frontend docs must follow:

```text
src/bootstrap
src/core
src/features
src/shells
src/pages
src/state
src/shared-kernel
```

Rules:

- TanStack Query owns server state.
- Zustand owns client workflow state.
- POS session/cart/offline state must be separated.
- Shared-kernel helpers may preview calculations.
- Backend remains final authority for price, tax, discount, payment, stock, receipt, and authorization.

---

## 14. Offline POS rule

Any POS sale, payment, receipt, stock, or session doc must explain:

- Cached data.
- Offline-allowed actions.
- Offline-blocked actions.
- Local IDs/idempotency keys.
- IndexedDB queue behavior.
- Sync endpoint behavior.
- Conflict cases.
- Audit/sync logs.

Offline queues are staging records only.

Accepted records must land in source-of-truth tables such as `sales`, `payments`, `stock_movements`, and `receipts`.

---

## 15. Financial and inventory rule

Never document behavior that allows:

- Refunds greater than captured payments.
- Payment totals to differ between frontend and backend.
- Stock deduction without stock movement.
- Silent offline stock overwrite.
- Receipt reprint without audit.
- Discount override without permission and reason.
- Reports to become source-of-truth records.

---

## 16. Acceptance criteria rule

Good:

```text
- [ ] Cashier cannot complete a sale without an open till session when session control is enabled.
- [ ] Completed sale creates sale, sale line, payment allocation, stock movement, and receipt records.
```

Bad:

```text
- [ ] System works correctly.
- [ ] UI is good.
```

---

## 17. AI IDE rule

Before backend work, read:

- [[14-ai-ide-rules/backend-implement]]
- [[05-backend/backend-overview]]
- [[03-data/database-overview]]
- Related module `feature-spec.md`

Before frontend work, read:

- [[14-ai-ide-rules/frontend-implement]]
- [[06-frontend/frontend-overview]]
- [[06-frontend/ui-ux-page-design-rules]]
- Related module `feature-spec.md`

Before full-stack work, read [[14-ai-ide-rules/fullstack-feature-implementation-rule]].

---

## 18. Correction checklist

When fixing a file:

- [ ] Preserve valid content.
- [ ] Remove duplicated or misleading content.
- [ ] Align to uploaded source documents.
- [ ] Add wiki links.
- [ ] Update `last_reviewed`.
- [ ] Record meaning-changing updates in [[13-project-history/production-alignment-change-log]].

---

## 19. Final review checklist

Before marking a file production-ready:

- [ ] It matches uploaded scope.
- [ ] It matches uploaded database design.
- [ ] It follows backend/frontend architecture where relevant.
- [ ] It uses correct table names.
- [ ] It belongs in the correct folder.
- [ ] It has useful wiki links.
- [ ] It includes tenant isolation where relevant.
- [ ] It includes access/security impact where relevant.
- [ ] It includes offline behavior where relevant.
- [ ] It is useful for humans and AI IDE tools.
