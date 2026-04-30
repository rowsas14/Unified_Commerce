---
title: API Spec Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, api, unified-commerce]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend: Back end Architecture final.txt
---

# API Spec Template

Use this template for API endpoint documentation.
API specs must align with tenant context, RBAC, feature access, idempotency, validation and database ownership.

## File location examples

```text
04-api/payment-refund-api-rules.md
07-modules/payments/features/payments/api-spec.md
07-modules/offline-sync/features/sync-batches/api-spec.md
```

## Copy template

```markdown
---
title: <Endpoint or API Group> API Spec
owner: Backend Team
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce, api, <module>]
module: <module>
feature: <feature>
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend: Back end Architecture final.txt
related_docs:
  - [[04-api/api-overview]]
  - [[04-api/request-response-standard]]
  - [[04-api/tenant-context-api-rules]]
---

# <Endpoint or API Group> API Spec

## 1. Purpose

<Explain what this endpoint/API group does.>

## 2. Endpoint summary

| Field | Value |
|---|---|
| Method | GET / POST / PUT / PATCH / DELETE |
| Path | `/api/...` |
| Module | `<module>` |
| Feature | `<feature>` |
| Channel | POS / E-Commerce / Admin / Platform |
| Auth required | Yes/No |
| Tenant context required | Yes/No |
| Outlet context required | Yes/No |
| Device/session context required | Yes/No |
| Idempotency required | Yes/No |

## 3. Access control

| Control | Rule |
|---|---|
| Tenant entitlement | <required feature key or not applicable> |
| Feature flag | <required runtime flag or not applicable> |
| Role feature assignment | <required or not applicable> |
| Permission | `<permission.code>` |
| Actor | <actor> |

## 4. Request headers

| Header | Required | Purpose |
|---|---:|---|
| `Authorization` | Yes | Bearer token. |
| `X-Tenant-Id` | Conditional | Tenant context where not inferred from token. |
| `X-Outlet-Id` | Conditional | Outlet context for POS/outlet actions. |
| `X-Device-Id` | Conditional | POS device context. |
| `Idempotency-Key` | Conditional | Required for retry-prone write operations. |

## 5. Request body

```json
{
  "exampleField": "value"
}
```

## 6. Request validation

| Field | Rule | Error code |
|---|---|---|
| `exampleField` | Required | `VALIDATION_ERROR` |

## 7. Response body

```json
{
  "success": true,
  "data": {},
  "errors": []
}
```

## 8. Response codes

| Status | Meaning |
|---|---|
| 200 | Successful read/update. |
| 201 | Resource created. |
| 400 | Validation or business rule failure. |
| 401 | Not authenticated. |
| 403 | Permission, feature access or tenant access denied. |
| 404 | Resource not found in allowed tenant scope. |
| 409 | Conflict, duplicate, invalid status transition or idempotency conflict. |
| 422 | Business validation failure. |
| 500 | Unexpected server error. |

## 9. Business rules

1. <Rule 1>
2. <Rule 2>
3. <Rule 3>

## 10. Database impact

| Table/entity | Operation | Notes |
|---|---|---|
| `<table_name>` | Create/Read/Update/Delete | <notes> |

## 11. Transaction boundary

<Explain what must commit atomically.>

## 12. Idempotency behavior

| Scenario | Expected behavior |
|---|---|
| First request | <creates/processes> |
| Retry with same key | <returns same result or safe duplicate response> |
| Same key with different body | <reject conflict> |

## 13. Offline behavior

| State | Behavior |
|---|---|
| Online | <normal endpoint call> |
| Offline | <blocked or queued locally> |
| Sync | <sync endpoint behavior> |

## 14. Audit behavior

| Action | Audit event |
|---|---|
| <action> | <audit event> |

## 15. Error examples

```json
{
  "success": false,
  "errors": [
    {
      "code": "FEATURE_DISABLED",
      "message": "Feature is not enabled for this tenant."
    }
  ]
}
```

## 16. Test cases

| Test | Expected result |
|---|---|
| <scenario> | <result> |

## 17. Related docs

- [[04-api/api-overview]]
- [[04-api/error-contract]]
- [[04-api/idempotency-rules]]
- [[05-backend/backend-overview]]
```

## API design rules

APIs must not bypass production controls.

Every protected endpoint must check:

```text
Authentication -> Tenant context -> Feature entitlement -> Feature flag -> Role feature assignment -> Permission -> Business validation
```

## Tenant context rules

| Endpoint type | Tenant source |
|---|---|
| Platform admin tenant management | Path/body target tenant controlled by platform permission. |
| Tenant admin operations | Authenticated user tenant context. |
| POS operations | Tenant + outlet + device + till/session context. |
| E-Commerce customer operations | Tenant from storefront/channel + customer account/guest token. |
| Offline sync | Tenant + outlet + device + sync batch. |

## Idempotency required for

- Payment creation/capture.
- Refund creation.
- POS sale completion.
- E-Commerce order placement.
- Offline sync item acceptance.
- Receipt generation where retry can duplicate records.
- Coupon redemption where retry can double count usage.

## Backend mapping

The backend architecture uses feature-based API grouping and Clean Architecture layers.
Document where endpoint implementation belongs:

| Layer | Expected location |
|---|---|
| API | `POS.Api/Modules/<Module>/Controllers` |
| Requests | `POS.Api/Modules/<Module>/Requests` |
| Responses | `POS.Api/Modules/<Module>/Responses` |
| Application service | `POS.Application/Modules/<Module>` |
| Domain logic | `POS.Domain/Modules/<Module>` |
| Repository | `POS.Infrastructure/Repositories/<Module>` |

## Error code examples

Use stable machine-readable error codes.

| Code | Meaning |
|---|---|
| `TENANT_NOT_ALLOWED` | Actor cannot access tenant. |
| `FEATURE_DISABLED` | Feature is not enabled or configured. |
| `PERMISSION_DENIED` | Actor lacks required permission. |
| `OUTLET_CONTEXT_REQUIRED` | Outlet context missing for outlet action. |
| `INVALID_STATUS_TRANSITION` | Requested status change is not allowed. |
| `INSUFFICIENT_STOCK` | Stock is not available. |
| `PAYMENT_AMOUNT_INVALID` | Payment amount violates rules. |
| `REFUND_EXCEEDS_CAPTURED_AMOUNT` | Refund exceeds allowed amount. |
| `OFFLINE_SYNC_CONFLICT` | Offline item cannot be accepted cleanly. |
| `DUPLICATE_CLIENT_TRANSACTION` | Offline transaction already accepted. |

## API checklist

- [ ] Endpoint path uses module ownership.
- [ ] Tenant context is defined.
- [ ] Permission is defined.
- [ ] Feature access is defined.
- [ ] Request fields are documented.
- [ ] Response body is documented.
- [ ] Validation and business errors are documented.
- [ ] Database tables are listed.
- [ ] Transaction boundary is clear.
- [ ] Idempotency behavior is defined where needed.
- [ ] Offline behavior is defined where relevant.
- [ ] Audit events are listed.
- [ ] Test cases are listed.
