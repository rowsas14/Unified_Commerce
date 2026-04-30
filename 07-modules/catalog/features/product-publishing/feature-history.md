---
title: Product Publishing Feature History
owner: Catalog Module Owner
status: active
last_reviewed: 2026-04-30
tags: [feature-history, catalog, product-publishing]
source: uploaded-scope uploaded-database backend-architecture frontend-architecture
---

# Product Publishing Feature History

## Purpose

This file records approved changes, bug fixes, implementation notes, and behavior decisions for
[[07-modules/catalog/features/product-publishing/feature-spec]].

It prevents developers and AI IDE tools from changing Catalog behavior without leaving a trace.

## Source baseline

| Source | Usage |
|---|---|
| Uploaded production scope | Defines Catalog as part of Product and Catalog Management, with products, variants, categories, brands, suppliers, attributes, pricing, images, channel visibility, return policy, and search. |
| Uploaded database design | Defines the tables and relationships used by this feature. |
| Backend architecture | Requires Clean Architecture with Service Pattern and Repository Pattern. |
| Frontend architecture | Requires React + TypeScript, TanStack Query, Zustand workflow state, offline/peripheral-aware POS behavior. |
| Current 2nd Brain | Provides cross-folder documentation rules and implementation references. |

## Current production baseline

| Area | Baseline |
|---|---|
| Module | [[07-modules/catalog/README]] |
| Feature spec | [[07-modules/catalog/features/product-publishing/feature-spec]] |
| Data reference | [[03-data/entities/product-catalog-entities]] |
| API rules | [[04-api/api-overview]] |
| Backend rules | [[05-backend/backend-overview]] |
| Frontend rules | [[06-frontend/frontend-overview]] |
| Security rules | [[09-security-and-compliance/authorization-model]] |

## Change log

| Date | Change type | Summary | Owner |
|---|---|---|---|
| 2026-04-30 | Documentation baseline | Created production-ready feature documentation aligned to uploaded scope and database design. | Documentation Writer |

## Bug history

| Date | Bug / issue | Impact | Resolution |
|---|---|---|---|
| 2026-04-30 | No implementation bug recorded in uploaded source documents. | None. | Keep this file ready for future real issues. |

## Decision history

| Date | Decision | Reason | Related doc |
|---|---|---|---|
| 2026-04-30 | Treat tenant isolation as mandatory for this feature. | Uploaded database design is tenant-scoped for tenant-owned catalog records. | [[03-data/tenant-consistency-rules]] |
| 2026-04-30 | Keep backend as final authority for validation and access. | Uploaded architecture and security context require server-side authorization and validation. | [[09-security-and-compliance/authorization-model]] |

## Implementation notes

- Do not add new tables, endpoints, permissions, or workflows here without approved source documentation.
- Any change that affects product sellability, price, tax, return policy, search, stock, POS billing, cart, order, return, exchange, or reporting must update related docs.
- If this feature is implemented, add notes about service class, repository, DTOs, validation, API contract, frontend screen/component, and tests.

## Regression watchlist

- Tenant isolation leakage.
- Cross-tenant FK assignment.
- SKU/barcode uniqueness breakage.
- Archived or inactive records appearing in active workflows.
- POS search becoming slow or website-like.
- E-commerce showing products not marked online-sellable.
- Catalog changes corrupting historical sale/order/return data.
- Frontend-only validation accepted without backend enforcement.

## Required update triggers

Update this history file when:

- A feature rule changes.
- A database column, FK, index, or constraint changes.
- An API contract is added or changed.
- A backend service/repository behavior changes.
- A frontend screen, cache, or workflow state changes.
- A bug is fixed.
- A QA scenario or test case is added.
- A production incident exposes a catalog behavior issue.

## Review checklist

- [ ] Feature spec still matches uploaded scope and database design.
- [ ] No undocumented tables or workflows were introduced.
- [ ] Internal links still resolve in Obsidian/GitHub.
- [ ] Related module docs were updated when behavior changed.
- [ ] QA and test documentation were updated when user-visible behavior changed.
