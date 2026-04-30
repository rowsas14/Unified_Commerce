---
title: Component Design Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Component Design Rules

This document defines reusable React component design rules for the Unified Commerce frontend.
It keeps components practical, module-aware and suitable for POS and admin workflows.

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

## Component purpose

Components should make workflows clear without hiding business meaning.
A POS cart line, payment method selector or stock movement row should be understandable to a developer, QA engineer and product owner.

## Component levels

| Level | Example | Rule |
|---|---|---|
| Base UI | Button, input, badge, modal | No business logic. |
| Feature UI | ProductSearchResult, CartLineRow | Module-specific display and actions. |
| Shell UI | CartShell, PaymentShell | Layout composition for POS regions. |
| Page | POSPage, ReturnPage | Route-level composition only. |

## Props rule

Props must be explicit and business-readable.

Good:

```text
variantId
lineTotal
isPaymentAllowed
onRemoveLine
syncStatus
```

Avoid:

```text
data
item
flag
handleThing
```

## Component state rule

Use local component state for:

- open/closed UI state;
- field focus;
- temporary input before submit;
- local tabs/filters.

Use Zustand for shared workflow state.
Use TanStack Query for server state.

## POS component rules

POS components must:

- support fast tap/click behavior;
- avoid small critical action buttons;
- show immediate feedback;
- avoid hiding totals;
- preserve scan/search focus where possible;
- display offline/pending status where relevant.

## Admin component rules

Admin components should:

- show tables with clear filters;
- show status badges;
- separate create/edit/view actions;
- show audit-sensitive warnings;
- avoid destructive actions without confirmation.

## E-commerce component rules

Customer-facing components should:

- show only e-commerce-visible products;
- handle variant selection;
- validate cart state through API before checkout;
- show order/payment/fulfillment status clearly;
- avoid showing tenant/admin internals.

## Reusable table rules

Tables used for admin/reporting must support:

- loading state;
- empty state;
- error state;
- pagination/filtering where API supports;
- permission-aware actions;
- export only if documented by module/API.

## Modal rules

Use modals carefully in POS.
Allowed modal examples:

- manager approval prompt;
- confirm void/cancel;
- receipt reprint confirmation;
- payment failure retry choice;
- offline conflict details.

Do not use deep modal stacks for scan/add/pay flow.

## Status badge rules

Use status badges for:

- sale status;
- payment status;
- order status;
- fulfillment status;
- refund status;
- sync status;
- device status;
- till session status.

Badge labels must match backend/API status values or approved display mapping.

## Error component rules

Every reusable error component should support:

- title;
- business-readable message;
- optional technical reference/code;
- primary action;
- secondary action;
- permission/feature-disabled state.

## Accessibility and touch

Components must:

- be keyboard reachable where relevant;
- not rely only on color;
- provide clear focus states;
- maintain readable contrast;
- use adequate touch area for POS critical actions.

## Checklist

- [ ] Component has one clear responsibility.
- [ ] Props are explicit and typed.
- [ ] Business status labels match backend/API.
- [ ] Loading/empty/error states are handled.
- [ ] Feature/permission state is handled where applicable.
- [ ] POS components are touch-friendly.
- [ ] Component does not perform backend-only authority logic.
