# Fleet Platform API — reference notes (client-facing)

For **how to call** each area, use **`fleet-platform-api-devices`**, **`fleet-platform-api-users`**, **`fleet-platform-api-catalog`**, and **`fleet-platform-api-carts-orders`**. For field-level detail, use the **OpenAPI** document Fleet provides.

## Catalog

- **Product groups** are families of merchandise; **products** under a group are the **orderable catalog products** (variants).
- **GET `/catalog/product-groups`**: **`delivery_country`** required; optional **`category`**, **`brand`**, **`search`**; paginated (default **`limit`** often **20**, max **100**).
- **GET `/catalog/product-groups/:product_group_id/products`**: **`delivery_country`** required. Listing rows may include **`keyboard_layouts`** when a layout must be chosen for pricing or carts.
- **GET `/catalog/products/:product_id/pricing`**: needs **`delivery_country`**, **`currency`**, **`acquisition_mode`**; optional **`keyboard_layout`** when required. Response includes **`amount`** plus **`min_shipping`** and **`max_shipping`** (bounds that depend on country and, when applicable, keyboard layout—see OpenAPI for units).

## Carts and orders

- **Carts** use **`order_id`** in paths; validate advances the workflow; errors like **`CART_ALREADY_EXISTS`** reflect Fleet rules (e.g. active cart per purchaser).
- **PATCH `/carts/:order_id`**: at least one of purchaser, line items, or acquisition mode.
- **POST `/carts/:order_id/validate`**: purchaser and optional signatory; may require devices on cart, payment readiness, etc.

## Users

- **List**: pagination; workspace implied by the credential.
- **Roles**: **`EMPLOYEE`** \| **`ADMIN`**; email uniqueness and delete blocking per OpenAPI error codes.

## Devices

- **List**: optional **`search`**, **`user_id`**, **`client_status`** (same uppercase values as in responses).
- **`client_status`** examples: **`ARCHIVED`**, **`PENDING_VALIDATION`**, **`IN_PREPARATION`**, **`DELIVERED`**, **`SHIPPED`**, **`AVAILABLE`**, **`IN_SERVICE`**, **`IN_MAINTENANCE`**, **`OUT_OF_SERVICE`** (full set in OpenAPI).
- **POST `/devices`**: body **`{ "devices": [ ... ] }`**; each item requires **`name`**, **`type`**, **`category`**.

## Keyboard layouts

- **`keyboard_layout`** on cart lines, catalog pricing, and devices uses the **same string enum** as **`keyboard_layouts`** on catalog product listing rows when Fleet exposes them.

## Rate limits

- **Production:** **100** requests **/ minute / platform API credential**. **Non-production:** **300** / minute / credential. **429** when over limit; **503** possible if limiter storage is unavailable (retry with backoff).
