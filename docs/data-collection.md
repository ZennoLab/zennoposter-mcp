**English** · [Русский](ru/data-collection.md)

# Data collection

ZennoPoster collects **anonymized, aggregated product telemetry** for PublicApi usage.
This page describes what is and is not collected, when collection began, and how the data
is handled.

**Collection started:** ZennoPoster **7.9.3** · ZennoDroid **2.6.2** (planned).

## What is collected

### Aggregated PublicApi call counts

Once per day the product sends a daily summary to `api/statistics/publicapi-usage`:

| Field | What it contains |
|---|---|
| `client` | The value of the `X-ZP-Client` header, or `direct-http` if absent |
| `date` | The UTC date of the summary (YYYY-MM-DD) |
| `callCount` | Total number of successful PublicApi calls attributed to that client |

No per-request data is sent. The summary is an aggregate counter — a single number for the
day's total, not a list of individual calls.

### Key issuance facts

When an ApiKey is issued, a fact event is sent:

| Field | What it contains |
|---|---|
| `event` | `"key_issued"` |
| `issuedAtUtc` | Timestamp of issuance |
| `tier` | The `maxTier` the key was issued with (`T0`–`T3`) |
| `scopes` | The set of scopes the key carries |

The key value itself, the key label, the issuing user's name, and the machine's IP address
are **never** included.

## What is NOT collected

- **Request or response bodies** — no template content, project code, variable values,
  action parameters, DOM data, or any user-supplied content.
- **Named analytics** — no user identifiers, key labels, IP addresses, or machine names.
- **Per-call details** — which operation was called, when exactly it was called, or its
  return value.
- **Browsing or automation data** — no URLs visited, no form data, no screenshots.

## Storage and handling

Telemetry is delivered over HTTPS to `zp7-service` (a ZennoLab backend). It is processed
and stored in accordance with the
[ZennoLab Privacy Policy](https://zennolab.com/privacy).

Collection is enabled by default for installations that have outbound HTTPS access to the
service endpoint. The product does **not** block startup or functionality if the endpoint is
unreachable — delivery failures are logged at debug level and silently dropped.

## See also

- [security-model.md](security-model.md) — what an ApiKey does and does not protect against,
  audit logging, localhost-only ingress.
- [developer-guide.md](developer-guide.md) — key issuance, scopes, and tier reference.
