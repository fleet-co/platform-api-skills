---
name: fleet-platform-api-catalog
description: >-
  How Fleet Platform API catalog works: product groups vs catalog products, delivery_country,
  pricing (amount plus min/max shipping bounds with keyboard + country), and GET routes under
  `/platform/v1/catalog`. Use when browsing or quoting before carts.
---

# Platform API — catalog

Read **`fleet-platform-api`**. Routes are **`/platform/v1/catalog/...`** — all **GET**, **READ** key is enough.

## Concepts

- **Product group** — a catalog family (brand, category, presentation).
- **Products in a group** — **orderable catalog products** (variants under that group). Cart lines reference **product** ids (`products.id`).
- **`delivery_country`** — required (or central) on group list, product list, and pricing so Fleet can apply availability and logistics.
- **Pricing** — returns **`amount`** plus **`min_shipping`** and **`max_shipping`** (lower/upper shipping bounds that depend on **delivery country** and, when a keyboard choice applies, on **`keyboard_layout`**). Units and nullability: **OpenAPI**.
- **`keyboard_layouts`** on a listed product row — when present, allowed strings for **`keyboard_layout`** on pricing and cart lines.

## Endpoints (paths only)

1. `GET /catalog/product-groups` — required **`delivery_country`**; optional **`category`**, **`brand`**, **`search`**; pagination (default **`limit`** often **20**).
2. `GET /catalog/product-groups/:product_group_id/products` — required **`delivery_country`**.
3. `GET /catalog/products/:product_id/pricing` — required **`delivery_country`**, **`currency`**, **`acquisition_mode`**; optional **`keyboard_layout`** when required for that product.

**Errors (examples):** `DELIVERY_COUNTRY_NOT_FOUND`, `PRODUCT_NOT_FOUND`, `KEYBOARD_LAYOUT_REQUIRED`, `PRODUCT_PRICING_UNAVAILABLE`.
