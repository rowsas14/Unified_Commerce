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
# API Error Contract

This document defines the standard API error structure and production error categories.
Errors must be consistent so POS, admin, storefront, offline sync, and AI IDE-generated clients can handle failures predictably.

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


## Standard error response

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

## Error categories

| Category | Use when |
|---|---|
| `AUTHENTICATION_REQUIRED` | Caller is not authenticated. |
| `FORBIDDEN` | Caller lacks permission or role access. |
| `FEATURE_DISABLED` | Feature is not entitled/enabled/allowed. |
| `TENANT_CONTEXT_INVALID` | Tenant context missing or invalid. |
| `OUTLET_CONTEXT_INVALID` | Outlet is invalid or user/device lacks outlet access. |
| `DEVICE_CONTEXT_INVALID` | POS device is inactive, blocked, or mismatched. |
| `SESSION_CONTEXT_INVALID` | Till/session is missing, closed, or invalid. |
| `VALIDATION_ERROR` | Input data is malformed or fails business validation. |
| `INVALID_STATUS_TRANSITION` | Workflow status transition is not allowed. |
| `IDEMPOTENCY_CONFLICT` | Same key/client ID reused with incompatible payload. |
| `CONCURRENCY_CONFLICT` | Resource state changed during operation. |
| `STOCK_CONFLICT` | Stock/reservation state cannot satisfy request. |
| `PAYMENT_FAILED` | Payment provider/manual payment failed. |
| `REFUND_FAILED` | Refund could not be created/completed. |
| `OFFLINE_SYNC_CONFLICT` | Sync item requires explicit conflict handling. |
| `NOT_FOUND` | Resource does not exist or is not visible to caller. |
| `SYSTEM_ERROR` | Unexpected server error. |

## Validation error details

```json
{
  "field": "qty",
  "code": "MUST_BE_GREATER_THAN_ZERO",
  "message": "Quantity must be greater than zero."
}
```

## Business error examples

| Area | Example error |
|---|---|
| Tenant | Tenant is suspended or inactive. |
| Feature access | POS billing feature is not enabled for tenant. |
| POS session | Till session is already closed. |
| Stock | Available stock changed before sale completion. |
| Payment | Captured amount is less than allocation request. |
| Refund | Refund amount exceeds original captured amount. |
| Discount | Discount exceeds approval threshold. |
| Coupon | Coupon usage limit reached. |
| Return | Returned quantity exceeds eligible sold quantity. |
| Order workflow | Delivered order cannot transition back to processing. |
| Offline sync | Client transaction is duplicate or conflicting. |

## Security-safe errors

Do not expose sensitive information in error responses.

| Avoid | Safer response |
|---|---|
| “Customer belongs to tenant B” | “Resource not found or not accessible.” |
| “Password hash mismatch” | “Invalid credentials.” |
| “Feature disabled by platform admin X” | “Feature is not available.” |
| Raw provider exception | “Payment provider error.” |
| SQL constraint details | Business-friendly validation error. |

## POS offline error handling

For offline sync, errors must indicate whether the local item is:

- accepted;
- rejected;
- conflict;
- retryable;
- permanently failed.

This allows the local queue to remain consistent.

## HTTP status guidance

| HTTP status | Use for |
|---|---|
| 400 | Validation or business rule failure. |
| 401 | Not authenticated. |
| 403 | Authenticated but forbidden. |
| 404 | Not found or hidden due to access. |
| 409 | Concurrency/idempotency/status/stock conflict. |
| 422 | Semantically invalid workflow request if used by project convention. |
| 500 | Unexpected server failure. |

## Error contract checklist

- [ ] Error has stable code.
- [ ] Message is user/developer understandable.
- [ ] Details are structured.
- [ ] Sensitive data is not leaked.
- [ ] POS/offline retry behavior is clear where applicable.
- [ ] Logs/audit contain deeper diagnostics where appropriate.
