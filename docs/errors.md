**English** · [Русский](ru/errors.md)

# Error handling

**See also:** [index.md](index.md) (index) · [developer-guide.md](developer-guide.md) (quickstart,
key issuance, request examples) · [security-model.md](security-model.md) (localhost, key model,
audit) · [versioning-and-deprecation.md](versioning-and-deprecation.md) (semver policy).

Every error response carries an `ApiError` JSON body:

```json
{
  "error": "forbidden",
  "message": "Insufficient tier or missing scope",
  "required": { "tier": 2, "scopes": ["task:control"] },
  "current": { "tier": 0, "scopes": ["task:read"] }
}
```

`required`/`current` (each: `tier` + `scopes`) are only populated on `403`; every other status omits
them (`message` alone, or `null`).

## Status codes and machine-readable `error` codes

The table below is the complete set of `error` codes this version returns.

| HTTP status | `error` code | Meaning | Where it comes from |
|---|---|---|---|
| `400` | `bad_request` | Malformed request (bad JSON, invalid parameter). | Any operation, request-shape validation. |
| `401` | `unauthorized` | Missing or invalid `Authorization: Bearer <api-key>` — no key, unknown key, or an expired/disabled one. | Auth gate, every operation except `ping`. |
| `403` | `forbidden` | Key is valid but lacks the required scope, or the operation's tier exceeds the key's `maxTier`. Body includes `required`/`current`. | Auth gate. |
| `404` | `not_found` | Two distinct origins, same code: (a) the path/verb resolves to no known operation; (b) on the ZennoPoster host, the operation was resolved and authorized, but the task, instance, tab or element it addresses doesn't exist — e.g. `GET /tasks/{id}` with an unknown `id`. | (a) Auth gate; (b) ZennoPoster/Instance domain handlers, ZennoPoster host only. |
| `404` | `session_not_found` | `GET/POST /api/v1/sessions/{id}` — no open `WaitForUserAction` window with that id. | Sessions domain, ZennoPoster host only. |
| `409` | `no_active_interaction` | `POST /sessions/{id}/complete` targets a `WaitForUserAction` window that is no longer open — already completed, or the window/task closed. `instance:interact` calls do **not** return this: driving tabs/DOM needs no open session. | Sessions domain, ZennoPoster host only. |
| `409` | `session_expired` | *Declared in the contract for a session whose window closed before `complete` was called.* **Not returned by this version** — treat it as reserved, not as a code you will actually see in v1. | Sessions domain (reserved). |
| `409` | `dom_unavailable` | `GET /instances/{id}/tabs/{tabId}/dom` — the DOM text couldn't be retrieved right now (e.g. the page is navigating). Retry rather than treat as permanent. | Instance domain, ZennoPoster host only. |
| `409` | `instance_busy` | `DELETE /instances/{id}` — the port belongs to a running task's worker thread and cannot be released via the API. | Instance domain. |
| `409` | `instance_view_protected` | `POST /instances/{id}/show` — the browser's view is protected (view protection enabled and no open `WaitForUserAction` window), so the window cannot be revealed. | Instance domain. |
| `409` | `task_scheduler_owned` | `DELETE /tasks/{id}` — the task is owned by a scheduler job and cannot be deleted directly; delete the scheduler job instead. | Tasks domain, ZennoPoster host only. |
| `413` | `payload_too_large` | Request body exceeds the host's upload limit (default 2 GB — a safety cap, not something normal usage hits). Applies to any operation with a JSON body on the ZennoPoster host. | ZennoPoster host, global request-body guard (not operation-specific business logic). |
| `429` | `rate_limited` | Too many concurrent `GET /sessions/events` long-polls (per-host cap, default 32); also reserved for remote-mode rate limiting. | Sessions events long-poll; remote/TLS mode (not yet enabled). |
| `500` | `internal_error` | Unhandled server-side failure. | Any operation. |
| `501` | `not_implemented` | The path/verb is a declared operation that isn't wired to a handler on this host (e.g. the human-confirmation `confirmations_*` trio). | Auth gate. |
| `503` | `service_unavailable` | The AI/PublicApi runtime master-switch is off — every call is refused before the key is even checked. | Auth gate, checked before key/operation detail. |

## A note on 409

The **auth gate itself** never returns `409`: it only ever produces `401` / `403` / `404` / `501`.
`409` is a **domain-level** status, returned only by the sessions/instance domain when the call is
well-authorized but the runtime context it needs (an open interaction window) isn't there. Don't conflate the two: a `403` means "your key can't do this"; a `409` means
"your key can do this, but not *right now*".

## Handling checklist for an integrator

1. **`401`** — the key is missing, wrong, expired, or was revoked. Re-issue a key via the UI; there
   is no refresh flow (opaque keys, no token exchange).
2. **`403`** — read `required` vs `current` in the body and either request a key with the missing
   scope/tier, or don't attempt the call. Don't retry as-is; it will never succeed with the same key.
3. **`404` / `409`** on the sessions/instance domain — these are expected, not exceptional: a
   `WaitForUserAction` window can close between your `GET /sessions` listing and your
   `POST /sessions/{id}/complete` call. Re-list and confirm the session is still open before retrying.
4. **`429`** — back off; only relevant once remote mode ships (not in this release).
5. **`501`** — the operation is declared in the contract but not wired on this host
   (`isAvailable: false` in `/capabilities` for that `operationId`); don't call it.
6. **`503`** — the AI/PublicApi master-switch is off host-side; nothing will succeed until it's
   turned back on. Not something a client can work around.
7. **`409 dom_unavailable`** — transient; retry the DOM-text call rather than treating it as a
   permanent failure.
8. **`413`** — your request body exceeds the host's upload limit; this is a safety cap (default
   2 GB), not a normal-usage limit — if you hit it, something is likely wrong client-side.
9. Always check `GET /api/v1/capabilities` first (see [developer-guide.md](developer-guide.md)) to
   avoid triggering `403`/`501`s you could have predicted client-side.
