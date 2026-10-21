**English** · [Русский](ru/data-collection.md)

# Data collection

ZennoPoster collects **aggregated product telemetry** for PublicApi usage.
This page describes what is and is not collected, when collection began, and how the data
is handled.

**Collection started:** ZennoPoster **7.9.3** · ZennoDroid **2.6.2** (planned).

## What is collected

### Daily PublicApi usage summary

On a timer (and once more on a clean shutdown) the product sends a daily summary to
`api/statistics/publicapi-usage`. The body carries two lists: `usage` and `issued`.

`usage` holds one row per combination of key, operation, client and edition for a given day:

| Field | What it contains |
|---|---|
| `date` | The UTC date the counts belong to (YYYY-MM-DD) |
| `keyId` | Identifier of the ApiKey — **never the key itself** |
| `keyKind` | `user` for a key you issued, `internal` for a key the product issues to its own MCP sidecars |
| `clientType` | The value of the `X-ZP-Client` header, or `direct-http` if absent |
| `operationId` | The PublicApi operation that was called, e.g. `tasks_list` |
| `edition` | Product edition (`Pro`, `Lite`, `Standard`, `Demo`, `ProjectMaker`, `ZennoBox`, `ZP8`), or `unknown` |
| `count` | Number of authorized calls that day for exactly this combination |

This is a **daily aggregate, not a request log**: no timestamps of individual calls, no
arguments and no results. It is, however, broken down per operation and per key — it is not a
single total for the day.

`issued` holds one row per key issuance:

| Field | What it contains |
|---|---|
| `date` | The date the key was issued |
| `keyId` | Identifier of the ApiKey — never the key itself |
| `keyKind` | `user` or `internal` |
| `edition` | Product edition at the time of issuance |

On the server the account is taken from the `sub` claim of the token the summary is sent with,
and rows are stored per account.

### OTel counters

Alongside the summary, two counters go to the telemetry collector:

```
zp7_publicapi_calls_total{operationId, keyKind, clientType, productName, productVersion, os}
zp7_publicapi_keys_issued_total{keyKind, productName, productVersion, os}
```

These carry **no key identifier and no account identifier** — only the low-cardinality labels
listed above. `os` is the OS release only (`Windows 10`, `Windows 11`), without the build number.

## What is NOT collected

- **Request or response bodies** — no template content, project code, variable values,
  action parameters, DOM data, or any user-supplied content.
- **The key value and its label** — only the key's identifier is sent.
- **Per-call detail** — no timestamp of an individual call, no arguments, no return value and
  no error text; only daily totals per combination.
- **Denied calls** — the counter runs only on an authorized call, so attempts rejected with
  401/403 are not counted at all.
- **Browsing or automation data** — no URLs visited, no form data, no screenshots.
- **IP addresses and machine names** — not sent by the product.

## Storage and handling

Telemetry is delivered over HTTPS to `zp7-service` (a ZennoLab backend). It is processed
and stored in accordance with the
[ZennoLab Privacy Policy](https://zennolab.com/privacy).

Collection is enabled by default for installations that have outbound HTTPS access to the
service endpoint. The product does **not** block startup or functionality if the endpoint is
unreachable — delivery failures are logged at debug level and the data is kept for the next
attempt.

The daily counts are also kept on the machine, in the product's data folder
(`PublicApiUsage.ZennoPoster.json`, `PublicApiUsage.ProjectMaker.json`), so that a restart later
in the day continues the day's count instead of starting it over. The file holds the same `usage`
rows that are sent, and nothing else; rows for past days are removed from it once delivered.

## See also

- [security-model.md](security-model.md) — what an ApiKey does and does not protect against,
  audit logging, localhost-only ingress.
- [developer-guide.md](developer-guide.md) — key issuance, scopes, and tier reference.
