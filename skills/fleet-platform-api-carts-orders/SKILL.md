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
- **Get cart / patch cart / add items:** `{ "data": { … } }`
- **Create cart:** **201** `{ "data": { "order_id" } }`
- **Validate:** `{ "data": { "order_id", "client_status" } }`
- **Get order:** rich read model — fields in **OpenAPI**

## Create cart body (concepts)

- **`purchaser_id`** — user placing the order.
- **`order_items`** — each needs **`product_id`**, **`delivery_country`**; optional **`keyboard_layout`** (required when pricing/catalog rules say so), optional **`user_id`** (assignee in your company), optional **`shipping_address_id`** (saved address id).
- **`acquisition_mode`** — **`LEASING`** (default) or **`BUY`**.

**Common errors:** `CART_ALREADY_EXISTS` (409), `INVALID_PURCHASER`, `INVALID_USER`, `INVALID_ADDRESS`, `KEYBOARD_LAYOUT_REQUIRED` (400), `DELIVERY_COUNTRY_NOT_FOUND`, `PRODUCT_NOT_FOUND`, `PRODUCT_PRICING_UNAVAILABLE` (404).

## Patch cart

At least one of **`purchaser_id`**, **`order_items`**, **`acquisition_mode`**. Sending **`order_items`** replaces all of the cart's lines. Sending **`acquisition_mode`** alone reprices the current lines in the new mode and switches the whole company's mode. **`CART_LOCKED`** (409) when an approved quote froze this cart's lines, or, for a mode switch, any of the company's carts.

## Add items

`POST /carts/:order_id/items` with **`order_items`** adds lines and keeps the current ones. **`CART_LOCKED`** (409) when an approved quote froze the cart.

## Validate

Body: **`purchaser_id`**; optional **`signatory_id`** (defaults to purchaser). Set **`is_signatory`** on the signatory first (`PATCH /users/:user_id`): validating with someone who is not yet a signatory still grants them the role, but that is deprecated and will be refused in a future version. Optional safeguards for checkout flows that show the cart before confirming:

- **`expected_device_ids`** — the cart device ids the purchaser reviewed; **`CART_CHANGED`** (409) if the cart no longer holds exactly these.
- **`require_signatory_role: true`** — **`SIGNATORY_REQUIRED`** (400) instead of granting the role to a signatory who lacks it.

Errors: `INVALID_PURCHASER`, `VALIDATE_CART_NO_DEVICES`, `SIGNATORY_REQUIRED` (400); `CART_NOT_FOUND` (404); `ORDER_NOT_CART`, `ORDER_STATUS_TRANSITION_FORBIDDEN`, `CART_CHANGED` (409).

## Orders

- **List** and **get** return validated orders only; a cart id on `GET /orders/:order_id` returns **`ORDER_NOT_FOUND`**.
- **Get** — detail including **`client_status`** and related fields per OpenAPI.

## 404 on cart get

`GET /carts/:order_id` returns **`CART_NOT_FOUND`** for an unknown id or a validated order.
