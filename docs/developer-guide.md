**English** · [Русский](ru/developer-guide.md)

# Developer guide

This guide shows one representative example per domain, enough to get an integration running. For
the complete list of every operation, method, path, tier and scope, use the
[operation reference](api-reference.md) or the OpenAPI document.

**See also:** [index.md](index.md) (index) · [errors.md](errors.md) (401/403/404/409/429/501/503
handling) · [security-model.md](security-model.md) (localhost, key model, audit) ·
[versioning-and-deprecation.md](versioning-and-deprecation.md) (semver policy) ·
[openapi/index.html](openapi/index.html) (full OpenAPI reference, rendered with Redoc).

## Quickstart (5 minutes, from nothing)

1. **Issue a key** — see below. Copy the raw key; it is shown exactly once.
2. **Confirm the key works:**
   ```bash
   curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/auth/whoami
   ```
   Expect `200` with your key's `scopes`/`maxTier` echoed back.
3. **Check what you can actually do on this host:**
   ```bash
   curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/capabilities
   ```
   Walk the `operations` array — every one your key can call right now has `isAvailable: true`.
4. **Make your first safe (`T0`) call** — no side effects, e.g.:
   ```bash
   curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/projects/current
   ```
5. **Handle the predictable failure modes** before writing anything that mutates state — see
   [errors.md](errors.md).

`5299` above is ProjectMaker's default port; a ZennoPoster Core host binds its own port the same
way (`localhost` only either way — see [security-model.md](security-model.md)).

## Issuing a key

Keys are managed from the **API Keys** page in ProjectMaker/ZennoPoster Settings, available to
whoever runs the product on that machine — there is no separate remote signup flow. The "Add key"
dialog collects:

- **Label** — free text, for your own bookkeeping (shown in `GET /auth/keys`, never the raw key).
- **Max tier** — `T0`–`T3` (see the table below); defaults to `T0` (least privilege).
- **Scopes** — a checklist grouped by domain (`project:*`, `task:*`, `instance:*`, `code:*`,
  `admin`); only `*:read` scopes are pre-checked, everything else (including anything mutating,
  running, or `code:author`) is opt-in.
- **Expiry** — optional; on the wire (`POST /auth/keys`) it is an absolute `expiresAt` ISO-8601
  timestamp, `null`/omitted for a non-expiring key. A past date is rejected with `400`.

On confirm, the **raw key is displayed once** (GitHub Personal-Access-Token style) — copy it
immediately; it cannot be retrieved again (see [security-model.md](security-model.md) for why).
From then on, send it as `Authorization: Bearer <raw-key>` on every request except `GET /api/v1/ping`.

## Scopes and tiers

| Tier | Meaning | Gate |
|---|---|---|
| `T0` | Read / inspect, no side effects. | ApiKey only. |
| `T1` | Edits the project/model, reversible, no execution. | ApiKey + scope. |
| `T2` | Execution with the application's own privileges (run a task, complete a session). | ApiKey + scope + audit. |
| `T3` | Reaches the OS level — filesystem/network/OwnCode compile+run, or key/audit administration. RCE-class. | ApiKey + scope + audit. |

Scopes are namespaced by domain: `project:read/edit/run/record`, `task:read/edit/control`,
`instance:read/control/interact`, `code:read/author`, and the cross-cutting `admin`. The content
scopes (`io:filesystem`, `io:network`, `io:database`, `system:exec`, `code:author`) are demanded at
**execution time**: starting a task or project whose actions touch the filesystem, network,
database or OS requires the matching `io:*`/`system:exec` scope on top of the operation's own
scope, and a code-based or OwnCode-containing project requires `code:author`. The `403` body enumerates exactly
what is missing (`missingScopes`, `dangerousCategories`, `unknownCategories`).

Every operation carries exactly one tier and at most one required scope (`null` for
`ping`/`whoami`/`capabilities`, which only need a valid key, if any at all — `ping` needs none).

## Request examples by domain

