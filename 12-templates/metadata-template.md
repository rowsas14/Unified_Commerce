---
title: Metadata Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, metadata, documentation-rules]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend: Back end Architecture final.txt
source_frontend: Frontend archi V1.txt
---

# Metadata Template

Use this template at the top of every Markdown file in the Unified Commerce 2nd Brain.
Metadata makes documents searchable, reviewable and safe for AI IDE tools.

## Copy block

```yaml
---
title: <Human-readable title>
owner: <Product Owner | Architect | Backend Team | Frontend Team | QA Team | DevOps Team | Documentation Owner>
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce]
module: <module-folder-name-or-cross-cutting>
feature: <feature-folder-name-or-none>
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend: Back end Architecture final.txt
source_frontend: Frontend archi V1.txt
related_docs:
  - [[00-start-here/README]]
  - [[01-product/project-scope]]
---
```

## Required metadata fields

| Field | Required | Purpose |
|---|---:|---|
| `title` | Yes | Human-readable document title. |
| `owner` | Yes | Role/team accountable for maintaining the file. |
| `status` | Yes | Current review state. |
| `last_reviewed` | Yes | Last meaningful content review date. |
| `tags` | Yes | Search and grouping labels. |
| `module` | Recommended | Module or cross-cutting area. |
| `feature` | Recommended for feature docs | Feature folder name. |
| `source_scope` | Recommended | Scope source document used. |
| `source_database` | Recommended | Database source document used. |
| `source_backend` | Recommended | Backend architecture source when relevant. |
| `source_frontend` | Recommended | Frontend architecture source when relevant. |
| `related_docs` | Recommended | Wiki links to required context. |

## Accepted status values

| Status | Meaning | Who can use it |
|---|---|---|
| `draft` | Content started but not approved. | Any writer. |
| `in-review` | Ready for technical/product review. | Writer or reviewer. |
| `production-ready` | Approved as implementation baseline. | Owner after review. |
| `deprecated` | Replaced or no longer valid. | Owner only. |

## Owner options

Use role-based ownership, not personal names, unless the team explicitly decides otherwise.

| Owner | Use for |
|---|---|
| `Product Owner` | Scope, user flows, business rules. |
| `Solution Architect` | Architecture, module boundaries, cross-cutting decisions. |
| `Backend Team` | API, services, database usage, integrations. |
| `Frontend Team` | React architecture, POS UI, e-commerce UI, state. |
| `QA Team` | Test strategy and test cases. |
| `DevOps Team` | Deployment, monitoring, backup, runbooks. |
| `Documentation Owner` | Templates, indexes, folder guides. |

## Tag conventions

Use predictable tags.

| Tag | Use for |
|---|---|
| `unified-commerce` | All project docs. |
| `multi-tenant` | Tenant isolation or tenant-owned records. |
| `pos` | POS sales, sessions, devices, receipts. |
| `ecommerce` | Storefront, cart, orders, fulfillment. |
| `offline-sync` | Offline POS, queues, conflicts. |
| `rbac` | Roles, permissions, feature access. |
| `database` | Entity/table/reference docs. |
| `api` | Endpoint and contract docs. |
| `backend` | .NET Clean Architecture rules. |
| `frontend` | React, UI, state, routing docs. |
| `qa` | Testing and release readiness. |
| `audit` | Sensitive action traceability. |

## Module field values

Use the folder name, not a casual title.

Examples:

```yaml
module: sales-pos
module: ecommerce-orders
module: offline-sync
module: identity-access
module: pricing
module: tax
```

## Feature field values

Use the feature folder name.

Examples:

```yaml
feature: pos-checkout
feature: payments
feature: order-status-transitions
feature: offline-sale-sync
feature: customer-memberships
```

## Source document fields

Only include source fields that were actually used.

Examples:

```yaml
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
```

If a file is purely frontend architecture, include:

```yaml
source_frontend: Frontend archi V1.txt
```

If a file is backend architecture, include:

```yaml
source_backend: Back end Architecture final.txt
```

## Related docs field

Use wiki links to point readers to required context.

Example:

```yaml
related_docs:
  - [[01-product/project-scope]]
  - [[03-data/database-overview]]
  - [[04-api/api-overview]]
  - [[05-backend/backend-overview]]
```

## Metadata examples

### Feature spec example

```yaml
---
title: POS Checkout Feature Spec
owner: Product Owner
status: in-review
last_reviewed: 2026-04-30
tags: [unified-commerce, pos, sales-pos, payments, inventory]
module: sales-pos
feature: pos-checkout
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
related_docs:
  - [[07-modules/sales-pos/README]]
  - [[03-data/entities/pos-device-sales-entities]]
  - [[08-user-flows/cashier/scan-add-pay]]
---
```

### Entity reference example

```yaml
---
title: Payments Entity Reference
owner: Backend Team
status: production-ready
last_reviewed: 2026-04-30
tags: [unified-commerce, database, payments, refunds]
module: payments
source_database: Unified_Commerce_Database_Design_final V2.docx
related_docs:
  - [[03-data/database-overview]]
  - [[07-modules/payments/README]]
---
```

### User flow example

```yaml
---
title: Cashier Scan Add Pay Flow
owner: Product Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [unified-commerce, pos, cashier, checkout]
module: sales-pos
feature: pos-checkout
source_scope: Unified_Commerce_Scope_V1.docx
related_docs:
  - [[07-modules/sales-pos/features/pos-checkout/feature-spec]]
  - [[10-testing-quality/pos-ux-test-cases]]
---
```

## Review date rules

Update `last_reviewed` only when content changes meaningfully.
Do not update it for spacing-only edits unless the file is otherwise reviewed.

## Deprecated document metadata

When replacing a document, do not delete historical content immediately.
Mark it as deprecated and link to the replacement.

```yaml
status: deprecated
replaced_by: <wiki-link-to-replacement-document>
```

## AI IDE safety rule

AI IDE tools should treat files with `status: production-ready` as stronger authority than files with `status: draft`.
If two files conflict, the AI IDE must stop and ask for clarification or create a decision record using [[12-templates/decision-record-template]].

## Checklist

Before saving metadata:

- [ ] Title is specific.
- [ ] Owner is role-based.
- [ ] Status is valid.
- [ ] Date uses `YYYY-MM-DD`.
- [ ] Tags include `unified-commerce`.
- [ ] Module name matches folder name.
- [ ] Feature name matches feature folder, if applicable.
- [ ] Source documents are listed only when used.
- [ ] Related docs use wiki links.
- [ ] No fake citations or placeholder reference IDs are included.
