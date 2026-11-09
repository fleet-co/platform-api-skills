---
name: fleet-platform-api
description: >-
  Documents Fleet's public Platform HTTP API (`/platform/v1`): Bearer auth, READ vs WRITE
  scope, catalog/carts/orders/users/devices/repairs/storage-stays/addresses/company/tickets routes, pagination, JSON
  envelopes, and error shapes. Use when integrating with the platform API, writing clients,
  Postman collections, or answering questions about platform endpoints and request/response
  contracts.
---

# Fleet Platform API (`/platform/v1`)

## Base URL

- **Production API**: `https://api.fleet.co` — all platform routes are under **`/platform/v1`** (example: `https://api.fleet.co/platform/v1/devices`).
- **Sandbox**: `https://demo-api.fleet.co` (same path prefix **`/platform/v1`**).

## Contract (OpenAPI)

Use the **OpenAPI 3** document at `https://api.fleet.co/platform/v1/openapi.json` for exact schemas, enums, query parameters, and response bodies. It is the authoritative reference for field-level detail beyond this overview; every operation has an `operationId`.

## Authentication

- Send **`Authorization: Bearer <your_platform_api_secret>`** on every request.
- The credential is bound to **your** Fleet company: every list, read, and mutation applies to that company's data only.
- A missing, invalid, or revoked key returns **401** with the plain-text body `Unauthorized`.

## Rate limits (platform API)

- Counted **per platform API credential** over a **1-minute** window.
- **Production:** up to **100** requests per minute. **Sandbox:** up to **300**.
- **429** with a plain-text body when exceeded; read the **`RateLimit-*`** headers for the reset. If rate-limit storage is unavailable, **503** with `{ "message": "..." }`: retry with backoff.

## Write scope

- API keys are issued with scope **`READ`** or **`WRITE`**.
- Every **POST**, **PATCH**, and **DELETE** requires **`WRITE`**. With a read-only key the API responds **403** **`{ "error": "INSUFFICIENT_SCOPE" }`**.
- Every **GET** works with a **`READ`** key.

## Pagination and path IDs

- **Lists** accept **`limit`** and **`offset`** (integers). Typical default **`limit`** is **50**; catalog product-groups listing defaults **`limit`** to **20**. Maximum **`limit`** is **100** unless your OpenAPI states otherwise.
- **Path parameters** such as `order_id`, `user_id`, `device_id`, `address_id`, `product_id`, and `product_group_id` are **positive decimal integers as strings** with **no leading zero** (pattern `^[1-9]\d*$`, e.g. `42`, not `042`).

## Errors

- **Domain errors**: JSON body **`{ "error": "<CODE>" }`** where `<CODE>` is a stable machine-readable string (e.g. `ORDER_NOT_FOUND`, `CART_ALREADY_EXISTS`, `USER_EMAIL_CONFLICT`, `INSUFFICIENT_SCOPE`, `DEVICE_NOT_FOUND`). The OpenAPI document lists the codes relevant to each operation.
- **Validation errors**: **400** with **`{ "message": "<every failure, comma-separated>", "type": "validation_error", "error": "validation_error" }`** when the path, query, or body fails schema validation. Branch on `error`, never on `message`.

## Success JSON envelope

- **Single resource** (get one, many PATCH responses, etc.): **`{ "data": <object> }`**.
- **Lists** (devices, repairs, storage stays, users, addresses, user addresses, tickets, carts, orders, catalog product-groups): **`{ "data": <array>, "meta": { "total", "limit", "offset" } }`**.
- **Batch device create**: **`{ "data": { "devices": [ ... ] } }`** with HTTP **201**.
- **Products of one catalog group**: **`{ "data": <array> }`**, every product at once, no `meta`.

## Task-oriented skills (calling the API)

Use these for concrete HTTP steps, example requests, and common error codes:

- **`fleet-platform-api-devices`** — list, get, batch create, patch, delete devices; read repairs and storage stays
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
| POST   | `/carts/:order_id/items`                             | yes            |
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
| GET    | `/repairs`                                           | no             |
| GET    | `/repairs/:repair_id`                                | no             |
| GET    | `/storage-stays`                                     | no             |
| GET    | `/storage-stays/:storage_stay_id`                    | no             |
| GET    | `/addresses`                                         | no             |
| POST   | `/addresses`                                         | yes            |
| GET    | `/addresses/:address_id`                             | no             |
| PATCH  | `/addresses/:address_id`                             | yes            |
| DELETE | `/addresses/:address_id`                             | yes            |
| GET    | `/user-addresses`                                    | no             |
| GET    | `/company`                                           | no             |
| GET    | `/tickets`                                           | no             |
| GET    | `/tickets/:ticket_id`                                | no             |
| POST   | `/tickets`                                           | yes            |

## Query/body highlights

- **Catalog product-groups**: Required **`delivery_country`** (ISO 3166-1 alpha-2); optional **`category`**, **`brand`**, **`search`**; pagination with catalog-specific default limit.
- **Catalog products in group**: Required **`delivery_country`**; path **`product_group_id`**.
- **Catalog pricing**: Required **`delivery_country`**, **`currency`** (ISO 4217 alpha-3), **`acquisition_mode`** (`LEASING` or `BUY`); optional **`keyboard_layout`** when pricing rules require it (see **`keyboard_layouts`** on listings from **`GET .../product-groups/:id/products`** when present).
- **Carts list**: Pagination + optional **`purchaser_id`**.
- **Company**: the API key's company: legal name, registration number for its country, ISO country, currency (use it for catalog pricing), acquisition mode, and its HR software connection (`hrms_integration`, null when none).
- **Tickets**: support requests to Fleet. `GET /tickets` filters by optional **`status`**. `POST /tickets` takes **`requester_id`** (an employee or admin of the company), **`subject`**, **`content`**, **`category`**, and optional **`order_id`** / **`device_id`** of the company; the response's **`reference`** is the one Fleet support quotes. **`TICKET_CREATION_FAILED`** (502) means support could not receive it and nothing was created; retry later.
- **Addresses**: `country` (ISO alpha-2, default `FR`); **`administrative_area`** (local or ISO 3166-2 code) required for US, CA, IT, AU, MX, JP, optional for ES, refused elsewhere (**`INVALID_ADDRESS`**, 400); `state` is deprecated. **`PATCH`** keeps the id for contact/label/type/comment/default changes, but a location change archives the address and returns a **new id** (old id → `ADDRESS_NOT_FOUND`): always store the returned id. **`DELETE`** archives.
- **User addresses**: `GET /user-addresses` lists `{ user_id, address_id }` links between employees and their personal addresses.
- **Cart line items**: Each line includes **`product_id`**, **`delivery_country`**, optional **`keyboard_layout`**, optional **`user_id`**, optional **`shipping_address_id`** (see **`fleet-platform-api-carts-orders`** and OpenAPI).

## Progressive disclosure

Keyboard layout and device status notes are in [reference.md](reference.md).
