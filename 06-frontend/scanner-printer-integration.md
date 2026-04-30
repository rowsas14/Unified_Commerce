---
title: Scanner and Printer Integration
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Scanner and Printer Integration

This document defines frontend rules for barcode scanners, receipt printers and cash drawer bridge behavior.
It aligns with the uploaded frontend architecture and the production scope for POS device, terminal and hardware management.

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

## Source architecture

The uploaded frontend architecture defines:

```text
core/peripherals/
├── printerBridge.ts
├── scannerListener.ts
└── cashDrawer.ts
```

These files provide browser/frontend integration helpers only.
Device registration and assignment remain backend/data responsibilities.

## Hardware responsibility boundary

| Concern | Frontend | Backend/database |
|---|---|---|
| Scanner input listening | Yes | No |
| Printer command bridge | Yes | No direct UI rule |
| Default printer assignment | Display/use assigned config | Source of truth |
| Device/till/outlet assignment | Display and send context | Source of truth |
| Print log | Trigger/display result | Store receipt print log |
| Cash drawer kick | Trigger bridge if configured | Authorize and audit where required |

## Scanner rules

Most barcode scanners behave like keyboard input.
The frontend should support this as the default.

Scanner listener must:

- direct scanner input into the scan/search field;
- avoid breaking normal keyboard input;
- handle Enter/suffix behavior;
- prevent duplicate add from repeated scanner events where possible;
- show quick feedback for found/not found products;
- work with offline cached product lookup where allowed.

## Scanner anti-patterns

Do not:

- require cashier to click into search after every scan;
- treat scanner input as normal product browsing only;
- open large modal for every scanned product;
- block manual search if scanner is unavailable;
- create scanner-specific product rules not backed by catalog/API.

## Printer bridge rules

`printerBridge.ts` should handle frontend printing operations such as:

- test print;
- receipt print;
- receipt reprint request display;
- print status feedback;
- printer failure message.

The final receipt payload must come from backend-confirmed receipt data or approved local offline receipt payload.

## Receipt print states

| State | UI behavior |
|---|---|
| Ready | Print button enabled. |
| Printing | Disable duplicate print action and show progress. |
| Printed | Show success and timestamp if available. |
| Failed | Show retry/reprint option based on permission. |
| Reprint | Mark duplicate/reprint state where applicable. |
| Offline pending | Print local receipt with pending-sync marker if allowed. |

## Printer failure rule

Printer failure must not corrupt a completed sale.
If sale/payment completed but print failed, UI must show:

- sale completed;
- receipt print failed;
- retry/reprint option if allowed;
- print log/error message if returned by backend.

## Cash drawer rules

`cashDrawer.ts` may trigger cash drawer behavior when configured.
Cash drawer action must be tied to allowed events such as:

- cash sale completion;
- cash refund where allowed;
- manager-approved drawer open/cash movement;
- till open/close workflow if supported.

Do not allow arbitrary drawer open without permission/audit flow if backend rules require control.

## Device and terminal status

Frontend should show hardware readiness where useful:

| Hardware item | UI display |
|---|---|
| POS device | Registered/active/blocked state if available. |
| Till | Active/inactive/maintenance. |
| Printer | Assigned/available/error. |
| Scanner | Input active/test result. |
| Cash drawer | Configured/unavailable where applicable. |

## Test mode

Admin/device setup screens should support test operations where scope allows:

- scanner test input;
- printer test print;
- cash drawer test where configured;
- status display.

These tests must not create sales, payments or receipts.

## Offline behavior

Offline POS may still:

- accept scanner input against cached products;
- print local pending receipt if allowed;
- queue receipt payload for backend sync;
- show printer failure independent of network state.

Offline print does not mean backend has accepted the sale.

## Checklist

- [ ] Scanner input targets scan/search field.
- [ ] Manual search remains available.
- [ ] Printer failure does not reverse sale completion.
- [ ] Reprint is permission-aware.
- [ ] Hardware status is clear.
- [ ] Offline receipts are marked as pending sync.
- [ ] Device/till/outlet assignment comes from backend data.
