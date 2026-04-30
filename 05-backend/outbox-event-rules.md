---
title: Outbox Event Rules
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

# Outbox Event Rules

This file exists because the current 2nd Brain structure contains backend event documentation.
However, the uploaded source documents do not define an outbox table, event bus, message broker, or outbox implementation.

## Current decision

Do not implement an outbox pattern from this file alone.
The current approved database design includes audit, offline sync audit, payment transaction logs, and reporting read models, but it does not include an outbox table.

## What is allowed now

| Existing mechanism | Use |
|---|---|
| `audit_logs` | Business audit trail for sensitive actions. |
| `offline_sync_audit_logs` | Technical audit trail for sync lifecycle. |
| `payment_transactions` | Provider/gateway event trace per payment. |
| Reporting read models | Derived operational summaries. |
| Status history tables | Order/payment/fulfillment status trace where applicable. |

## What is not allowed without approval

- Creating `outbox_events` table.
- Adding message broker dependencies.
- Adding CQRS/event handler architecture.
- Publishing domain events as required implementation pattern.
- Replacing service/repository flow with event-driven command handlers.

## If future architecture approves outbox

A future decision record must define:

- table schema;
- event payload rules;
- retry policy;
- idempotency keys;
- tenant scope;
- delivery guarantees;
- failure monitoring;
- cleanup/retention;
- which workflows publish events.

Until then, use only documented tables and workflows.

## Backend rule

If a developer or AI IDE sees this file, the correct action is to **avoid adding outbox code** unless a new approved ADR/database migration exists.

## Checklist

- [ ] No outbox table added without approved database update.
- [ ] No broker dependency added from this file.
- [ ] No CQRS/event handler conversion.
- [ ] Existing audit/history/payment/sync logs are used as documented.