All paths below are relative to `/api/v1`. Every response uses the shared envelope (`resultCode`
for the ProjectMaker domain, or a domain-specific body — see the OpenAPI document for the exact
per-operation schema; this guide shows one representative example per domain to get you oriented,
not the full set).

### ProjectMaker (`project:*`) — host `:5299`

```
GET  /projects/current           (T0, project:read)   — info about the open project
POST /projects/current/actions   (T1, project:edit)    — add an action to the project
POST /projects/current/actions/{groupId}/{actionId}/execute
                                  (T3, project:run)    — execute a single action (RCE class)
```

Adding an action whose `type` is `OwnCode` additionally requires
`code:author` (T3) on top of `project:edit`: authoring new executable code and running an existing
project are gated separately, so a low-privilege key can run a trusted project without being able
to inject and execute new code.

### ZennoPoster tasks (`task:*`) — host = ZennoPoster Core

```
GET  /tasks                (T0, task:read)     — list tasks
POST /tasks/{id}/start      (T3, task:control)  — start a task (content-classified, see above)
PUT  /tasks/{id}/config     (T1, task:edit)     — update a task's user settings
```

`POST /tasks` (add a task) is `T3`/`task:edit` — it either references a project file already on
disk (`application/json` body `{ "path": "C:\\bots\\my.zp" }`) or uploads the project bytes
(`application/octet-stream`, `?name=my.zp`). Only `.zp`, `.zpproj` and `.vsproj` are accepted.
Treat it as risky regardless of variant: an imported project can itself contain `OwnCode`.

**Which start-shaped calls are gated.** Everything that can launch project code runs the same
content-classification gate as `/start`: adding/setting a non-zero tries value on a READY task
(`Newbie`/`Complete`/`NotComplete` — the ZP7 semantics shared with the UI "+N" buttons),
`PUT /settings/execution` whose resulting tries counter is non-zero on a ready task, and
`PUT /settings/scheduler` with `isActive: true` (an active schedule is a deferred start). On a
stopped/running/scheduled task the same calls are plain `T1` edits.

#### Task settings

`GET /tasks/{id}` returns `settingsType`, which tells you where the task's user-facing settings
live: **`InputSettings`** — the project declares a raw input-settings block; **`BotUI`** — the
project uses a BotUI form; **`None`** — the project has no user settings. Either way,
`GET`/`PUT /tasks/{id}/config` is THE settings pair: it serves the unified view covering both block
kinds. The `settingsType` field can be empty until the project has been scanned at least once
(first start or UI open).

Settings are name+value pairs bound to project variables:

```json
// GET /tasks/{id}/config
{ "settings": [ { "name": "Login", "variable": "login", "value": "user@example.com" } ] }

// PUT /tasks/{id}/config — send only what you change; matching is by name (+variable to
// disambiguate duplicates). The response tells you what actually matched:
{ "applied": ["Login"], "skipped": ["Loginn"] }
```

A name in `skipped` matched nothing on the task (typo, or the setting belongs to the other block
kind) — the import semantics skip it rather than fail the batch.

#### Execution settings (merge semantics)

`PUT /tasks/{id}/settings/execution` merges: every field is optional and an omitted field keeps its
current value, so `{ "priority": 50 }` changes the priority and nothing else. `GET` returns the
full current set. Validation is strict (`400` otherwise): `limitOfThreads >= 1`;
`numberOfTries >= 0`; `priority` ∈ 10 (low), 50 (medium), 100 (high), 100000 (critical);
`maxNumOfSuccessStop`/`maxNumOfFailStop` = `-1` (unlimited) or `>= 1`; `timeout` = `-1` (none) or
minutes `>= 1`; `proxy` ∈ `DoNotUseProxy`, `IfPossible`, `UseProxyWithoutRemove`, `UseProxy`.
The system-managed `maxAllowOfThreads` ceiling is not writable — read it in `GET /tasks/{id}/state`.

#### Threads and tries

`/tasks/{id}/threads/max` (GET/PUT) is the user-set thread **limit** (`limitOfThreads`);
`/tasks/{id}/tries` (GET/PUT) and `/tries/add` (POST) manage the queued-tries counter. The number
of **currently running** threads is a different value — `countOfThreads` in `GET /tasks/{id}/state`.
PUT responses return the value actually applied after the commit, not an echo of the request.

