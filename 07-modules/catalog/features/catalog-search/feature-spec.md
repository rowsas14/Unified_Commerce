---
title: Catalog Search Feature Spec
owner: Catalog Module Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [feature-spec, catalog, catalog-search]
source: uploaded-scope uploaded-database backend-architecture frontend-architecture
---

# Catalog Search Feature Spec

## Purpose

Supports barcode, SKU, name, category, brand, and channel-aware product discovery without becoming the source of truth.

This feature belongs to [[07-modules/catalog/README]]. It must stay aligned with the production scope,
the uploaded database design, the API rules, backend Clean Architecture rules, frontend POS/e-commerce rules,
and security/tenant isolation rules.

## Source references

- [[00-start-here/README]]
- [[01-product/project-scope]]
- [[01-product/production-module-catalog]]
- [[02-architecture/system-overview]]
- [[03-data/database-overview]]
- [[03-data/entities/product-catalog-entities]]
- [[04-api/api-overview]]
- [[05-backend/backend-overview]]
- [[06-frontend/frontend-overview]]
- [[09-security-and-compliance/authorization-model]]
- [[12-templates/feature-spec-template]]

## Actors

| Actor | Usage |
|---|---|
| Cashier / Catalog Manager / Customer | Primary actor for this feature. |
| Tenant Admin | May configure tenant-owned catalog settings or approve sensitive catalog setup where permitted. |
| Outlet Manager | May use catalog data for POS, outlet price, stock, or return workflow decisions where permitted. |
| Cashier | Reads allowed catalog/variant data during POS scan, search, return, and exchange flows. |
| E-Commerce Customer | Reads online-visible catalog data only through the storefront. |
| Backend System | Validates tenant consistency, feature access, rules, and persistence. |

## Permission and feature access rules

| Control | Rule |
|---|---|
| Tenant entitlement | Catalog-related platform feature must be enabled for the tenant when feature-gated. |
| Feature flag | Tenant/outlet/user-scoped runtime feature flag may further restrict behavior. |
| RBAC | User must have a role that grants the required catalog permission through the documented RBAC model. |
| Backend authority | Backend must check tenant, role, permission, feature entitlement, and runtime flag before writes. |
| Frontend behavior | UI may hide disabled actions, but hiding is not security. |
| Audit | Sensitive changes must be traceable through audit behavior where required by the security docs. |

## Database impact

| Table | Usage |
|---|---|
| `products` | Search by name, slug, descriptions, status, channel flags. |
| `product_variants` | Search by SKU, barcode, variant name, status. |
| `categories` | Filter/browse by active category. |
| `brands` | Filter/browse by active brand. |
| `product_images` | Optional thumbnail display. |
| `price_list_items` | Price display after pricing resolver applies channel and effective price rules. |
| `inventory_balances` | Optional stock display by outlet/variant. |

## Important relationships

| Relationship | Rule |
|---|---|
| Tenant ownership | Tenant-owned tables must include `tenant_id` and must never mix records across tenants. |
| Product to variant | `product_variants.product_id` must reference a product in the same tenant. |
| Product to category/brand | Optional references must belong to the same tenant. |
| Product to tax class | Tax class is referenced for calculation, but tax calculation is owned by the Tax module. |
| Product to return policy | Return policy is required by product and used by return/exchange workflows. |
| Variant to price | Price list items reference variants, not product masters. |
| Variant to stock | Inventory uses variants, but stock quantities remain outside Catalog. |

## Business rules

- Catalog records are tenant-owned unless the uploaded database design marks them as platform-owned templates.
- SKU and barcode uniqueness must be enforced inside the tenant boundary at product variant level.
- Do not store stock quantity on `products` or `product_variants`; stock belongs to inventory balances and movements.
- A product may be POS-only, e-commerce-only, or available in both channels through the documented sellable flags.
- Every sellable item used by POS, cart, order, return, exchange, stock, and pricing must resolve to a `product_variants.id`.
- Tax class and return policy references are part of catalog validation, but tax calculation and return execution are owned by their modules.
- Backend validation is the final authority; frontend validation only improves operator speed and user feedback.
- Catalog changes that affect selling, pricing, tax, returns, or channel visibility must be auditable.

