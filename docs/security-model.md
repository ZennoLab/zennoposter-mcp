**English** · [Русский](ru/security-model.md)

# Security model for integrators

**See also:** [index.md](index.md) (index) · [developer-guide.md](developer-guide.md) (quickstart,
key issuance, request examples) · [errors.md](errors.md) (401/403/404/409/429/501/503 handling) ·
[versioning-and-deprecation.md](versioning-and-deprecation.md) (semver policy).

Three things every integrator needs to understand before treating this API as a trust boundary.

## 1. Localhost only

Both hosts — ProjectMaker (`:5299`) and ZennoPoster Core (`:5300`) — accept loopback callers only.

A request whose peer address is not loopback (`127.0.0.0/8`, `::1`, or their IPv4-mapped forms) is
refused with a bodiless `403`, as is a request with no resolvable peer address. The check runs
before routing and before authentication, so it also covers the anonymous `T0` routes (`/ping`,
`/capabilities`, `/auth/whoami`). The MCP servers apply the same rule on their own ingress.

There is **no external network exposure** in this release, and no remote or TLS mode. Publishing
the OpenAPI contract and this documentation is exactly that — publishing the *contract*, not
opening the API to the network. If you need remote access, you are responsible for your own
tunnel or proxy and its security.

## 2. A key is not a god-key

An issued `ApiKey` is deliberately **not** equivalent to running as the machine's owner:

- **Scoped** — every operation requires a specific scope (`task:read`, `project:edit`,
  `code:author`, …); a key only carries the scopes it was issued with.
- **Tiered** — every operation also carries a risk tier (`T0` read → `T3` OS-level/RCE-class,
  e.g. OwnCode compile+run); a key only works up to its `maxTier`.
- **Revocable immediately** — revoking a key removes its record; the very next authentication
  attempt with that key fails (`401`). There is no propagation delay to reason about.
- **Expiring** — issued with an explicit absolute `expiresAt`; past expiry, authentication fails the
  same as a revoked key.
- **Never stored or recoverable in raw form** — the raw key is shown exactly once at issuance
  (GitHub-PAT style). Only a salted PBKDF2/HMAC-SHA256 hash (210 000 iterations, per-key random
  salt) is stored, in a key registry that is itself encrypted at rest. If you lose a raw key, there
  is no recovery path: revoke it and issue a new one.

**What this does *not* protect against:** a key does not defend against someone who controls the
machine the API runs on — they can read the process memory, the UI and the filesystem regardless.
What it *does* buy you: it closes the open-localhost hole to *other* local processes that don't
hold a valid key, it bounds how much any one integration (including an AI agent) can do (scope +
tier), and it gives every call a traceable identity (see audit, below).

## 3. Audit

Every authorized call is written to an append-only log (JSONL — one JSON object per line),
queryable via `GET /api/v1/audit` (admin scope only): timestamp, key id, method, path, status code,
required scopes, tier, duration. The audit log holds **call metadata only** — never a raw key or
its hash — so, unlike the key registry, it is **not** encrypted at rest; treat it as operational
log data, not a secret.

There is **no human-in-the-loop confirmation** for high-tier calls (`T2`/`T3`): authorization is
scope + tier + audit only. Don't design an integration assuming a human will be prompted before a
`T3` (OS-level) operation runs.
