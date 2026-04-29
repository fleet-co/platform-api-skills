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

**`UserDetail`** includes **`id`**, **`company_id`** (metadata from Fleet), **`email`**, profile fields, **`role`** (`EMPLOYEE` \| `ADMIN`). Full model: **OpenAPI**.

## WRITE

| GET | POST / PATCH / DELETE |
|-----|------------------------|
| no | yes |

## Operations (concepts)

1. **List** — `GET /users?limit=&offset=` — workspace implied by credential.
2. **Get** — `GET /users/:user_id` — **`USER_NOT_FOUND`** if missing.
3. **Create** — `POST /users` — required **`email`**; optional profile + **`role`** (default **EMPLOYEE**). **`USER_EMAIL_CONFLICT`**, **`USER_EMAIL_NOT_ALLOWED`**.
4. **Patch** — `PATCH /users/:user_id` — omit vs **`null`** per OpenAPI (where allowed). **`USER_ROLE_UPDATE_FORBIDDEN`** when applicable.
5. **Delete** — `DELETE /users/:user_id` — **`USER_DELETE_BLOCKED`** when rules forbid.