#### Runtime state and failure points

`GET /tasks/{id}/state` returns counters plus `failurePoints`: a **snapshot of the last failure per
worker thread**, NOT an event queue. Reading is non-destructive — repeated GETs return the same
entries; an entry is overwritten in place when its thread fails again (keyed by taskId+threadId),
expires after 2 hours, is capped at 64 entries per task, and is fully cleared only when the task is
deleted. Deduplicate on the client by `(threadId, timestamp)` or `uowId`. Note: the `variables`
snapshot inside a failure point can contain sensitive values and is readable with `task:read` (T0) —
scope your keys accordingly.

#### Scheduler settings

`GET`/`PUT /tasks/{id}/settings/scheduler`. Formats: lists are comma-separated; `daysOfWeek` uses
English day names, capitalized (`Monday` … `Sunday` — other casings are accepted and normalized,
unknown names are a `400`); `intervals` entries are `"HH:mm-HH:mm"` (an end of `"00:00"` means
24:00 — end of day; a single time like `"09:30"` is a point interval); `attemptsRange`,
`repeatCountDayRange` and `repeatCountTotalRange` are `"n"` or `"a-b"`; **dates
(`startDate`/`endDate`) are invariant-culture strings in the MACHINE'S LOCAL TIME, with no time
zone** — the one exception to the API-wide absolute-time convention, because the scheduler runs on
the machine's clock. `isActive: true` requires a real `executePeriod`
(`400` on `NULL`) and is gated like `/start` (see above). Four typical bodies:

```json
// 1. One-time run at a specific date/time
{ "isActive": true, "executePeriod": "OneTime", "startDateType": "OnDate",
  "startDate": "08/30/2026 09:00:00", "attemptsRange": "1", "isClearSuccess": false }

// 2. Every day, between 09:00 and 18:00, repeating regularly through the day
{ "isActive": true, "executePeriod": "EveryDay", "startDateType": "Immediately",
  "attemptsRange": "1", "isClearSuccess": true,
  "intervals": ["09:00-18:00"], "stopExecutionOutsideOfIntervals": true,
  "repeatType": "Regularly", "repeatCountDayRange": "10", "endDateType": "Infinity" }

// 3. On selected weekdays
{ "isActive": true, "executePeriod": "EveryWeek", "daysOfWeek": ["Monday", "Wednesday", "Friday"],
  "startDateType": "Immediately", "attemptsRange": "1-3", "isClearSuccess": false,
  "repeatType": "Continued", "endDateType": "Infinity" }

// 4. When a trigger file appears (the file is deleted after firing)
{ "isActive": true, "executePeriod": "OnDemand", "fileName": "C:\\triggers\\run.txt",
  "isNeedDeleteFile": true, "attemptsRange": "1", "isClearSuccess": false }
```

### Interaction sessions (`task:*`) — host = ZennoPoster Core

A session is a live `WaitForUserAction` window on a running task/instance.

```
GET  /sessions                  (T0, task:read)     — list open windows
POST /sessions/{id}/complete    (T2, task:control)   — resolve the window, resume the thread
```

`{ "summary": "approved", "metadata": {} }` is a typical `complete` body. If the window already
closed (timeout, or another caller resolved it), you get `409 no_active_interaction` — see
[errors.md](errors.md). There is no `cancel`/fail-branch endpoint in this version.

A session's `started` is the UTC ISO-8601 moment the `WaitForUserAction` window opened; together
with `deadline` (present when the action has a timeout) it gives you the remaining time.

**Watching for windows:** `GET /sessions/events?timeoutSeconds=N` (1–60, default 25) is a bounded
long-poll — it blocks until at least one open/close event arrives (bursts are batched) or the
window elapses, then returns `{ "events": [...], "timedOut": bool }`. It is a live watch, not a
durable log: there is no backlog, so events between two polls are missed — re-poll in a loop, and
use `GET /sessions` for a point-in-time snapshot. Concurrent polls per host are capped (default
32; excess gets `429 rate_limited`).

