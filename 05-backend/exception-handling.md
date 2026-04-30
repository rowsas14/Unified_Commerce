---
title: Exception Handling
folder: 05-backend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
stack: .NET Web API, Clean Architecture, PostgreSQL, EF Core
patterns: Service Pattern, Repository Pattern, Unit of Work
cqrs: not-used
---

# Exception Handling

The backend must return controlled errors that match the API error contract and help frontend, POS, e-commerce, QA, and support teams understand failures safely.
Exceptions must not leak secrets, SQL details, stack traces, provider credentials, or cross-tenant information.

## Exception ownership

| Layer | Exception responsibility |
|---|---|
| API | Convert exceptions to HTTP responses through middleware. |
| Application | Throw or return meaningful application/domain errors. |
| Domain | Signal pure business rule violations. |
| Infrastructure | Wrap database/provider errors into safe application exceptions. |

## Middleware rule

Use a global exception middleware in `POS.Api/Middlewares/`.
The middleware should map known exceptions to the API error contract documented in [[04-api/error-contract]].

## Error categories

| Category | Example cause | Typical response |
|---|---|---|
| Validation | Missing field, invalid quantity, wrong status | `400 Bad Request` |
| Authentication | Missing/invalid login token | `401 Unauthorized` |
| Authorization | User lacks permission | `403 Forbidden` |
| Feature disabled | Tenant/role/flag does not allow feature | `403 Forbidden` or documented feature error |
| Not found | Tenant-scoped record not found | `404 Not Found` |
| Conflict | Duplicate idempotency key, stock conflict, invalid status race | `409 Conflict` |
| Business rule | Refund exceeds captured amount, return exceeds sold quantity | `422 Unprocessable Entity` if used by API contract |
| Provider failure | Payment provider failure | Controlled payment error response |
| Server error | Unexpected failure | `500` with safe generic message |

## Production-specific exception examples

| Exception type | When used |
|---|---|
| `TenantAccessDeniedException` | User tries to access another tenant's data. |
| `FeatureNotEnabledException` | Tenant or role does not have feature access. |
| `PermissionDeniedException` | User lacks required permission. |
| `InvalidStatusTransitionException` | Order/payment/fulfillment transition is not allowed. |
| `StockConflictException` | Inventory cannot satisfy a transaction or offline sync conflict exists. |
| `PaymentValidationException` | Payment amount/status/allocation is invalid. |
| `RefundLimitExceededException` | Refund would exceed original captured amount. |
| `OfflineSyncConflictException` | Offline item cannot be accepted cleanly. |
| `TillSessionRequiredException` | POS billing attempted without required open session. |

These names are documentation examples, not a requirement to use exact class names.
If implemented, keep names consistent with [[05-backend/naming-conventions]].

## Safe error response rule

Do not return:

- SQL query text;
- connection strings;
- provider secret references beyond safe identifiers;
- raw OTP values;
- password hash details;
- stack traces in production;
- data from a different tenant;
- raw payment provider payload unless intentionally exposed through a secure support endpoint.

## Error response shape

The exact API error shape is owned by [[04-api/error-contract]].
Backend exception middleware must map exceptions into that shape.

Expected fields usually include:

| Field | Purpose |
|---|---|
| `code` | Stable machine-readable error code. |
| `message` | Safe human-readable message. |
| `details` | Optional validation details. |
| `traceId` | Support/debug correlation id. |

## Logging rule

| Error type | Log level |
|---|---|
| Validation failure | Information or warning depending on frequency. |
| Permission denied | Warning for sensitive actions. |
| Tenant isolation violation attempt | Warning/security log. |
| Payment provider failure | Warning/error depending on outcome. |
| Offline sync conflict | Information/warning and sync audit log. |
| Unexpected exception | Error. |

## Audit vs log distinction

| Record type | Purpose |
|---|---|
| Application log | Technical diagnosis. |
| `audit_logs` | Business/sensitive action trace. |
| `offline_sync_audit_logs` | Offline sync lifecycle diagnostics. |
| `payment_transactions.raw_payload` | Provider event trace where applicable. |

A failed sensitive business action may need both log and audit depending on the workflow.

## Exception handling flow

```text
Controller calls Application service
  -> service validates and executes workflow
  -> known business failure occurs
  -> service throws/returns known exception/result
  -> exception middleware maps to API error contract
  -> log/audit/sync audit is written where required
  -> frontend receives safe error
```

## Checklist

- [ ] Known business failures have stable error codes.
- [ ] Middleware maps exceptions to documented API response.
- [ ] Production response does not leak stack traces.
- [ ] Tenant isolation failures do not reveal cross-tenant existence.
- [ ] Payment/provider failures are sanitized.
- [ ] Offline sync failures create conflict/audit records where required.
- [ ] Sensitive action failures are logged/audited according to rules.
