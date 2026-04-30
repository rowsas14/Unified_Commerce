---
title: Frontend Naming Conventions
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Frontend Naming Conventions

This document defines naming rules for frontend files, folders, components, hooks, stores and types.
Naming must align with the production Unified Commerce modules and uploaded frontend architecture.

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

## Folder naming

Use lowercase kebab-case for feature folders where already defined:

```text
features/till-session
features/products
features/payments
features/returns
features/inventory
```

Do not create duplicate folder names for the same concept.

## Component naming

Use PascalCase for React components:

```text
ProductSearchResult.tsx
CartLineRow.tsx
PaymentMethodSelector.tsx
TillSessionStatusBadge.tsx
OfflineSyncBanner.tsx
```

Component names must describe business purpose, not visual shape only.

## Hook naming

Use `use` prefix and business meaning:

```text
useProductSearch
useCartActions
usePaymentMethodsQuery
useCreateSaleMutation
useOfflineSyncStatus
useFeatureAccess
```

TanStack Query hooks should make query/mutation intent visible.

## Store naming

Store files follow uploaded architecture:

```text
app.store.ts
session.store.ts
till.store.ts
cart.store.ts
ui.store.ts
offline.store.ts
```

Store actions should use business verbs:

```text
addCartLine
removeCartLine
openTillSession
markSyncConflict
clearCartAfterSaleCompletion
```

## API file naming

Feature API files should be clear:

```text
products.api.ts
payments.api.ts
orders.api.ts
returns.api.ts
offline-sync.api.ts
```

Avoid vague names such as `service.ts` unless the folder context is very clear.

## Type naming

Use explicit frontend/API type names:

| Type suffix | Use |
|---|---|
| `Request` | Payload sent to API. |
| `Response` | Payload returned by API. |
| `Dto` | Data transfer shape when matching backend DTO. |
| `ViewModel` | UI-specific display shape. |
| `FormValues` | Form input state. |
| `Status` | Union/status values. |

Example:

```text
CreateSaleRequest
PaymentMethodResponse
CartLineViewModel
TillOpenFormValues
OfflineSyncStatus
```

## Status naming

Status labels must match backend/API documented values or map explicitly.

Do not invent frontend-only statuses that conflict with backend values.

Example mappings should be documented when display labels differ:

| Backend value | Display label |
|---|---|
| `pending_sync` | Pending Sync |
| `partially_refunded` | Partially Refunded |
| `ready_for_pickup` | Ready for Pickup |

## Event handler naming

Use business event names:

```text
handleAddScannedProduct
handleCompletePayment
handleOpenTill
handleSubmitReturn
handleRetryReceiptPrint
```

Avoid ambiguous names:

```text
handleClick
handleSubmit
processData
```

## CSS/Tailwind class naming

Tailwind does not require CSS class names for most UI.
If custom classes are used, name by purpose:

```text
pos-total-panel
receipt-preview-area
offline-sync-banner
```

Avoid purely decorative names.

## Route naming

Routes should be business-readable:

```text
/pos
/pos/payment
/pos/returns
/till/open
/till/close
/inventory/stocktake
/reports/daily-sales
```

Route paths must match documented modules and guards.

## Test naming

Test names should describe behavior:

```text
POSPage.blocksPaymentWhenTillSessionClosed
CartStore.clearsAfterSaleCompletion
OfflineSyncBanner.showsConflictCount
```

## Naming checklist

- [ ] Name matches business meaning.
- [ ] Name matches documented module language.
- [ ] No duplicate concept names exist.
- [ ] Status values align with API/backend docs.
- [ ] Components use PascalCase.
- [ ] Hooks use `use` prefix.
- [ ] Store files follow approved names.
- [ ] Route names match module/user flow.
