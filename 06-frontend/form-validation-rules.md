---
title: Form Validation Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Form Validation Rules

This document defines frontend validation behavior for forms in the Unified Commerce frontend.
Frontend validation improves usability, but backend validation remains the authority.

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

## Validation boundary

| Validation type | Frontend | Backend |
|---|---|---|
| Required UI fields | Yes | Yes |
| Basic format | Yes | Yes |
| Permission/feature access | Reflect result | Final authority |
| Stock availability | Display/preview | Final authority |
| Tax/pricing totals | Preview only | Final authority |
| Payment/refund validity | Basic input guard | Final authority |
| Status transition | Disable obvious invalid action | Final authority |
| Tenant ownership | No final decision | Final authority |

## Form locations

Forms may appear in:

- login and auth screens;
- product/catalog management;
- customer management;
- payment entry;
- discount request/approval;
- return/exchange reason selection;
- stock adjustment/stocktake/transfer;
- tenant settings/theme configuration;
- POS device/printer/scanner setup;
- fulfillment status update.

## Required behavior

Every form must:

- show required fields clearly;
- validate basic input before submit;
- show backend validation errors accurately;
- prevent duplicate submit while pending;
- preserve user input after recoverable validation error;
- clear form only after confirmed success;
- show permission/feature disabled state when relevant.

## Backend error mapping

Frontend must consume validation errors from [[04-api/error-contract]].

| Error | UI behavior |
|---|---|
| Field validation | Attach to field when possible. |
| Business validation | Show form-level message. |
| Permission denied | Show restricted state. |
| Feature disabled | Show feature unavailable state. |
| Conflict | Show conflict resolution instruction. |
| Status invalid | Refresh data and show latest state. |

## POS form rules

POS forms must be minimal.

| POS form | UX rule |
|---|---|
| Payment amount | Numeric keypad-friendly, clear balance/change. |
| Discount entry | Show approval required when threshold exceeded. |
| Return reason | Quick selection, not long free text first. |
| Cash count | Denomination-friendly where implemented. |
| Receipt reprint reason | Required only where backend policy requires. |

## Admin form rules

Admin forms should include:

- section grouping;
- status fields;
- tenant/outlet/channel scope display;
- save/cancel controls;
- audit-sensitive warning where needed;
- backend uniqueness error handling.

## Field naming

Labels should use business language:

| Prefer | Avoid |
|---|---|
| Outlet | LocationId when shown to user |
| Till session | Session object |
| Payment reference | Provider transaction thing |
| Return reason | Reason code id |
| Barcode | Variant identifier |

## Validation examples

| Area | Frontend validation |
|---|---|
| Product SKU | Required before submit if UI asks for SKU. |
| Barcode | Basic non-empty/format when provided. |
| Payment amount | Positive amount and not above remaining balance unless allowed. |
| Refund amount | Positive and user-visible not beyond eligible amount; backend confirms. |
| Coupon code | Non-empty and trimmed before submit. |
| Theme token | Basic readability/format validation. |

## Async validation caution

Use async validation carefully for:

- SKU/barcode uniqueness;
- coupon validity;
- payment reference check;
- customer duplicate lookup;
- device code assignment.

Async validation must not replace submit-time backend validation.

## Offline validation

When offline:

- validate only against cached safe data;
- show stale-data warning where needed;
- queue transaction only when offline mode allows;
- let backend final validation happen during sync;
- show conflict if server rejects later.

## Checklist

- [ ] Required fields are visible.
- [ ] Basic input errors are shown before submit.
- [ ] Backend validation errors are mapped correctly.
- [ ] Duplicate submit is prevented.
- [ ] Form clears only after confirmed success.
- [ ] Offline validation limitations are visible.
- [ ] Backend remains final authority.
