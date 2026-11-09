---
name: fleet-platform-api-devices
description: >-
  How to use Fleet Platform API `/platform/v1/devices`, `/repairs` and `/storage-stays`: list
  with filters, get by id, batch create, patch, soft-delete, location, repair and storage
  history; WRITE scope; common errors. Use when integrating device inventory, assignment,
  location, or lifecycle via HTTP.
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

Field lists and **`client_status`** enum: **OpenAPI**. **`tracking_url`** on device detail is a read-only tracking page URL derived from **`tracking_number`** when needed.

**Assignment and location** (read-only on device detail): **`user_id`** (assignee → **`GET /users/:id`**), **`shipping_address_id`** (first delivery, never changes), **`assigned_address_id`** (where the device is kept once delivered), and **`location`** `{ type, address_id }` = where the device is now. `type` is `IN_TRANSIT` (address = destination), `COMPANY_ADDRESS` (address = assigned address), `AT_REPAIRER`, `FLEET_STORAGE`, `STOCK_REGION` or `NONE` (address `null`). Address ids are only returned for addresses of the caller's company.

**`repair`** is the open repair (else `null`); **`storage`** is the current storage stay while `ACTIVE` or `ENDING` (else `null`). Same shapes as **`GET /repairs/:id`** and **`GET /storage-stays/:id`**.

**`mdm`** holds MDM details once enrollment has started, else `null`.

**`parcels`** on device detail lists return and inter-office shipments (newest first). Initial delivery uses root **`tracking_number`** / **`tracking_url`**. Each parcel includes **`type`** (`RETURN` \| `INTER_OFFICE_DELIVERY`), **`status`** (`WAITING`, `SCHEDULED`, `SHIPPED`, `RECEIVED`, `FAILED`, `CANCELLED`), tracking fields, address ids (**`pickup_address_id`**, **`reception_address_id`** → **`GET /addresses/:id`**, see **`fleet-platform-api`** addresses), and dates. Read-only.

## WRITE

| Operations | WRITE |
|------------|-------|
| GET list, GET one (devices, repairs, storage stays) | no |
| POST, PATCH, DELETE | yes |

Otherwise **`INSUFFICIENT_SCOPE`**.

## Operations (concepts)

1. **List** — `GET /devices` with `limit` / `offset`; optional **`search`**, **`user_id`**, **`client_status`**, **`location_type`** filter (e.g. `location_type=FLEET_STORAGE` for every stored device).
2. **Get** — `GET /devices/:device_id` — path id: positive integer string, no leading zero (`^[1-9]\d*$`).
3. **Create** — `POST /devices` with `{ "devices": [ … ] }` (min one item). Each item **must** include **`name`**, **`type`**, **`category`**; optional assignee (**`user_id`**), serial, specs, **`keyboard_layout`**, etc. (OpenAPI). **`category`** must be a catalog / `device_category` enum code (e.g. `COMPUTER`, `SCREEN`) — not a display label like `Laptop`. Invalid values fail request validation (400), not insert. Rental and purchase dates are **`client_rental_start_date`**, **`client_rental_end_date`**, **`client_warranty_end_date`**, **`client_purchase_date`**; the former camelCase names are refused with a 400.
4. **Patch** — `PATCH /devices/:device_id` — at least one field (validation error otherwise); **`null`** clears where OpenAPI allows. Errors: **`DEVICE_NOT_FOUND`**, **`DEVICE_MUTATION_FORBIDDEN`** (403: only devices the company added can change), plus the create serial and keyboard errors.
5. **Delete** — `DELETE /devices/:device_id` — soft-delete semantics; same error families as patch when blocked.

**Common create errors:** validation errors for bad `category` / body shape; `INVALID_SERIAL_NUMBER`, `INVALID_APPLE_SERIAL_NUMBER`, `SERIAL_NUMBER_IS_IMEI`, `KEYBOARD_LAYOUT_REQUIRED`, `INVALID_USER` (400). **`DEVICE_PROVISIONING_FAILED`** (500) means the devices **were created** but MDM provisioning failed: list devices before retrying, or the retry creates duplicates.

## Repairs and storage stays (read-only)

- **`GET /repairs`** — newest first; optional **`device_id`**, **`status`** (`REQUEST_RECEIVED`, `AWAITING_PICKUP`, `PICKED_UP`, `REPAIR_IN_PROGRESS`, `IN_RETURN`, `ON_HOLD`, `COMPLETED`, `CANCELLED`). **`GET /repairs/:repair_id`** → **`REPAIR_NOT_FOUND`** when missing.
- **`GET /storage-stays`** — newest first; optional **`device_id`**, **`status`** (`ACTIVE`, `ENDING`, `ENDED`, `CANCELLED`; unsubmitted drafts are never returned). **`GET /storage-stays/:storage_stay_id`** → **`STORAGE_STAY_NOT_FOUND`** when missing.
- There is no write endpoint: repairs and storage are requested in the Fleet Cockpit or with Fleet support.

## Keyboard layout

Use the same **`keyboard_layout`** strings as catalog **`keyboard_layouts`** (when Fleet lists them for that product) whenever a layout is required for the device or related flows.
