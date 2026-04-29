---
name: fleet-platform-api
description: >-
  Documents Fleet's public Platform HTTP API (`/platform/v1`): Bearer auth, READ vs WRITE
  scope, catalog/carts/orders/users/devices routes, pagination, JSON envelopes, and error
  shapes. Use when integrating with the platform API, writing clients, Postman collections,
  or answering questions about platform endpoints and request/response contracts.
---

# Fleet Platform API (`/platform/v1`)

## Base URL

- **Production API**: `https://api.fleet.co` — all platform routes are under **`/platform/v1`** (example: `https://api.fleet.co/platform/v1/devices`).
- **Sandbox / staging**: use the base URL Fleet provides for your environment (same path prefix **`/platform/v1`**).

## Contract (OpenAPI)

Use the **OpenAPI 3** document Fleet ships with the Platform API for exact schemas, enums, query parameters, and response bodies. It is the authoritative reference for field-level detail beyond this overview.

## Authentication

- Send **`Authorization: Bearer <your_platform_api_secret>`** on every request.
- The credential is bound to **your** Fleet workspace: every list, read, and mutation applies to that workspace’s data only.

## Rate limits (platform API)

- **Window:** **1 minute** per rolling/windowed counter (see Fleet’s deployment).
- **Production:** up to **100** requests per minute **per platform API credential** (normal authenticated traffic).
- **Sandbox:** up to **300** requests per minute **per credential**.
- **429** when exceeded; **`RateLimit-*`** headers may be present. If rate-limit storage is unavailable, **503** with a short “try again” message is possible—retry with backoff.

## Write scope

- API keys are issued with scope **`READ`** or **`WRITE`**.
- **Mutations** (creating or changing carts, users, devices, validating a cart) require **`WRITE`**. If the key is read-only, the API responds with **`{ "error": "INSUFFICIENT_SCOPE" }`**.
- **Read-only** (a `READ` key is enough): all `GET` endpoints (catalog, carts, orders, users, devices).

## Pagination and path IDs

- **Lists** accept **`limit`** and **`offset`** (integers). Typical default **`limit`** is **50**; catalog product-groups listing defaults **`limit`** to **20**. Maximum **`limit`** is **100** unless your OpenAPI states otherwise.
- **Path parameters** such as `order_id`, `user_id`, `device_id`, `product_id`, and `product_group_id` are **positive decimal integers as strings** with **no leading zero** (pattern `^[1-9]\d*$`, e.g. `42`, not `042`).

## Errors

- **Domain errors**: JSON body **`{ "error": "<CODE>" }`** where `<CODE>` is a stable machine-readable string (e.g. `ORDER_NOT_FOUND`, `CART_ALREADY_EXISTS`, `USER_EMAIL_CONFLICT`, `INSUFFICIENT_SCOPE`, `DEVICE_NOT_FOUND`). The OpenAPI document lists the codes relevant to each operation.
- **Validation errors**: JSON body with **`{ "error": "<message>" }`** (human-readable message) when the request fails schema validation.

## Success JSON envelope

- **Single resource** (get one, many PATCH responses, etc.): **`{ "data": <object> }`**.
- **Lists** (devices, users, carts, orders, catalog product-groups): **`{ "data": <array>, "meta": { "total", "limit", "offset" } }`**.
- **Batch device create**: **`{ "data": { "devices": [ ... ] } }`** with HTTP **201**.

## Task-oriented skills (calling the API)

Use these for concrete HTTP steps, example requests, and common error codes:

- **`fleet-platform-api-devices`** — list, get, batch create, patch, delete devices
- **`fleet-platform-api-users`** — list, get, create, patch, delete users
- **`fleet-platform-api-catalog`** — product groups, products in a group, pricing
- **`fleet-platform-api-carts-orders`** — carts, validate, orders

## Route map (summary)

| Method | Path                                                 | WRITE required |
| ------ | ---------------------------------------------------- | -------------- |
| GET    | `/catalog/product-groups`                            | no             |
| GET    | `/catalog/product-groups/:product_group_id/products` | no             |
| GET    | `/catalog/products/:product_id/pricing`              | no             |
| GET    | `/carts`                                             | no             |
| POST   | `/carts`                                             | yes            |
| GET    | `/carts/:order_id`                                   | no             |
| PATCH  | `/carts/:order_id`                                   | yes            |
| POST   | `/carts/:order_id/validate`                          | yes            |
| GET    | `/orders`                                            | no             |
| GET    | `/orders/:order_id`                                  | no             |
| GET    | `/users`                                             | no             |
| POST   | `/users`                                             | yes            |
| GET    | `/users/:user_id`                                    | no             |
| PATCH  | `/users/:user_id`                                    | yes            |
| DELETE | `/users/:user_id`                                    | yes            |
| GET    | `/devices`                                           | no             |
| GET    | `/devices/:device_id`                                | no             |
| POST   | `/devices`                                           | yes            |
| PATCH  | `/devices/:device_id`                                | yes            |
| DELETE | `/devices/:device_id`                                | yes            |

## Query/body highlights

- **Catalog product-groups**: Required **`delivery_country`** (ISO 3166-1 alpha-2); optional **`category`**, **`brand`**, **`search`**; pagination with catalog-specific default limit.
- **Catalog products in group**: Required **`delivery_country`**; path **`product_group_id`**.
- **Catalog pricing**: Required **`delivery_country`**, **`currency`** (ISO 4217 alpha-3), **`acquisition_mode`** (`LEASING` or `BUY`); optional **`keyboard_layout`** when pricing rules require it (see **`keyboard_layouts`** on listings from **`GET .../product-groups/:id/products`** when present).
- **Carts list**: Pagination + optional **`purchaser_id`**.
- **Cart line items**: Each line includes **`product_id`**, **`delivery_country`**, optional **`keyboard_layout`**, optional **`user_id`**, optional **`shipping_address_id`** (see **`fleet-platform-api-carts-orders`** and OpenAPI).

## Progressive disclosure

More behavioral notes (catalog filters, cart lifecycle, keyboard layouts) are in [reference.md](reference.md).
