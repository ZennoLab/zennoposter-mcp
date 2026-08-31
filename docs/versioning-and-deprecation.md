**English** · [Русский](ru/versioning-and-deprecation.md)

# Versioning & deprecation policy

**See also:** [index.md](index.md) (index) · [developer-guide.md](developer-guide.md) (quickstart,
key issuance, request examples) · [errors.md](errors.md) (401/403/404/409/429/501/503 handling) ·
[security-model.md](security-model.md) (localhost, key model, audit).

Applies to every operation in the published contract.

## Contract version & base path

- Every operation lives under **`/api/v1`**. The major version is part of the path.
- The contract itself carries a semantic version, currently `1.0.0`.

## Semver rules

| Change | Class | Why |
|---|---|---|
| Documentation/description/example fix, no shape change | **PATCH** | Nothing a client's code depends on changes. |
| New operation, or a new **optional** field on a request/response | **MINOR** | Existing clients keep working unmodified; new capability is opt-in. |
| New scope grantable on a key, without changing any existing operation's required scope | **MINOR** | Purely additive to what a key *can* be issued; doesn't affect callers already using the API. |
| Removing or renaming a field, or an operation id | **MAJOR** | Breaks any client reading/writing that field or referencing that id. |
| Tightening validation, or narrowing/raising the required tier/scope for an existing operation | **MAJOR** | A call that used to succeed with a given key now fails with 403. |
| Changing a path, or an operation's HTTP verb | **MAJOR** | The client's request no longer resolves. |
| Changing a default (e.g. an operation's tier/scope requirement changes without the client asking for it) | **MAJOR** | Silent behavior change under an unchanged call shape. |

A new **MAJOR** version bumps the path segment (`/api/v1` → `/api/v2`); `/api/v1` keeps serving
existing clients until it is itself sunset (see below). Operation ids are part of the public
surface and are **never renamed or reused**, even across major versions.

### Examples

- **MINOR** — adding `GET /api/v1/tasks/{id}/state` (new, optional to use): existing clients
  unaffected.
- **MINOR** — adding an optional `priority` field to the `PUT /api/v1/tasks/{id}/config` request
  body: old clients that don't send it keep working with the existing default.
- **MAJOR** — renaming `sessionId` to `session_id` in the `/api/v1/sessions/*` response bodies: any
  client reading `sessionId` breaks.
- **MAJOR** — raising `POST /api/v1/tasks` from tier `T2` to `T3`, or adding a newly-required scope
  to an existing operation: a previously-sufficient key now gets `403`.

## Deprecation window & headers

- **Window: 90 days** from the moment an operation is announced deprecated to the moment it is
  removed (sunset), for any given operation.
- **Manifest**: a deprecated operation is published in the OpenAPI document with the standard
  `deprecated: true` flag plus an `x-sunset-date` extension (the sunset date, RFC 8594 HTTP-date
  format).
- **Response headers**: every response from a deprecated operation carries `Deprecation: true`
  (RFC 8594). The `Sunset` header follows once a concrete sunset date is announced for an
  operation.

### Zero deprecated operations today

No operation is deprecated. While `/api/v1` is **unreleased**, superseded operations are removed
outright instead of walking the deprecation cycle — for example the
`GET`/`PUT /tasks/{id}/settings/input` pair (fully covered by `GET`/`PUT /tasks/{id}/config`) and
`GET /tasks/{id}/threads` (covered by `countOfThreads` in `GET /tasks/{id}/state`) were dropped
without a deprecation window. The 90-day window and the `Deprecation`/`Sunset` headers apply from
the moment v1 ships to external integrators.

Declared-but-not-implemented operations (currently the confirmations trio,
`confirmations_list`/`confirmation_approve`/`confirmation_reject`) are a **feature gap**, not a
deprecation: they surface as `isAvailable: false` in `/capabilities` and answer `501`, not
`deprecated: true` in the OpenAPI document. The two concepts are distinct and should not be
confused when reading the manifest.

### Removed pre-contract routes: `/api/neurobot/*`

Before the versioned `/api/v1` surface existed, the ProjectMaker host also answered a set of
`/api/neurobot/*` aliases. They were never part of the published contract — absent from the OpenAPI
document and from `/capabilities` — and they no longer exist: `/api/neurobot/*` returns `404`, while
every documented `/api/v1` route is unaffected.

Because those aliases were outside `/api/v1`, their removal is **not** a semver change to the
published contract. The 90-day deprecation window above governs operations *inside* the contract
and does not apply to routes that were never in it.
