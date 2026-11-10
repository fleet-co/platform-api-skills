# Fleet Platform API — reference notes (client-facing)

For **how to call** each area, use **`fleet-platform-api-devices`**, **`fleet-platform-api-users`**, **`fleet-platform-api-catalog`**, and **`fleet-platform-api-carts-orders`**. For field-level detail, use the **OpenAPI** document.

## Device `client_status`

Examples: **`ARCHIVED`**, **`PENDING_VALIDATION`**, **`IN_PREPARATION`**, **`DELIVERED`**, **`SHIPPED`**, **`AVAILABLE`**, **`IN_SERVICE`**, **`IN_MAINTENANCE`**, **`OUT_OF_SERVICE`** (full set in OpenAPI). Filter `GET /devices` with the same values.

## Keyboard layouts

- **`keyboard_layout`** on cart lines, catalog pricing, and devices uses the **same string enum** as **`keyboard_layouts`** on catalog product listing rows when Fleet exposes them.
