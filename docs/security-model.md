**English** · [Русский](ru/security-model.md)

# Security model for integrators

**See also:** [index.md](index.md) (index) · [developer-guide.md](developer-guide.md) (quickstart,
key issuance, request examples) · [errors.md](errors.md) (401/403/404/409/429/501/503 handling) ·
[versioning-and-deprecation.md](versioning-and-deprecation.md) (semver policy).

Seven things every integrator needs to understand before treating this API as a trust boundary.

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

Nor does it defend against code this product executes on your behalf. A project you bought but are
only allowed to run, and a third-party plugin, both execute inside the product's own process: such
code reads whatever key that process holds out of its own memory and reaches the API as a trusted
local caller. Read "cannot be opened in the editor" as protection for the author's work, not as a
safety property for whoever runs it.

What it *does* buy you: it closes the open-localhost hole to *other* local processes that don't
hold a valid key, it bounds how much any one integration (including an AI agent) can do (scope +
tier), and it ties every call to a named key you can revoke on its own.

## 3. Audit

Every call at tier `T2` or above is written to an append-only log (JSONL, one JSON object per
line), queryable via `GET /api/v1/audit` (admin scope only). That is the rule the tier table states,
so it covers running a project, executing an action, starting or stopping a task, creating an
instance, and it keeps covering the `admin` surface, which sits at `T0` but has been journaled
since it shipped. **Attempts that were refused are recorded too**, with the `401`/`403`/`429` they
got, because an attempt is often the interesting part.

**Each host keeps its own log**, so `GET /api/v1/audit` on ProjectMaker (`:5299`) answers for
ProjectMaker and the same call on ZennoPoster Core (`:5300`) answers for ZennoPoster. Ask both if
you want the whole picture. The two are separate processes; giving each its own file keeps them off
each other's back and means a log identifies its writer before you open it.

Each entry carries: timestamp, operation id, host, key id and label, method, path, status code,
tier, required scope, duration, peer address, and the `X-Zenno-Uow-Id` the caller sent if any.
Two notes on reading it:

- The **peer address** is always a loopback address, since that is the only kind this API accepts.
  It is recorded for completeness, not because it tells callers apart.
- The **unit-of-work id** is whatever the caller put in the header. It is useful for lining a call
  up with a running task; it is not proof of who made the call.

**`T0` and `T1` calls are not journaled.** Reads and reversible edits would bury the entries that
matter: an AI agent walking a project structure alone would produce thousands. If you need those,
the product's own logs are where to look.

The log holds **call metadata only**, never a raw key or its hash, so unlike the key registry
it is **not** encrypted at rest; treat it as operational log data, not a secret. It also **grows
without a size limit and is not rotated**: roughly 250 bytes per entry, so a busy runner will
accumulate tens of megabytes. Archiving it is yours to schedule.

## 4. Human confirmation

The machine's owner can require that a person releases a call before it runs. It is **off in a fresh
install**, because a runner with nobody at the keyboard has to keep working, and it is turned on from
**Settings → Api-Keys → Confirmations**, where the owner also picks the risk tier it starts at
(`T3` by default, the OS-level/RCE-class operations), how long a call waits, whether an unanswered
call is refused or allowed, and any operations to leave out.

What an integration sees when it is on:

- The call **blocks** while somebody decides. Nothing is returned, no job id, no polling handle:
  budget a client timeout longer than the owner's wait, or expect your own timeout to fire first.
- A refusal is `403` with `confirmation_rejected` (a person said no) or `confirmation_timeout`
  (nobody answered), both carrying the `confirmationId` in the error body. Neither means your key is
  insufficient, so retrying with a bigger key changes nothing.
- Nothing is retried for you. The confirmation is gone once it is answered, and a new call raises a
  new one.

The queue belongs to the process that parked the call, so a call to ProjectMaker is released through
ProjectMaker. `GET /api/v1/confirmations` lists what that host is holding, and
`POST /api/v1/confirmations/{id}/approve` or `.../reject` answers one. All three need the `admin`
scope, deliberately: whoever tripped the gate must not be able to release itself.

Note what this does and does not buy. It is the one control here that does not rest on a secret, so
it works the same against a stolen key and against code the product runs on your behalf, both of
which can reach the API but neither of which can click. What it cannot do is tell you afterwards what
happened: **ordinary domain calls are still not journaled**, confirmed or not (see section 3).

## 5. Repeated failed authentication is slowed down

Failed authentications are counted per presented credential and per peer address. Past the cap the
answer is `429` with a `Retry-After` header instead of another `401`, and it comes back before the
key is verified, so guessing tokens cannot be used to spend the server's CPU on password hashing.

