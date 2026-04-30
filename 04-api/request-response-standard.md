---
title: {title}
owner: API Architecture / Backend Architecture
status: production-ready
last_reviewed: {DATE}
tags: [{tags}]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
---
# Request and Response Standard

This document defines standard request and response behavior for `/api/v1` APIs.
The goal is predictable frontend integration, safe backend validation, and consistent error handling.

## Required reading

- [[00-start-here/README|Start Here]]
- [[01-product/project-scope|Production Project Scope]]
- [[01-product/production-module-catalog|Production Module Catalog]]
- [[02-architecture/system-overview|System Overview]]
- [[02-architecture/tenancy-architecture|Tenancy Architecture]]
- [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
- [[03-data/database-overview|Database Overview]]
- [[03-data/tenant-consistency-rules|Tenant Consistency Rules]]
- [[09-security-and-compliance/authorization-model|Authorization Model]]
- [[09-security-and-compliance/audit-requirements|Audit Requirements]]


## Standard response envelope

Use a consistent envelope unless a binary download, stream, or file response needs a different contract.

```json
{
  "success": true,
  "data": {},
  "message": null,
  "meta": {
    "requestId": "...",
    "timestamp": "2026-04-30T00:00:00Z"
  }
}
```

## Standard error envelope

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": []
  },
  "meta": {
    "requestId": "...",
    "timestamp": "2026-04-30T00:00:00Z"
  }
}
```

See [[04-api/error-contract]].

## Request metadata

| Metadata | Source | Required for |
|---|---|---|
| Request ID | Client/gateway/server | Support traceability. |
| Idempotency key | Client | Payments, orders, sales, refunds, sync submissions. |
| Auth token/session | Client auth state | Staff, platform, customer APIs. |
| Tenant context | Auth/session/platform selection | Tenant-owned operations. |
| Outlet context | Route/query/session where valid | POS, inventory, fulfillment, reporting. |
| Device context | Registered POS terminal | POS sale/session/offline sync. |
| User agent/IP | Server captured | Audit, OTP abuse monitoring, security review. |

## Request body rules

| Rule | Explanation |
|---|---|
| Body carries intent | Backend calculates or validates final business state. |
| IDs are never trusted blindly | Every FK-like ID must be checked for tenant consistency. |
| Amounts are validated server-side | Prevent tampering in sales, payments, refunds, discounts, and tax. |
| Status values are controlled | Status transitions must follow documented workflow rules. |
| JSONB-like payloads are validated | Settings, themes, provider config, receipt payloads, and sync payloads need validation. |

## Data type conventions

| Data type | API format | Notes |
|---|---|---|
| UUID | string | Canonical UUID string. |
| Date/time | ISO-8601 string | Use UTC or explicit offset. |
| Business date | `YYYY-MM-DD` | Used for till sessions and summaries. |
| Currency | ISO 4217 code | Example: `LKR`, `USD`. |
| Money | decimal-compatible value | Backend stores numeric precision. |
| Quantity | decimal-compatible value | Inventory supports decimal quantities. |
| Status | documented string | Must match database CHECK/status values. |
| Boolean | true/false | Do not use `0/1` strings. |

## Success response rules

| API type | Response guidance |
|---|---|
| Create | Return created resource summary and server ID. |
| Update | Return updated resource summary or status result. |
| Workflow action | Return resulting state and relevant side-effect references. |
| List | Return `items` and pagination metadata. |
| Report | Return report rows plus filter/date/outlet/channel metadata. |
| Sync | Return batch/item acceptance, rejection, and conflict summaries. |

## Validation response detail

Validation errors should identify fields and business rule failures.

```json
{
  "field": "variantId",
  "code": "INVALID_TENANT_REFERENCE",
  "message": "Variant does not belong to the current tenant."
}
```

## Response shaping by caller

| Caller | Response rule |
|---|---|
| Cashier | Operational fields only; no hidden admin/security details. |
| Manager/admin | Include management fields permitted by role. |
| Customer | Include only customer-facing and own tenant-scoped data. |
| Platform admin | Include platform support fields only in platform routes. |
| POS offline sync | Return deterministic sync status and conflict details needed for local queue update. |

## Sensitive field rules

Do not return by default:

- password hashes;
- OTP code hashes;
- payment secrets;
- provider private config values;
- internal raw provider payloads to normal users;
- complete audit snapshots to unauthorized users;
- secret references except in restricted configuration views;
- cross-tenant IDs or internal support data.

## List response shape

List APIs should use:

```json
{
  "success": true,
  "data": {
    "items": [],
    "page": 1,
    "pageSize": 25,
    "totalCount": 0
  },
  "meta": {}
}
```

See [[04-api/pagination-filtering-sorting]].

## Request/response checklist

- [ ] Uses the standard envelope.
- [ ] Includes request ID metadata.
- [ ] Avoids sensitive field leakage.
- [ ] Validates tenant-owned IDs.
- [ ] Validates computed totals server-side.
- [ ] Uses documented status values.
- [ ] Uses clear validation error details.
- [ ] Matches frontend API client expectations.
