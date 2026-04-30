---
title: Transaction Boundary Rules
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

# Transaction Boundary Rules

Multi-table workflows must be committed through Unit of Work so sales, payments, stock, receipts, returns, exchanges, and offline sync do not become inconsistent.

## Unit of Work purpose

The uploaded backend architecture includes a Unit of Work folder under Infrastructure.
Use Unit of Work to coordinate repository changes inside one business transaction.

## Transaction examples

| Workflow | Tables involved |
|---|---|
| POS sale completion | `sales`, `sale_lines`, `payments`, `sale_payment_allocations`, `stock_movements`, `receipts` |
| E-commerce order placement | `orders`, `order_items`, `stock_reservations`, `payments`, `order_payment_allocations`, `order_status_history` |
| Return completion | `returns`, `return_lines`, `refunds`, `return_refund_allocations`, `stock_movements` |
| Exchange completion | `exchanges`, `exchange_lines`, `payments` or `refunds`, allocation tables, `stock_movements` |
| Offline sync item acceptance | sync tables plus accepted source-of-truth tables |
| Till close | `till_sessions`, `cash_count_denominations`, audit/report records where applicable |

## Boundary rule

A service method should define one business transaction boundary when the workflow must be atomic.
Do not commit partial sale/payment/stock state unless the workflow explicitly supports partial status.

## External provider caution

Payment gateway calls may happen outside or around DB transactions depending on provider behavior.
Do not hold long DB transactions across slow external calls without a documented design.
At minimum, persist traceable payment status and provider transaction results.

## Document sequence rule

Document number allocation must use row-level locking or equivalent safe transaction handling as described by the database rules for `document_sequences`.

## Stock movement rule

Stock movement and inventory balance update must stay consistent.
Do not create sale completion without required stock movement where stock is tracked.

## Offline sync rule

Sync item status and source-of-truth creation must be consistent.
If source record creation fails, item must be rejected or conflicted with error details.

## Audit timing

Audit records should be committed with the business action when they are required for traceability.
Do not audit success before the business transaction succeeds.

## Checklist

- [ ] Workflow tables identified.
- [ ] Unit of Work wraps required repositories.
- [ ] External calls handled safely.
- [ ] Document sequence allocation is safe.
- [ ] Stock/payment/report side effects are consistent.
- [ ] Failure leaves record in controlled status.
- [ ] Audit/history is committed with action.
