---
title: Frontend Folder Structure
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Frontend Folder Structure

This document defines the approved React folder structure for the Unified Commerce frontend.
It is based on the uploaded frontend architecture and aligned with the production scope and backend/API boundaries.

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

## Approved root structure

```text
src/
├── bootstrap/
├── core/
├── features/
├── shells/
├── pages/
├── state/
└── shared-kernel/
```

No other top-level folder should be introduced unless the architecture documents are updated.

## Folder ownership table

| Folder | Owns | Must not own |
|---|---|---|
| `bootstrap/` | App startup, router, guards, providers, layouts | Business workflows or API logic |
| `core/` | Shared infrastructure helpers | Feature-specific UI or module workflows |
| `features/` | Feature-specific API hooks, components, services and types | Global app bootstrapping |
| `shells/` | POS screen composition regions | Business data fetching logic |
| `pages/` | Routed screens | Deep business logic or reusable domain helpers |
| `state/` | Zustand workflow stores | Server cache that belongs to TanStack Query |
| `shared-kernel/` | Shared frontend helpers for money, tax, pricing, receipt | Final backend business authority |

## `bootstrap/`

```text
bootstrap/
├── main.tsx
├── App.tsx
├── router/
│   ├── index.tsx
│   ├── routes.tsx
│   └── guards/
├── providers/
└── layouts/
```

### Rules

- `main.tsx` should only mount the app.
- `App.tsx` should compose providers and router.
- Guards belong under `bootstrap/router/guards`.
- Layouts must separate POS, admin and auth user experience.
- Do not put feature-specific API calls in bootstrap.

## `core/api`

```text
core/api/
├── http.ts
├── endpoints.ts
└── queryClient.ts
```

| File | Responsibility |
|---|---|
| `http.ts` | Shared HTTP client, base `/api/v1`, headers, auth and error mapping. |
| `endpoints.ts` | Central endpoint constants or builders. |
| `queryClient.ts` | TanStack Query client configuration. |

API-specific rules are documented in [[06-frontend/api-client-and-query-rules]].

## `core/auth`

```text
core/auth/
├── tokenManager.ts
└── sessionManager.ts
```

Use this folder only for frontend session handling.
Backend authentication rules remain in [[05-backend/authentication-authorization]] and [[09-security-and-compliance/authentication-model]].

## `core/offline`

```text
core/offline/
├── syncQueue.ts
└── connectivityMonitor.ts
```

Use this for offline detection and IndexedDB-backed queue orchestration.
It must align with [[06-frontend/offline-frontend-rules]] and [[04-api/offline-sync-api-rules]].

## `core/peripherals`

```text
core/peripherals/
├── printerBridge.ts
├── scannerListener.ts
└── cashDrawer.ts
```

This folder owns frontend browser/device integration helpers.
It must not become the source of truth for printer assignment or device registration.
Those records are handled by backend and data rules.

## `features/`

```text
features/
├── auth/
├── till-session/
├── products/
├── cart/
├── sales/
├── payments/
├── customers/
├── discounts/
├── returns/
├── inventory/
├── receipts/
└── config/
```

Each feature folder may contain:

```text
api/
components/
hooks/
services/
types/
index.ts
```

### Feature folder rules

- API hooks belong in `api/`.
- Presentational pieces belong in `components/`.
- Feature-specific hooks belong in `hooks/`.
- UI orchestration helpers belong in `services/` when they do not belong in Zustand.
- Types are frontend DTO/view types, not backend EF entities.

## `shells/`

```text
shells/
├── POSHeaderShell/
├── ProductGridShell/
├── CartShell/
├── PaymentShell/
├── CustomerShell/
├── DiscountShell/
├── ReturnShell/
├── TillSessionShell/
├── NotificationShell/
└── ReceiptShell/
```

Shells are layout composition units for operational POS screens.
They should remain thin and compose feature components.

## `pages/`

Approved source pages from uploaded frontend architecture:

| Page | Purpose |
|---|---|
| `LoginPage.tsx` | Authentication entry. |
| `OutletSelectPage.tsx` | Select or confirm outlet context. |
| `TillOpenPage.tsx` | Open till/session. |
| `POSPage.tsx` | Main cashier billing screen. |
| `PaymentPage.tsx` | Payment capture screen. |
| `ReturnPage.tsx` | Return/exchange operation entry. |
| `TillClosePage.tsx` | Close till/session. |
| `CashManagementPage.tsx` | Cash in/out and drawer operation. |
| `StocktakePage.tsx` | Stock count workflow. |
| `TransferPage.tsx` | Stock transfer workflow. |
| `EndOfDayPage.tsx` | End-of-day operational summary. |
| `ManagerDashboardPage.tsx` | Management dashboard. |

Do not add new pages without module/user-flow documentation.

## `state/`

```text
state/
├── app.store.ts
├── session.store.ts
├── till.store.ts
├── cart.store.ts
├── ui.store.ts
├── offline.store.ts
└── cart.orchestrator.ts
```

State rules are documented in [[06-frontend/state-management-rules]].

## `shared-kernel/`

```text
shared-kernel/
├── money/
├── tax/
├── pricing-engine/
├── receipt-engine/
└── invoice-engine/
```

Shared-kernel helpers are for frontend consistency and preview behavior.
They must never override backend-calculated production results.

## Folder change checklist

- [ ] Does the new file belong to an approved folder?
- [ ] Is the owning module documented in [[07-modules/README]]?
- [ ] Does the page have a matching user flow in [[08-user-flows/README]]?
- [ ] Are API paths documented in [[04-api/module-endpoint-map]]?
- [ ] Does the feature update history in its module folder?
- [ ] Does the change preserve POS speed and operational clarity?
