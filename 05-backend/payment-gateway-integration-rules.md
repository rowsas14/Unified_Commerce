---
title: Payment Gateway Integration Rules
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

# Payment Gateway Integration Rules

The uploaded database design supports both recorded payments and provider/gateway traceability through payment provider configuration and payment transaction records.
The scope also requires clarification between manual card/QR references and integrated gateway payments.

## Database source

| Table | Purpose |
|---|---|
| `payment_method_types` | Global method types such as cash, card, QR, wallet, bank transfer, gift card. |
| `tenant_payment_methods` | Tenant-enabled payment methods. |
| `payment_provider_configs` | Tenant provider configuration using non-secret config and secret reference. |
| `payments` | Unified payment/outbound refund record. |
| `payment_transactions` | Provider event log per payment. |
| Allocation tables | Link payments to sales/orders/exchanges. |
| `refunds` | Business refund record linked to original and outbound payment. |

## Manual vs gateway-backed payment

| Type | Backend behavior |
|---|---|
| Cash | Record payment directly for POS when valid. |
| Manual card/QR reference | Store method, provider/reference if entered, and captured status according to workflow. |
| Gateway-backed payment | Use tenant provider config, store payment record, and store provider events in `payment_transactions`. |
| Refund | Validate against original captured payment and create outbound refund payment where applicable. |

## Secret rule

`payment_provider_configs.config` must hold non-secret config only.
Secrets must not be stored directly in JSON.
Use `secret_ref` for vault/secret manager reference.

## Payment service responsibilities

- Validate tenant payment method is enabled.
- Validate provider config belongs to tenant.
- Validate idempotency key.
- Create payment record.
- Create provider transaction record where applicable.
- Allocate payment to sale/order/exchange.
- Update status according to actual outcome.
- Validate refund limits.

## Payment statuses

Use statuses from the database design:

- `pending`
- `authorized`
- `captured`
- `failed`
- `cancelled`
- `voided`
- `partially_refunded`
- `refunded`
- `expired`

## Idempotency rule

Payment creation and gateway callbacks must be idempotent.
Use `payments.idempotency_key` where applicable and provider transaction IDs in transaction logs.

## Refund rule

Total refunds against `original_payment_id` must not exceed original captured amount.
Refund payment rows must be outbound and purpose `refund`.

## Offline payment rule

Offline card/QR behavior must follow documented business rule.
If a payment was captured outside the app, backend stores the reference and validates dedupe.
If not safely confirmable, backend must reject or conflict according to offline sync rules.

## Provider payload rule

Raw provider payload may be stored for integration trace in `payment_transactions.raw_payload`.
Do not expose raw payload to normal frontend responses.

## Checklist

- [ ] Tenant payment method is enabled.
- [ ] Provider config belongs to tenant.
- [ ] No plain secrets stored.
- [ ] Payment amount is positive.
- [ ] Captured amount does not exceed amount.
- [ ] Allocation does not exceed captured amount.
- [ ] Refund does not exceed original captured amount.
- [ ] Gateway events are logged when applicable.
