**English** · [Русский](ru/index.md)

# ZennoLab PublicApi — developer documentation

The local HTTP API of ProjectMaker and ZennoPoster: what it exposes, how a key is issued and
scoped, what the errors mean, and how the contract evolves. The API is loopback-only — see the
security model.

- **[developer-guide.md](developer-guide.md)** — start here: quickstart, issuing a key,
  scopes/tiers, request examples per domain, `/capabilities`.
- **[api-reference.md](api-reference.md)** — the complete operation reference: every method
  grouped by domain with its verb, path, risk tier, required scope, request-body fields and
  response codes.
- **[versioning-and-deprecation.md](versioning-and-deprecation.md)** — semver policy (what's a
  breaking change), the deprecation window, and the `Deprecation`/`Sunset` header contract.
- **[errors.md](errors.md)** — every HTTP status / error code this API returns, and how to handle
  each one.
- **[security-model.md](security-model.md)** — what an ApiKey does and doesn't protect against,
  localhost-only ingress, and audit.
- **[openapi/openapi.v1.json](openapi/openapi.v1.json)** — the machine-readable OpenAPI 3.0.3
  document. **[openapi/index.html](openapi/index.html)** renders it with Redoc — that's the API
  explorer in the site navigation.

Connecting an LLM client to the MCP servers is documented separately, in the
[repository README](https://github.com/ZennoLab/zennoposter-mcp).

## Русская версия

A Russian translation of the hand-written documents lives in **[ru/](ru/index.md)**. The operation
reference and the OpenAPI document are English-only.