### Instance (browser) — `instance:*`

Shared contract, two hosts: ProjectMaker exposes its single debug instance; a ZennoPoster runner
exposes one instance per running task/port. Driving tabs and the DOM is ordinary automation: an
`instance:interact` call needs no open session window — it is bounded by the `instance:interact`
scope and the operation's tier alone. (A `WaitForUserAction` window is simply the common case where
a task is *waiting* for you to drive it; it is not a precondition.)

```
GET  /instances                                     (T0, instance:read)
POST /instances/{id}/tabs/{tabId}/navigate          (T1, instance:interact)  — { "url": "...", "timeout": 5 }
POST /instances/{id}/tabs/{tabId}/elements/event    (T1, instance:interact)  — { "xpath": "...", "eventName": "click" }
```

**Release ≠ close.** `DELETE /instances/{id}` gracefully releases an API-started browser back to
the **pool**, and the pool decides release-vs-restart itself — the window may stay alive for
reuse; that is by design, not a leak. Ports owned by a running task's worker thread are protected
with `409 instance_busy`. `POST /instances/{id}/show` answers `409 instance_view_protected` when
the browser's view is protected (view protection enabled, without an open `WaitForUserAction`
window) — a `200` always means the window was actually revealed.

### Cross-cutting (`admin`, or no scope)

```
GET  /auth/whoami      (T0, no scope)  — your key's own scopes/tier
GET  /capabilities     (T0, no scope)  — the manifest below
POST /auth/keys        (T3, admin)     — issue a key (same operation the UI calls)
GET  /audit            (T0, admin)     — the call log
```

**`GET /audit` is the journal of ALL PublicApi HTTP calls on the host** — not just key
operations. Each record carries the timestamp, the key's id/label (never the token itself), the
method, path and status code; filter with `?since=`/`?until=` (ISO-8601), `?keyId=`, `?method=`,
`?statusCode=`, page with `?skip=`/`?take=`.

**`GET /auth/keys`** lists every key record with `systemProvisioned` (auto-issued by the host for
its own built-in MCP servers — you did not create those, and the "My keys" UI hides them) and
`status` (`active`, `expired` or `disabled`). An expired/disabled key already fails
authentication but stays listed until revoked — revocation removes the record physically and is
irreversible. Filter with `?includeSystemProvisioned=false` and `?status=` (`active`, `expired`
or `disabled`).

**`GET /code-api`** (ProjectMaker host, `code:read`) serves the C# API surface available to
OwnCode cubes, from the product's embedded code-api digest. Quick recipe:

```bash
# index of available types
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/code-api
# members of one type
curl -H "Authorization: Bearer <api-key>" "http://localhost:5299/api/v1/code-api?typeName=IZennoTable"
```

## `/capabilities`

`GET /api/v1/capabilities` returns, for the key on the request (or an anonymous view if called
without one — see below):

```json
{
  "host": "zennoposter",
  "version": "1.0.0",
  "currentScopes": ["task:read", "task:control"],
  "currentMaxTier": 2,
  "operations": [
    {
      "operationId": "session_complete",
      "method": "POST",
      "path": "/api/v1/sessions/{id}/complete",
      "tier": 2,
      "requiredScope": "task:control",
      "isAvailable": true
    }
  ],
  "tierLegend": { "T0": "read", "T1": "edit", "T2": "execute+audit", "T3": "host/RCE" }
}
```

`host` tells you which surface you're talking to (`zennoposter` serves `ZennoPoster`+`Instance`+
cross-cutting operations; ProjectMaker serves `ProjectMaker`+`Instance`+cross-cutting — the two
hosts do **not** expose the same operation set). `isAvailable` is `true` only when the operation is
**both** implemented on this host **and** allowed for your key (scope + tier) — a
`false` entry can mean either "not yours to call" or "not built yet on this host"; check
`requiredScope`/`tier` against your own `currentScopes`/`currentMaxTier` to tell which. Calling
`/capabilities` before attempting a call is the cheapest way to avoid a predictable `403`/`501`.