This is **not** a lockout: nothing is disabled, no administrator has to unblock anything, and the
counters expire on their own within a minute. A key that authenticates successfully clears its own
counter immediately. That is deliberate: on a loopback-only surface every caller shares one
address, so a real lockout would let any local process disable your integration by failing on its
behalf.

What this means for a client: treat `429` as "wait and retry", honour `Retry-After`, and fix the
credential rather than retrying it in a tight loop.

## 6. Plugins are trusted by content, not by author

A plugin (`.zpg`) is a packed project that the product executes recursively, in the same thread and
process as the project calling it. Its manifest carries an author field, but that field is written by
whoever makes the file, so a "trusted authors" list would be defeated by the very thing it is meant to
stop. The product therefore does not have one.

What it has instead is a pin on the file itself. An `admin` key records the SHA-256 of an installed
plugin file with `POST /api/v1/plugins/trust` (body `{"path": "Local\\my-plugin.zpg"}`, relative to
the plugins folder or absolute), lists the pins with `GET /api/v1/plugins/trust`, and withdraws one
with `DELETE /api/v1/plugins/trust/{sha256}`. The owner can do the same from the "Trusted plugins"
dialog in the Api-Keys settings. The server hashes the file itself, so a pin always names bytes that
exist on this machine. Any change to the file, an update included, gives it a new hash and no trust
carries over: the owner looks at the new version and pins it again, or does not.

The execution gate consults the pins. A project that calls a plugin nobody pinned is opaque content
and demands `code:author` at `T3`, exactly as a project running its own C# does (section 2). A pinned
plugin runs under the caller's `project:run` like the rest of the project; the gate does not look
inside it, the pin is the owner's word for what it does. When a start is refused because of an
unpinned plugin, the `403` body names the plugin and its hash, so the owner knows what to pin, and
`GET /projects/current/structure` reports the same hash on each plugin cube as `contentHash`. A plugin
whose path is only known at run time (a macro) cannot be hashed in advance and stays opaque.

Worth keeping apart: a `.zpg` also ships execute-only, so the project calling it cannot open it as
source. That protects the plugin author's intellectual property — the caller can run the plugin but
not read what it does. It is not a safety measure for whoever runs the caller, and does not claim to
be one; safety for that person is the hash pin above, plus the process boundary section 2 describes.
Nothing here trusts a plugin any more because it happens to be execute-only, and nothing here needs it
to be for the pin to work.

Two limits, stated plainly. The pins are consulted by the API gate only: a person starting a project
from the product's own window is not asked, and the runner does not refuse an unpinned plugin. And a
pin proves the bytes, not their safety: a pinned plugin can do whatever the product can, which is the
boundary section 2 describes.

## 7. Code the product runs is kept off its internal AI stack

The product runs code that is not yours: a project you bought, a plugin, a C# snippet. That code
executes inside the product's own process (section 2). This section is about what it can reach from
there, and what the product does about it.

- The internal AI servers (the MCP servers behind the built-in chat and the AI action, the orchestrator
  that drives them) and the API of the chat window itself answer only callers that present a
  per-machine secret the product hands to its own processes. A call without it is refused with `401`.
  These ports are internal and not meant for integrations; use your own copy of the public MCP servers
  instead ([install.md](install.md)). When the secret itself could not be loaded, the chat API stays
  open to any local caller instead of refusing everyone — the same posture as before this protection
  existed.
- The orchestrator connects only to the MCP servers its host started, whatever endpoints a request
  names, so it cannot be pointed at a listener that would collect the secret.
- The product's local control channels (named pipes) accept only the product's own processes: an
  executable from the product's folder, or one signed by the same publisher.
- An AI action's browser calls carry the id of the task attempt they act for, and the Instance domain
  uses it to keep one attempt off another attempt's browser. That id is honoured only from the
  product's own AI channel. Sent with any other key, `X-Zenno-Uow-Id` is ignored as if it were absent,
  which is exactly what an integration that never sent it already gets. This keeps one legitimate AI
  session from cross-driving another attempt's browser; it is not a boundary against the adversary the
  closing paragraph below describes — code that already holds the product's rights can simply not send
  the id and land in the same unrestricted case as any other caller. The audit still records what the
  caller sent (section 3).

What this does and does not buy. None of it is a sandbox. To any local check, code running inside the
product's process is the product: it has the product's process id, so the pipe check lets it through,
and it shares the product's memory, so a determined author can dig the secret out and present it the
way the product does. What changes is the cost. Three lines of documented HTTP copied from a forum no
longer work; doing the same now means taking the product's internals apart. A guarantee needs project
code to run in a separate, restricted process, which a later release brings. Until then, treat every
project and plugin you run as having the product's rights (section 2).