## Workflow

1. User enters barcode, SKU, or search text depending on POS/admin/e-commerce context.
2. System restricts search by tenant and channel.
3. System returns products/variants that are active and allowed for the current workflow.
4. POS search prioritizes barcode/SKU and fast add behavior.
5. E-commerce search prioritizes online-visible product results and variant selection.

## Validation rules

- Search must never cross tenant boundary.
- POS search must use outlet/session context where stock or price display depends on outlet.
- E-commerce search must hide products not sellable online.
- Search result data must be treated as read model/display data; checkout validates again on backend.

## API impact

This feature must follow the API rules in [[04-api/api-overview]], [[04-api/endpoint-design]],
[[04-api/request-response-standard]], [[04-api/error-contract]], and [[04-api/tenant-context-api-rules]].

Do not treat this feature spec as the final API contract. The exact endpoint contract belongs in the API folder.
This feature only defines what the API must support for the module.

| API concern | Required behavior |
|---|---|
| Versioning | Catalog APIs must sit under the documented `/api/v1` versioning model. |
| Tenant context | Tenant must be resolved from authenticated context or approved tenant route/header strategy. |
| Request DTO | Request fields must be explicit and mapped to application DTOs. |
| Response DTO | Response must not expose internal-only or cross-tenant data. |
| Errors | Use stable validation, permission, feature-disabled, conflict, and not-found errors. |
| Pagination | List/search APIs must support documented pagination and filtering rules where lists are returned. |

## Backend impact

| Layer | Rule |
|---|---|
| Controller | Thin API controller; no domain or persistence logic. |
| Application service | Owns workflow orchestration, validation calls, repository calls, and transaction boundary. |
| Validator | Validates request shape, required fields, IDs, and basic business constraints. |
| Repository | Owns tenant-scoped persistence queries using EF Core/PostgreSQL rules. |
| Domain model/service | Holds pure business rules only where behavior belongs in domain layer. |
| Unit of Work | Used for multi-table catalog changes where persistence must be atomic. |

## Frontend impact

| Area | Rule |
|---|---|
| Admin UI | Must show loading, empty, validation error, permission denied, and save success states. |
| POS UI | Must remain fast and cashier-friendly; search/scan data must be optimized for operation. |
| E-Commerce UI | Must show only online-allowed product data. |
| TanStack Query | Server catalog state belongs in API queries/mutations. |
| Zustand | Use only for UI/workflow state, not as source of truth for catalog data. |
| Offline | Cached catalog data may support POS offline use; backend validates after sync. |

## QA checklist

- [ ] Tenant A cannot read or update Tenant B catalog records.
- [ ] Invalid FK references from another tenant are rejected.
- [ ] Feature-disabled users cannot write through API even if UI is manipulated.
- [ ] List/search behavior respects tenant, status, and channel rules.
- [ ] Audit-sensitive changes are traceable.
- [ ] Backend validation catches cases missed by the UI.
- [ ] Existing sale/order/stock history is not corrupted by catalog edits.
- [ ] API responses follow the documented response and error standard.

## Related Catalog specs

- [[07-modules/catalog/features/product-management/feature-spec]]
- [[07-modules/catalog/features/product-variant-management/feature-spec]]
- [[07-modules/catalog/features/category-management/feature-spec]]
- [[07-modules/catalog/features/brand-management/feature-spec]]
- [[07-modules/catalog/features/supplier-management/feature-spec]]

## Open questions

- No open question is added here unless it already appears in uploaded source documents or approved project context.
- If implementation discovers a missing rule, document it in [[13-project-history/decisions]] or the relevant feature history before coding assumptions.
