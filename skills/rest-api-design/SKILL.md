---
name: rest-api-design
description: Design, review, or implement REST API contracts — resources, HTTP methods and status codes, errors, pagination, filtering, auth, idempotency, rate limits, and versioning. Not for GraphQL, gRPC, or just calling an existing API.
metadata:
  author: Aaron
  version: "2.0.0"
---

# REST API Design

Follow user instructions and the project's existing API conventions first; the defaults below apply only where none exist. Don't break a published contract just to match these defaults.

## Approach

- Read only the relevant routes, models, auth, error handling, and OpenAPI spec.
- For a new endpoint, settle method, path, auth, input, and success and error responses first; then pagination, retries, etc.
- Reuse the project's validators, serializers, and error middleware.
- Ask only when the answer changes the contract or causes a breaking change; otherwise pick a sensible default and state it.
- In reviews, report the location, trigger, impact, and minimal fix. Separate protocol bugs, compatibility risks, and style nits.

## Resources and paths

Plural nouns, lowercase, kebab-case; singletons like `/profile` may be singular.

```text
GET    /users                  # list
POST   /users                  # create → 201 + Location
GET    /users/{id}             # read
PUT    /users/{id}             # full replace
PATCH  /users/{id}             # partial update
DELETE /users/{id}             # delete → 204
GET    /users/{id}/orders      # owned sub-collection
POST   /orders/{id}/cancel     # explicit action
```

- Filter with query params (`/orders?status=pending`), not verbs in paths (`/getOrders`).
- Nest only for real ownership, and verify both the parent–child relation and access, not just the child ID.
- Model actions as state updates, sub-resources, or action endpoints, whichever fits. Don't hide a real action to avoid a verb.

## Methods

- `GET`, `HEAD`: safe and idempotent.
- `PUT`, `DELETE`: idempotent — same effect on repeat, not necessarily the same response (a second `DELETE` may return `404`).
- `POST`, `PATCH`: not idempotent by default.
- `PATCH` applies atomically: all or nothing. Define what missing fields and `null` mean. With `application/merge-patch+json`, an omitted field means unchanged, `null` deletes it, and arrays are replaced whole.

## Status codes

Return the real outcome; never wrap errors in `200`.

| Code | When |
| --- | --- |
| `200` | Success with body (including an empty list) |
| `201` | Created; include `Location` |
| `202` | Accepted for async processing; provide a status URL |
| `204` | Success, no body |
| `400` | Malformed request |
| `401` | Missing or invalid credentials; include `WWW-Authenticate` |
| `403` | Authenticated but not allowed (or `404` to hide existence) |
| `404` | Not found |
| `405` | Method not allowed; include `Allow` |
| `409` | Conflict with current state or a uniqueness constraint |
| `412` | Precondition (`If-Match`) failed |
| `413` / `415` | Payload too large / unsupported media type |
| `422` | Well-formed but semantically invalid |
| `429` | Rate limited; include `Retry-After` |
| `500` | Unexpected server error |
| `502` / `503` / `504` | Bad upstream / unavailable / upstream timeout |

Pick one rule for validation errors (`400` or `422`) and apply it everywhere.

## Request and response bodies

- Keep the project's field casing. Specify types, required fields, defaults, ranges, enums, read-only fields, and absent vs `null`.
- Timestamps in ISO 8601 with a timezone (`2026-09-01T10:30:00Z`). Money as integer minor units or a decimal string, plus a currency. Send integers above 2^53 as strings.
- Use one envelope style throughout: either bare resources or `{ "data": …, "meta": … }`. A `204` has no body.
- Use explicit input and output schemas. Never bind the request body straight to a model or return raw DB rows; clients must not be able to set `role`, `tenant_id`, or similar fields.

## Errors

