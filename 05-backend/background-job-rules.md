---
title: Background Job Rules
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

# Background Job Rules

The uploaded documents do not prescribe a specific background job framework.
This file documents only backend behaviors that may require deferred processing, without choosing a technology.
Do not introduce a job library until architecture approval.

## Allowed job-like responsibilities from current scope/database

| Responsibility | Why deferred processing may be needed |
|---|---|
| Offline sync processing | Large sync batches may be processed item by item. |
| Reporting read model refresh | Daily summaries derive from transaction tables. |
| OTP cleanup | Old OTP verification rows may need retention cleanup. |
| Cart expiry/reservation expiry | Carts and stock reservations have expiry semantics. |
| Payment provider event processing | Provider events may arrive asynchronously. |

## Not defined by uploaded docs

The uploaded docs do not define:

- a job framework;
- queue technology;
- message broker;
- retry library;
- scheduler product.

Do not invent these in feature implementation.

## Job safety rules

| Rule | Required behavior |
|---|---|
| Idempotency | Job can be retried safely. |
| Tenant scope | Every job item carries tenant context where needed. |
| Auditability | Sensitive outcomes create audit/history/sync audit records. |
| Failure visibility | Failed job items must be traceable. |
| Source of truth | Jobs must derive from source tables, not unsupported shadow state. |

## Offline sync processing

If sync processing is deferred, batch and item statuses must still reflect accurate processing state.
Use `offline_sync_audit_logs` for lifecycle events.

## Reporting refresh

Reporting read models are not source of truth.
They can be refreshed from sales, payments, inventory, discounts, returns, and exchanges.

## Expiry processing

Expiry-related concepts in uploaded docs include:

- carts `expires_at`;
- stock reservations `expires_at`;
- OTP `expires_at`;
- loyalty points `expires_at` where applicable.

If automated expiry processing is implemented, it must update statuses safely and leave traceable records where required.

## Checklist

- [ ] Job behavior is traceable to uploaded scope/database.
- [ ] No unapproved technology is introduced.
- [ ] Job is idempotent.
- [ ] Tenant context is preserved.
- [ ] Failure state is visible.
- [ ] Source-of-truth tables remain authoritative.
