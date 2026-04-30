---
title: Frontend Overview
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Frontend Overview

The frontend is the operational interface for the Unified Commerce platform.
It supports POS counter workflows, admin configuration, inventory operations, e-commerce storefront behavior, fulfillment operations, reporting and security-aware user access.

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

## Frontend purpose

The frontend must turn the production scope into usable screens without weakening backend controls.
It must be fast for cashiers, structured for admins, safe for tenant isolation and predictable for AI-assisted development.

| Frontend area | Primary users | Main responsibility |
|---|---|---|
| POS terminal | Cashier, outlet manager | Scan, add, pay, print, return, exchange, hold, recall and close till. |
| Admin console | Tenant admin, platform admin | Configure tenants, outlets, roles, features, themes, catalog, pricing and tax. |
| Inventory screens | Inventory staff, manager | Stock view, stocktake, transfer, adjustment and receiving. |
| E-Commerce storefront | Customer | Browse products, cart, checkout and order tracking. |
| Fulfillment screens | E-commerce operator | Prepare pickup/delivery orders and update fulfillment status. |
| Reporting dashboards | Manager, admin | View operational reports and audit-sensitive summaries. |

## Source architecture alignment

The uploaded frontend architecture defines this structure:

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

This folder documents how that structure must be used for the production Unified Commerce system.

## Frontend responsibility boundary

| Concern | Frontend responsibility | Backend responsibility |
|---|---|---|
| Authentication | Capture credentials, store session safely, route user | Validate identity, issue/validate tokens, account status |
| Authorization | Hide/disable unavailable actions, guard routes | Enforce permission and feature access |
| Tenant context | Load selected tenant/outlet/session/device context | Validate tenant ownership and isolation |
| Pricing/tax | Preview current calculation where safe | Confirm final totals |
| Stock | Display available stock and warnings | Validate stock, reservations and movements |
| Payment | Capture selected method and reference | Validate payment, allocation and status |
| Offline POS | Queue local actions and show status | Accept/reject/conflict sync items |
| Reports | Show filtered summaries | Produce trusted report data |

## Frontend application layers

```mermaid
flowchart TD
  Pages[pages/] --> Shells[shells/]
  Shells --> Features[features/]
  Features --> Core[core/api, auth, offline, peripherals]
  Features --> State[state/]
  Core --> API[/api/v1 Backend]
  State --> UI[Workflow UI]
  Shared[shared-kernel/] --> Features
```

## Production channels

| Channel | Frontend behavior |
|---|---|
| POS | Touchscreen-first, scan-first, low typing, session-aware, offline-aware. |
| E-Commerce | Customer-facing browsing, cart, checkout and order tracking. |
| Admin | Data management, configuration, reporting and approval workflows. |
| Fulfillment | Operational order processing, pickup and delivery status management. |

## Frontend must optimize for POS speed

The POS screen is not a normal website product listing.
It must prioritize:

- always-focused scan/search input;
- visible cart and payable total;
- large touch targets;
- fast quantity changes;
- clear offline state;
- quick payment entry;
- receipt print/reprint visibility;
- safe hold/recall behavior;
- locked state when no till session is open.

## Backend authority rule

Frontend shared-kernel helpers such as `PriceResolver`, `DiscountEngine`, `TaxCalculator`, `ReceiptBuilder` and `InvoiceBuilder` are UI-support helpers only.
They must not become the final business authority.

Final authority belongs to backend services described in:

- [[05-backend/service-layer-rules]]
- [[05-backend/transaction-boundary-rules]]
- [[05-backend/payment-gateway-integration-rules]]
- [[05-backend/offline-sync-backend-rules]]

## Frontend module ownership

| Frontend module | Related product module |
|---|---|
| `features/auth` | Identity and access |
| `features/till-session` | Cash drawer, shift and session |
| `features/products` | Catalog and pricing display |
| `features/cart` | POS cart and e-commerce cart workflow |
| `features/sales` | POS checkout |
| `features/payments` | Payments and refunds |
| `features/customers` | Customer management |
| `features/discounts` | Discounts, coupons and approval |
| `features/returns` | Returns and exchanges |
| `features/inventory` | Inventory, stocktake and transfer |
| `features/receipts` | Receipt generation and print UI |
| `features/config` | Tenant settings, theme and feature flags |

## Frontend quality expectations

- The UI must show clear user state: tenant, outlet, device, till session, cashier and connectivity.
- Every destructive/sensitive action must show clear confirmation and permission result.
- Backend validation errors must be displayed in business language, not raw technical stack traces.
- Offline pending transactions must be visible and not hidden from operators.
- Every route must belong to a documented module and user role.

## Related frontend documents

- [[06-frontend/frontend-folder-structure]]
- [[06-frontend/react-architecture-rules]]
- [[06-frontend/api-client-and-query-rules]]
- [[06-frontend/state-management-rules]]
- [[06-frontend/pos-ui-rules]]
- [[06-frontend/offline-frontend-rules]]
