---
name: fleet-platform-api-carts-orders
description: >-
  How Fleet Platform API carts and orders work: shared order_id, line items vs catalog,
  validate, acquisition_mode, and GET list/detail. Use when building checkout flows on
  `/platform/v1/carts` and `/platform/v1/orders`.
---

# Platform API — carts and orders

Read **`fleet-platform-api`**. Mutations need **WRITE**. Carts use **`order_id`** in the path (cart state is still an order).

## Headers

- `Authorization: Bearer <secret>`
- `Content-Type: application/json` on `POST` / `PATCH` / validate

## Response ideas

- **List carts / list orders:** `{ "data": [ … ], "meta" }`
- **Get cart / patch cart:** `{ "data": { … } }`
- **Create cart:** **201** `{ "data": { "order_id" } }`
- **Validate:** `{ "data": { "order_id", "client_status" } }`
- **Get order:** rich read model — fields in **OpenAPI**

## Create cart body (concepts)

- **`purchaser_id`** — user placing the order.
- **`order_items`** — each needs **`product_id`**, **`delivery_country`**; optional **`keyboard_layout`** (required when pricing/catalog rules say so), optional **`user_id`** (assignee in your workspace), optional **`shipping_address_id`** (saved address id).
- **`acquisition_mode`** — **`LEASING`** (default) or **`BUY`**.

**Common errors:** `CART_ALREADY_EXISTS`, `INVALID_PURCHASER`, `INVALID_ORDER_LINE`, `PRODUCT_NOT_FOUND`, `INSUFFICIENT_SCOPE`.

## Patch cart

At least one of **`purchaser_id`**, **`order_items`**, **`acquisition_mode`**.

## Validate

Body: **`purchaser_id`**; optional **`signatory_id`** (defaults to purchaser). Errors e.g. `VALIDATE_CART_NO_DEVICES`, `VALIDATE_CART_CONFLICT`, `PAYMENT_METHOD_REQUIRED`, `ORDER_STATUS_TRANSITION_FORBIDDEN`, `INVALID_USER`.

## Orders

- **List** — paginated summaries.
- **Get** — detail including **`client_status`** and related fields per OpenAPI.

## 404 on cart get

May return **`CART_NOT_FOUND`** or **`ORDER_NOT_FOUND`** depending on state—see OpenAPI for that GET.