Reuse the existing error format. For new projects, use Problem Details (`application/problem+json`):

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "errors": [{ "pointer": "/email", "code": "invalid_format" }],
  "request_id": "req_123"
}
```

- Clients branch on `type` or a stable `code`, never on message text.
- `status` must match the HTTP status.
- Point validation errors at fields.
- For 5xx errors, return only safe info and a request ID — no stack traces, SQL, or secrets.
- Malformed JSON must return `400`, not fall through to `500`.

## Pagination

Paginate any collection that can grow. Use offset or page numbers when users need to jump to a page and data is small; use a cursor (keyset) for feeds and large or changing data.

- Always use a deterministic order with a unique tiebreaker: `ORDER BY created_at DESC, id DESC`.
- Set a default and a maximum page size. Signal the end explicitly (`has_next: false` / `next_cursor: null`), not by a short page. Include `total` only if needed, and say whether it's exact.
- A cursor is an opaque token that encodes the last row's sort keys. Bind it to the filters and sort, validate it, and re-check auth on every page. Base64 isn't integrity protection; sign the cursor if tampering matters.
- Fetch `limit + 1` rows to detect a next page:

```sql
WHERE tenant_id = $1
  AND (created_at, id) < ($2, $3)   -- omit on the first page
ORDER BY created_at DESC, id DESC
LIMIT $4                            -- page size + 1
```

Back it with an index on the filter and sort columns, here `(tenant_id, created_at DESC, id DESC)`.

## Filtering, sorting, fields

```text
?status=pending&customer_id=usr_123
?price[gte]=10&price[lte]=100
?category=books,games
?sort=-created_at,-id
?q=wireless+headphones
?fields=id,name
```

- Define the semantics: AND/OR for multiple values, case sensitivity, date boundaries, nulls, and the default sort.
- Allowlist filter fields, operators, and sort keys; always bind values as parameters, never concatenate them into SQL.
- `fields` / `include` must not bypass field or relation permissions; cap expansion depth.

## Auth

- Keep the existing scheme. Send tokens in `Authorization: Bearer`, never in URLs.
- Check object-level and tenant access on every request — including lists, bulk operations, and nested resources. An unguessable ID is not authorization.

## Idempotency and concurrency

- For costly non-idempotent operations (payments, orders), accept an `Idempotency-Key`:
  - Scope it per caller and operation, store a fingerprint of the request, and set an expiry.
  - Replay the stored result for the same key and payload; reject the same key with a different payload.
  - Dedupe concurrent duplicates atomically.
- To prevent lost updates, use `ETag` + `If-Match` and check the version atomically in the write; return `412` on mismatch.
- Retry only idempotent requests or ones known not to have executed, with capped exponential backoff and jitter, honoring `Retry-After`.

## Rate limiting

- Set limits per caller and endpoint, based on cost and capacity.
- Return `429` with `Retry-After` when exceeded. An overloaded service returns `503` instead.
- Document any `RateLimit-*` / `X-RateLimit-*` headers you expose.

## Versioning and compatibility

- Keep the existing scheme; for new APIs, a path version (`/v1`) is simplest. Don't add a prefix to an unversioned API that's already in use.
- **Breaking changes** include removing or renaming a field, changing a type, adding a required input, tightening validation, changing permissions or the default sort. Even a new enum value can break strict clients and generated SDKs.
- For a breaking change, ship a new version or a migration path.
- Deprecate with a replacement and a timeline. Optionally send `Deprecation: @<unix-seconds>` and `Sunset: <HTTP-date>`; the sunset must not be earlier than the deprecation.

## Verification

Test what changed:

- Happy path, empty results, and not-found; check status code, headers, and body against the contract.
- Malformed JSON, boundary values, read-only fields, and unauthenticated, forbidden, or cross-tenant requests.
- Pagination edges, duplicate sort values, and invalid cursors.
- Duplicate or concurrent requests, conflicts, and rate limits.
- OpenAPI spec and examples match the implementation.

Report what was actually run; mark anything not run.
