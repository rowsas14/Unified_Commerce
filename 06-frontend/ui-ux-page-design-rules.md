---
title: UI/UX Page Design Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# UI/UX Page Design Rules

This document defines general page design rules for the Unified Commerce frontend.
It covers POS, admin, e-commerce, fulfillment and reporting pages.

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

## Design principle

Design must match the user's job.
A cashier, tenant admin, inventory staff member, e-commerce customer and manager do not need the same interface style.

## Page categories

| Category | Design priority |
|---|---|
| POS | Speed, clarity, touch operation, visible total. |
| Admin | Configuration accuracy, tables, forms, audit clarity. |
| Inventory | Item/location/quantity accuracy. |
| E-Commerce | Customer browsing and checkout clarity. |
| Fulfillment | Order processing and status action clarity. |
| Reporting | Filters, summaries and traceable totals. |

## POS page design

POS pages must not look like a normal e-commerce website.
They must:

- keep scan/search prominent;
- keep cart visible;
- keep payable total visible;
- use large action buttons;
- minimize navigation depth;
- show session/offline/device status;
- make payment action dominant.

## Admin page design

Admin pages should use:

- clear page title;
- scope indicator: tenant/outlet/channel;
- filter/search area;
- table or form body;
- explicit save/cancel buttons;
- status badges;
- audit-sensitive confirmations.

## E-commerce page design

E-commerce pages should:

- show only online-visible products;
- support product/variant selection;
- keep checkout steps clear;
- show order summary and payment status;
- avoid exposing admin/POS internals.

## Fulfillment page design

Fulfillment pages should prioritize:

- order queue visibility;
- status filters;
- item picking clarity;
- pickup/delivery method display;
- allowed next actions;
- tracking/status history.

## Reporting page design

Reporting pages should:

- place filters at top;
- show summary cards before details;
- show tables for verification;
- avoid unsupported metrics;
- show permission/empty states clearly.

## Page state requirements

Every page must handle:

| State | Required behavior |
|---|---|
| Loading | Show progress/skeleton. |
| Empty | Explain empty result. |
| Error | Show business-readable error. |
| Permission denied | Show restricted state. |
| Feature disabled | Explain unavailable feature. |
| Offline | Show allowed/blocked behavior. |
| Conflict | Show next action or escalation. |

## Color and visual hierarchy

Use visual hierarchy to guide action:

- dominant action for payment/submit;
- warning color for risky/attention states;
- error color for failed/blocking states;
- success color for completed states;
- neutral style for secondary actions.

Do not use too many competing action colors on POS.

## Touch and spacing

POS/touch areas should use:

- comfortable button height;
- clear spacing between critical actions;
- large quantity/payment controls;
- readable font sizes;
- no tiny icon-only critical actions.

Admin screens can be denser but must remain readable.

## Text rules

UI text should be operational and clear.

Good:

```text
Till session is closed. Open a session before billing.
```

Bad:

```text
Invalid session state.
```

## Confirmation rules

Require confirmation for:

- void sale;
- cancel sale/order;
- refund;
- receipt reprint where permission/audit applies;
- stock adjustment posting;
- close till session;
- resolve offline conflict.

## Checklist

- [ ] Page matches user role and job.
- [ ] Main action is visually clear.
- [ ] Loading/empty/error states are present.
- [ ] Permission/feature/offline states are present.
- [ ] POS pages are terminal-like, not website-like.
- [ ] Backend authority is respected.
- [ ] UI text is business-readable.
