# Platform API Skills

Small collection of Cursor skills for Fleet's Platform API.

## Structure

- `skills/fleet-platform-api/` - shared overview for `/platform/v1`: auth, scopes, errors, route map, addresses, company, and tickets
- `skills/fleet-platform-api-devices/` - devices, repairs, and storage stays endpoints
- `skills/fleet-platform-api-users/` - users endpoints
- `skills/fleet-platform-api-catalog/` - catalog endpoints
- `skills/fleet-platform-api-carts-orders/` - carts and orders endpoints

Each skill folder contains a required `SKILL.md`. Some skills also include optional reference docs.

The source of truth is `platform-docs/skills/` in the Fleet monorepo; sync this repository from it when the API changes.
