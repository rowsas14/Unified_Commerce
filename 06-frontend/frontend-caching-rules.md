---
title: Frontend Caching Rules
owner: Frontend Lead
status: production-ready
last_reviewed: 2026-04-30
tags: [frontend, caching, offline, pos]
source: Unified Commerce production scope + database design + frontend architecture
---

# Frontend Caching Rules

## Purpose

This file defines frontend caching rules for the React + TypeScript Unified Commerce frontend.

Frontend caching exists for user experience, network efficiency, POS speed and offline support.

It is not security, financial truth, payment truth or stock truth.

Read with:

- [[06-frontend/api-client-and-query-rules]]
- [[06-frontend/offline-frontend-rules]]
- [[06-frontend/state-management-rules]]
- [[05-backend/caching-strategy]]
- [[02-architecture/offline-first-architecture]]

---

## Core rules

- Cache only safe data.
- Respect tenant and outlet in query keys.
- Do not reuse cached data across tenant switches.
- Do not treat frontend cache as backend validation.
- Invalidate after mutations.
- Refresh server state after payment, sale, refund, return, exchange and sync actions.

---

## Backend cache boundary

Frontend cache and backend cache are separate.

| Layer | Purpose | Final authority? |
|---|---|---:|
| TanStack Query cache | Server-state UX caching. | No |
| Zustand store | Local workflow state. | No |
| IndexedDB offline store | Offline POS continuity. | No |
| Backend application cache | Safe backend read optimization. | No |
| PostgreSQL source tables | Business transaction truth. | Yes |

A successful frontend mutation must rely on backend response, not old local cache.

---

## Query key rules

Every tenant-owned query key must include tenant context.

Example shapes:

```text
['tenant', tenantId, 'catalog', 'barcode', barcode]
['tenant', tenantId, 'outlet', outletId, 'pos-products']
['tenant', tenantId, 'settings', scope]
['tenant', tenantId, 'feature-access', userId]
['tenant', tenantId, 'inventory', outletId, variantId]
```

Never use global keys for tenant-owned data:

```text
['products']          // wrong
['settings']          // wrong
['permissions']       // wrong
```

---

## Safe frontend cache data

| Data | Cache allowed | Notes |
|---|---:|---|
| Tenant settings | Yes | Must clear on tenant switch. |
| Feature flags | Yes | Must refresh after config change. |
| UI themes | Yes | Safe for display. |
| Product browse data | Yes | Must refresh after catalog changes. |
| Barcode/SKU lookup data | Yes | Useful for POS speed. |
| Category/brand lists | Yes | Safe master data. |
| Payment method list | Yes | Do not cache secrets. |
| Delivery methods/zones | Yes | Refresh after checkout-related changes. |
| Product/pricing/tax bootstrap for offline POS | Yes | Must sync/refresh when online. |

---

## Unsafe frontend cache data

Do not trust cached values for final decisions:

| Data | Required behavior |
|---|---|
| Stock availability | Backend revalidates before sale/order/sync acceptance. |
| Payment status | Backend returns final status. |
| Refund eligibility | Backend validates original payment and return rules. |
| Discount approval | Backend validates status and permission. |
| Coupon usage limit | Backend validates transactionally. |
| Till session state | Backend validates before sale completion. |
| Offline sync status | Backend returns accepted/rejected/conflict. |
| OTP state | Backend validates only. |
| Permissions for sensitive actions | Backend is final authority. |

---

## POS offline cache

POS offline cache may store:

- Product lookup data.
- Variant SKU/barcode data.
- Price data.
- Tax data.
- Safe discount rule data.
- Current cart state.
- Offline sale/payment/receipt payloads.
- Last sync metadata.

But server must validate after reconnect.

Offline cache is not accepted business truth.

---

## Mutation invalidation rules

After successful mutations, invalidate related queries.

| Mutation | Invalidate |
|---|---|
| Product update | Product list/search/detail/barcode caches. |
| Price update | Price and POS bootstrap caches. |
| Tax update | Tax and checkout calculation caches. |
| Feature flag update | Feature access/config caches. |
| Payment completion | Sale/order/payment queries. |
| Return/exchange completion | Sale/order/stock/payment/receipt queries. |
| Offline sync result | Sync queue, conflict, sales, payments, stock queries. |
| Session open/close | Till/session/POS route guard queries. |

---

## Zustand boundary

Zustand may hold workflow state such as:

- Active cart.
- POS UI state.
- Till/session client state.
- Payment screen state.
- Offline indicator state.

Do not use Zustand as server-state source for:

- Permissions.
- Feature access.
- Final stock.
- Final payment status.
- Final order status.
- Audit state.

---

## QA checklist

- [ ] Tenant switch clears tenant-owned cache.
- [ ] Outlet switch clears outlet-specific POS data.
- [ ] POS cache key includes tenant/outlet/channel where needed.
- [ ] Payment mutation refreshes payment/sale/order state.
- [ ] Offline sync response updates local queue state.
- [ ] Stale stock display does not allow final checkout without backend validation.
- [ ] Feature access hidden in UI is still backend-protected.
- [ ] Cached product data is refreshed after catalog update.

---

## Related files

- [[06-frontend/api-client-and-query-rules]]
- [[06-frontend/offline-frontend-rules]]
- [[06-frontend/state-management-rules]]
- [[05-backend/caching-strategy]]
