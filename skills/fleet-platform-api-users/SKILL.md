---
name: fleet-platform-api-users
description: >-
  How to use Fleet Platform API `/platform/v1/users`: paginated list, get, create, patch,
  soft-delete; roles; WRITE scope; common errors. Use when managing people records over HTTP.
---

# Platform API — users

Read **`fleet-platform-api`** first. Paths: **`/platform/v1/users`**.

## Headers

- `Authorization: Bearer <secret>`
- `Content-Type: application/json` on `POST` / `PATCH`

## Responses

- **List:** `{ "data": [ … ], "meta" }`
- **Get / create / patch / delete:** `{ "data": { … } }` — create returns **201**.

**`UserDetail`** includes **`id`**, **`company_id`** (metadata from Fleet), **`email`**, profile fields, **`role`** (`EMPLOYEE` \| `ADMIN`), **`is_signatory`** (can sign leasing contracts, on top of the role), and **`teams`** (`{ id, label }[]`). Full model: **OpenAPI**.

Optional **`team_ids`** on **create** / **patch** assigns teams; patch **replaces** all memberships when provided. **`USER_TEAM_NOT_FOUND`** when an id is missing or not a company team.

## WRITE

| GET | POST / PATCH / DELETE |
|-----|------------------------|
| no | yes |

## Operations (concepts)

1. **List** — `GET /users?limit=&offset=` — company implied by credential; optional **`search`** (name or email); `is_signatory=true` lists who can sign contracts.
2. **Get** — `GET /users/:user_id` — **`USER_NOT_FOUND`** if missing.
3. **Create** — `POST /users` — required **`email`** (work); optional **`secondary_email`** (personal, never used to sign in; `PATCH` with null clears it) + profile + **`role`** (default **EMPLOYEE**) + **`is_signatory`** + **`team_ids`**. **`SIGNATORY_PHONE_REQUIRED`** when `is_signatory` is true without a **`phone`**. **`USER_EMAIL_CONFLICT`** (also when the personal email belongs to another user of the company), **`USER_EMAIL_NOT_ALLOWED`**, **`USER_TEAM_NOT_FOUND`**.
4. **Patch** — `PATCH /users/:user_id` — omit vs **`null`** per OpenAPI (where allowed); optional **`team_ids`** (replace-all); **`is_signatory`** grants or removes the signatory role, and needs a phone in the body or on the user (**`SIGNATORY_PHONE_REQUIRED`**). **`USER_ROLE_UPDATE_FORBIDDEN`**, **`USER_TEAM_NOT_FOUND`** when applicable.
5. **Delete** — `DELETE /users/:user_id` — **`USER_DELETE_BLOCKED`** (409) when the user is the only possible purchaser of an order.

Statuses: `USER_NOT_FOUND`, `USER_TEAM_NOT_FOUND` (404); `USER_EMAIL_CONFLICT` (409); `USER_EMAIL_NOT_ALLOWED`, `SIGNATORY_PHONE_REQUIRED`, `USER_ROLE_UPDATE_FORBIDDEN` (400).
