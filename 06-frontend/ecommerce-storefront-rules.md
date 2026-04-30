---
title: E-Commerce Storefront Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# E-Commerce Storefront Rules

This document defines frontend rules for customer-facing e-commerce browsing, cart, checkout and order tracking.
It aligns with the production scope and database entities for products, carts, orders, customers, payments and fulfillment.

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

## Storefront purpose

The e-commerce frontend allows customers to:

- browse active online products;
- view product details and variants;
- manage cart;
- checkout as customer or guest where supported;
- place orders;
- track order/payment/fulfillment status.

## Data visibility rules

Only products marked for e-commerce availability should appear in storefront UI.
Frontend must not show products that backend/API excludes.

Relevant data concepts:

| Entity | Frontend use |
|---|---|
| `products` | Product listing/detail. |
| `product_variants` | Variant selection and sellable SKU. |
| `product_images` | Online product display. |
| `price_lists` / `price_list_items` | Price display via API result. |
| `inventory_balances` | Availability display where API exposes it. |
| `carts` / `cart_items` | Customer/guest cart. |
| `orders` / `order_items` | Checkout result and history. |
| `order_addresses` | Checkout address snapshots. |

## Product listing rules

Product listing must support:

- product search;
- category/filter display where implemented;
- variant-aware availability;
- price display from API;
- out-of-stock display rule returned by backend/config;
- e-commerce-only visibility.

Do not reuse POS product grid as storefront UI without adapting to customer browsing.

## Product detail rules

Product detail should show:

- product name;
- image(s) where available;
- description where available;
- variant selector;
- price;
- availability state;
- add-to-cart action;
- return policy information where displayed by API/config.

## Cart rules

Cart UI must:

- show item lines;
- allow quantity change/remove;
- show subtotal/discount/tax/shipping/grand total where available;
- validate current price and availability before checkout;
- distinguish guest and logged-in cart state;
- handle expired/converted/abandoned cart state from backend.

Frontend previews are not final checkout truth.

## Checkout rules

Checkout must collect/display:

- customer/guest identity information;
- shipping/billing address where required;
- delivery/pickup method;
- payment method or payment recording flow;
- order summary;
- final confirmation response.

Backend creates order and confirms status.

## Guest checkout rule

The scope supports guest checkout readiness.
Frontend must not merge customers globally across tenants.
Guest identity belongs to the tenant/order context.

## Payment rule

Payment UI must follow [[06-frontend/api-client-and-query-rules]] and [[04-api/payment-refund-api-rules]].
Frontend must not show order as paid unless backend confirms payment status.

## Order tracking rules

Customer order tracking should show separated state groups when exposed by API:

| State group | Meaning |
|---|---|
| Order status | Overall order lifecycle. |
| Payment status | Payment capture/refund state. |
| Fulfillment status | Pickup/delivery progress. |

Do not mix payment and fulfillment states into one confusing label.

## Wishlist and reviews

The database design includes wishlists and product reviews.
If the current storefront includes these features, UI must:

- show wishlist actions only for supported customer context;
- show review submission only where allowed;
- reflect moderation status for reviews;
- not display pending/rejected reviews as public approved content.

## Security/access rules

- Customer session is separate from tenant staff session.
- Do not expose admin/outlet operational data to customer storefront.
- Do not show other tenant products/customers/orders.
- Do not store unnecessary sensitive customer data in browser storage.

## Checklist

- [ ] Product listing uses e-commerce-visible API data.
- [ ] Cart validates through backend before checkout.
- [ ] Checkout does not assume payment success.
- [ ] Guest checkout remains tenant-scoped.
- [ ] Order tracking separates order/payment/fulfillment states.
- [ ] Wishlist/review UI follows backend status and access.
