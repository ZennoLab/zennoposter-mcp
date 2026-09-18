# Security Policy

## Supported versions

| Component | Supported |
|---|---|
| MCP servers | Latest release, see [Releases](https://github.com/ZennoLab/zennoposter-mcp/releases) |
| ZennoPoster | 7.9.2 and newer |
| ZennoDroid | 2.6.1 and newer |

Older product versions have no PublicApi and cannot run these servers.

## Security model

The MCP servers are a thin layer over the product's PublicApi. What an AI assistant is allowed
to do is defined entirely by the API key you issue.

- **Local connections only.** The servers accept connections from the loopback interface only
  and refuse requests addressed to any other host name. They are not designed to be exposed to
  a network and must not be published through a reverse proxy or port forwarding.
- **API keys with scopes and tiers.** Every key carries a set of permissions (scopes) and a
  maximum operation tier. `T0` is read-only. Only read scopes are enabled by default when a key
  is created.
- **Keys are shown once.** A key is displayed only at creation time; afterwards the product
  keeps only a salted hash of it. A lost key cannot be recovered and should be deleted and
  reissued.
- **Structured authorization errors.** A missing or invalid key returns `401`. A key without
  the required scope or tier returns `403` naming what was required and what the key has, so a
  failure is never silent.
- **Internal ports.** The range `6107`-`6113` belongs to the product's own internal MCP
  infrastructure. Do not bind third-party processes to it.

More detail: [security model](https://zennolab.github.io/zennoposter-mcp/security-model.html).

## Recommended practice

- Issue a separate key per client and per machine, so a single key can be revoked without
  affecting others.
- Start with a `T0` read-only key and raise permissions only when read-only is no longer enough.
- On a shared machine, put the key into `appsettings.json` next to the server or into an
  environment variable rather than on the command line: command-line arguments are visible to
  other processes and stay in the shell history.
- Keep a copy of a project before letting an assistant modify it.
- Revoke keys issued on shared or temporary machines when the work is finished.
- Never commit an API key to a repository or paste it into a chat with a third-party service.

## Reporting a vulnerability

Report security issues privately through
[GitHub private vulnerability reporting](https://github.com/ZennoLab/zennoposter-mcp/security/advisories/new).
Do not open a public GitHub issue for a suspected vulnerability.

Please include:

- affected server and version;
- product and version (ZennoPoster / ZennoDroid);
- steps to reproduce, and the impact you were able to demonstrate.

Please give us a reasonable window to release a fix before any public disclosure.
