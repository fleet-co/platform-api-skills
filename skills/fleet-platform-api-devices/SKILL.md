---
name: fleet-platform-api-devices
description: >-
  How to use Fleet Platform API `/platform/v1/devices`: list with filters, get by id, batch
  create, patch, soft-delete; WRITE scope; common errors. Use when integrating device
  inventory, assignment, or lifecycle via HTTP.
---

# Platform API — devices

Read **`fleet-platform-api`** first (base URL, Bearer auth, envelopes, **READ vs WRITE**, **rate limits**). Paths are under **`/platform/v1/devices`**.

## Headers

- Always: `Authorization: Bearer <secret>`
- For `POST` / `PATCH`: `Content-Type: application/json`

## Responses

- **List:** `{ "data": [ … ], "meta": { total, limit, offset } }`
- **Get / patch / delete:** `{ "data": { … } }`
- **Batch create:** **201** `{ "data": { "devices": [ … ] } }`

Field lists and **`client_status`** enum: **OpenAPI**.

## WRITE

| Operations | WRITE |
|------------|-------|
| GET list, GET one | no |
| POST, PATCH, DELETE | yes |

Otherwise **`INSUFFICIENT_SCOPE`**.

## Operations (concepts)

1. **List** — `GET /devices` with `limit` / `offset`; optional **`search`**, **`user_id`**, **`client_status`** filter.
2. **Get** — `GET /devices/:device_id` — path id: positive integer string, no leading zero (`^[1-9]\d*$`).
3. **Create** — `POST /devices` with `{ "devices": [ … ] }` (min one item). Each item **must** include **`name`**, **`type`**, **`category`**; optional assignee (**`user_id`**), serial, specs, **`keyboard_layout`**, etc. (OpenAPI).
4. **Patch** — `PATCH /devices/:device_id` — at least one field; **`null`** clears where OpenAPI allows. Errors: **`DEVICE_NOT_FOUND`**, **`DEVICE_MUTATION_FORBIDDEN`**, **`EMPTY_PATCH_BODY`**.
5. **Delete** — `DELETE /devices/:device_id` — soft-delete semantics; same error families as patch when blocked.

**Common create errors:** `INVALID_SERIAL_NUMBER`, `DEVICE_PROVISIONING_FAILED`, `INVALID_USER` (bad assignee id).

## Keyboard layout

Use the same **`keyboard_layout`** strings as catalog **`keyboard_layouts`** (when Fleet lists them for that product) whenever a layout is required for the device or related flows.
