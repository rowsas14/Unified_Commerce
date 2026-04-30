---
title: API Client and TanStack Query Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# API Client and TanStack Query Rules

This document defines how frontend code communicates with the backend API.
It aligns with the `/api/v1` API documentation and the Clean Architecture backend rules.

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

## API base rule

All production API calls must use the versioned API base:

```text
/api/v1
```

The API versioning rule is documented in [[04-api/api-overview]].

## API client ownership

| File | Responsibility |
|---|---|
| `core/api/http.ts` | HTTP client, base URL, auth header, tenant/context headers, error normalization. |
| `core/api/endpoints.ts` | Endpoint builders/constants. |
| `core/api/queryClient.ts` | TanStack Query client configuration. |
| `features/*/api/` | Feature-specific queries and mutations. |

Do not create direct API clients inside page components.

## Request context

Frontend API calls must preserve the context needed by backend validation.
The exact transport may be headers, route parameters or request body depending on API rules, but the frontend must never lose:

| Context | Used for |
|---|---|
| Tenant context | Tenant data isolation. |
| Outlet context | POS stock, sales, till and fulfillment operations. |
| Device context | POS device and offline sync tracking. |
| Till session context | Cash drawer, billing and payment flow. |
| User/session context | Role, permission and audit. |
| Idempotency key | Payments, sales, orders and sync operations where duplicate requests are possible. |

Related docs:

- [[04-api/tenant-context-api-rules]]
- [[04-api/device-session-api-rules]]
- [[04-api/idempotency-rules]]

## Query vs mutation

| Operation type | TanStack Query usage |
|---|---|
| Read products, customers, reports | Query |
| Create sale, payment, return, order | Mutation |
| Update status, approve discount, close session | Mutation |
| Sync offline queue | Mutation or controlled sync service call |
| Fetch lookup/reference data | Query with suitable stale time |

## Server state rule

Server state must live in TanStack Query.
Do not copy server lists into Zustand unless they become part of an active local workflow.

Example:

| Data | Correct owner |
|---|---|
| Product search results | TanStack Query |
| Active cart lines | Zustand |
| Payment methods list | TanStack Query |
| Current payment entry before submit | Zustand/local component state |
| Report result | TanStack Query |
| Offline queue item status | Offline store + IndexedDB + sync API response |

## Mutation safety

Mutations that can create financial or stock records must use careful client behavior:

- disable duplicate submit buttons while pending;
- use idempotency keys where API rules require;
- show success only after backend confirmation;
- show clear failure states;
- do not silently retry payment capture unless API rules allow it;
- do not overwrite conflict responses with optimistic success.

## Optimistic update rule

Optimistic updates are allowed only for low-risk UI feedback.
They are not allowed as final truth for:

- completed POS sale;
- payment capture;
- refund completion;
- stock movement posting;
- offline sync acceptance;
- receipt final issue;
- order status transition.

## Error normalization

`http.ts` should normalize backend errors into a predictable UI format based on [[04-api/error-contract]].

| Error category | UI behavior |
|---|---|
| Validation error | Show field or form-level messages. |
| Unauthorized | Clear or refresh session and redirect when required. |
| Forbidden | Show permission-denied state. |
| Feature disabled | Hide or disable related action with explanation. |
| Conflict | Show conflict state and next action. |
| Offline/network | Queue only where offline rules allow. |
| Payment failed | Keep transaction visible and safe for retry/manager action. |

## Query key design

Query keys must include the context that changes the result.

```text
['products', tenantId, outletId, channel, searchTerm]
['inventory-balance', tenantId, outletId, variantId]
['orders', tenantId, filters]
['reports', tenantId, outletId, channel, dateRange]
```

Do not use global query keys for tenant-specific data.

## Cache invalidation rules

| Mutation | Invalidate/refetch |
|---|---|
| Product update | Product detail, product lists, POS product cache if applicable. |
| Sale completion | Sales, inventory balance, reporting summaries, receipts. |
| Payment capture | Payment, sale/order payment status, reports. |
| Return/refund | Sale/order, return, payment/refund, inventory, report summaries. |
| Stock adjustment | Inventory balance, stock movement list, reports. |
| Feature setting update | Feature/config queries and route availability. |

## Offline handling

When offline:

- Do not call online-only APIs.
- Use cached POS data only where offline POS rules allow.
- Queue offline sale/payment/receipt payloads through `core/offline/syncQueue.ts`.
- Show pending sync status.
- Submit queued records to offline sync API after reconnection.

See [[06-frontend/offline-frontend-rules]].

## API client checklist

- [ ] API path uses `/api/v1`.
- [ ] Request includes required tenant/outlet/session/device context.
- [ ] Query key includes tenant and relevant filters.
- [ ] Mutations use idempotency where required.
- [ ] Error contract is mapped to UI state.
- [ ] Cache invalidation is explicit.
- [ ] Offline behavior is documented for the feature.
- [ ] No raw API call exists inside page JSX.
