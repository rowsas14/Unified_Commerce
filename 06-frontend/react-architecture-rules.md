---
title: React Architecture Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# React Architecture Rules

This document defines how React and TypeScript must be used in the Unified Commerce frontend.
It follows the uploaded frontend structure and the production system boundaries.

## Required reading

- [[00-start-here/README]] — entry point for the Unified Commerce 2nd Brain.
- [[01-product/project-scope]] — production scope and system boundary.
- [[01-product/production-module-catalog]] — product-level module map.
- [[02-architecture/frontend-architecture]] — source architecture for React frontend structure.
- [[02-architecture/offline-first-architecture]] — offline-first POS design.
- [[03-data/database-overview]] — source data model and table ownership.
- [[04-api/api-overview]] — API design and `/api/v1` rules.
- [[05-backend/backend-overview]] — backend authority, service, repository and transaction rules.
- [[09-security-and-compliance/authorization-model]] — RBAC, feature access and tenant isolation rules.

## Core rule

React components must present and orchestrate user workflows.
They must not become the final authority for business rules such as permission, feature access, stock, payments, refunds, tax, pricing or offline sync acceptance.

## Architecture layers

```mermaid
flowchart TD
  Page[Page Component] --> Layout[Layout]
  Page --> Shell[Shell]
  Shell --> FeatureComponent[Feature Components]
  FeatureComponent --> FeatureHook[Feature Hooks]
  FeatureHook --> Query[TanStack Query]
  FeatureHook --> Store[Zustand Store]
  Query --> API[/api/v1]
```

## Component categories

| Category | Location | Example responsibility |
|---|---|---|
| Page | `pages/` | Route-level composition such as POSPage or PaymentPage. |
| Layout | `bootstrap/layouts/` | POSLayout, AdminLayout, AuthLayout. |
| Shell | `shells/` | POS screen regions such as CartShell and PaymentShell. |
| Feature component | `features/*/components/` | Product search result, payment method selector, cart line row. |
| Shared component | Shared UI folder if present | Button, modal, table, empty state. |
| Provider | `bootstrap/providers/` | Query, theme and session providers. |

## Page rules

Pages must:

- compose layouts, shells and feature components;
- read route params and guard state;
- avoid direct business calculations;
- avoid direct raw `fetch` or Axios calls;
- avoid duplicating module services;
- show loading, error, empty and permission-denied states.

Pages must not:

- contain stock movement logic;
- contain payment validation logic;
- contain final tax or discount calculations;
- contain offline conflict resolution logic;
- mutate global state without a defined store action.

## Shell rules

Shells are used heavily in the POS experience.
They organize fixed operational panels such as product grid, basket, payment and receipt areas.

| Shell | Expected behavior |
|---|---|
| `POSHeaderShell` | Tenant, outlet, cashier, time, connectivity and session status. |
| `ProductGridShell` | Product search, scan feedback and quick-add product area. |
| `CartShell` | Current sale/cart lines, totals and action buttons. |
| `PaymentShell` | Payment method selection and payment entry. |
| `TillSessionShell` | Open/close session and lock-state visibility. |
| `NotificationShell` | Operational warnings, sync status and backend errors. |
| `ReceiptShell` | Receipt preview, print status and reprint controls. |

Shells should delegate data and actions to feature hooks/stores.

## Hook rules

Feature hooks must use clear names:

```text
useProductsQuery
useProductSearch
useCreateSaleMutation
useCartActions
usePaymentMethodsQuery
useOfflineSyncStatus
```

Hooks must:

- wrap TanStack Query for server state;
- wrap Zustand selectors for workflow state;
- expose business-safe actions to components;
- map backend errors into UI-ready messages;
- avoid hidden side effects that change unrelated modules.

## TypeScript rules

- Use explicit types for API responses and component props.
- Avoid `any` unless integrating with an unknown external browser/peripheral API and document why.
- Use discriminated unions for UI states such as `idle`, `loading`, `error`, `success`, `offline`, `conflict`.
- Keep backend DTO types separate from UI view models when the UI shape differs.
- Do not model EF Core database entities directly as frontend domain entities.

## Server state vs client state

| State type | Tool |
|---|---|
| Products from backend | TanStack Query |
| Price lists returned by API | TanStack Query |
| Active cart lines | Zustand |
| Selected customer for current POS sale | Zustand |
| Offline sync status | Zustand + connectivity monitor |
| Current tenant/outlet/session | Session store and backend session response |
| Reports | TanStack Query |

See [[06-frontend/state-management-rules]].

## Error handling rule

Components must not parse raw backend exceptions.
They should consume the standard API error contract documented in [[04-api/error-contract]].

## UI state pattern

Every data-driven component should cover:

| State | Required UI behavior |
|---|---|
| Loading | Skeleton or clear loading state. |
| Empty | Explain why no data appears. |
| Error | Show business-readable error from API contract. |
| Permission denied | Show restricted state, not broken UI. |
| Feature disabled | Show unavailable state when feature is not enabled. |
| Offline | Show cached/queued behavior where supported. |

## Accessibility and touch usability

The uploaded scope emphasizes cashier-speed and touchscreen-first POS.
React components in POS areas must:

- use large tap targets;
- avoid tiny icon-only critical actions;
- keep totals visible;
- keep scan/search input prominent;
- show action feedback immediately;
- avoid deep nested modals for common cashier actions.

## Architecture checklist

- [ ] Component is in the correct folder.
- [ ] Server state uses TanStack Query.
- [ ] Workflow state uses Zustand only where needed.
- [ ] API errors use the documented error contract.
- [ ] Feature access is reflected in UI.
- [ ] Backend remains final authority.
- [ ] POS components are fast and touch-friendly.
- [ ] No CQRS/Mediator concept is introduced into frontend docs.
