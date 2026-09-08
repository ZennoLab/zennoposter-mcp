**English** · [Русский](ru/compatibility.md)

# Compatibility

**See also:** [index.md](index.md) (index) · [developer-guide.md](developer-guide.md) (quickstart,
key issuance, request examples) · [versioning-and-deprecation.md](versioning-and-deprecation.md)
(what counts as a breaking change).

Three version numbers move independently here: the product you installed, the API contract it
serves, and the MCP server binaries you downloaded. This page maps them onto each other.

## Supported combinations

Contract version: **1.1.0**.

Every server release states the minimum product version it works with; a dash means the server does
not apply to that product. Take the newest row your installed version satisfies — archives of every
release stay on the
[releases page](https://github.com/ZennoLab/zennoposter-mcp/releases).

### MCP.ProjectMaker

| Version | ZennoPoster | ZennoDroid |
|---|---|---|
| 0.2.0 | 7.9.2.0 and newer | 2.6.1.0 and newer |

### MCP.ZennoPoster

| Version | ZennoPoster | ZennoDroid |
|---|---|---|
| 0.2.0 | 7.9.2.0 and newer | 2.6.1.0 and newer |

### MCP.Instance

| Version | ZennoPoster | ZennoDroid |
|---|---|---|
| 0.2.0 | 7.9.2.0 and newer | — |

### MCP.Android

| Version | ZennoPoster | ZennoDroid |
|---|---|---|
| 0.2.0 | — | 2.6.1.0 and newer |

## API ports

| Application | ZennoPoster | ZennoDroid |
|---|---|---|
| ProjectMaker | 5299 | 5309 |
| ZennoPoster | 5300 | 5310 |

## Reading the versions off a live installation

One call gives the contract version and the product build behind it. 5299 below is ProjectMaker on
ZennoPoster; any other combination takes its port from the table above.

```bash
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/capabilities
```

- `version` is the contract version this host serves.
- `productVersion` is the product build behind it. An empty or absent value means the host does not
  report one — read that as unknown, not as old.
- `operations` is the authoritative list for that installation: an operation missing from it, or
  carrying `isAvailable: false`, cannot be called there.

The MCP server version is the one in the release tag and in the archive name, for example
`mcp-projectmaker-v0.2.0` and `MCP.ProjectMaker-v0.2.0-win-x64.zip`.

## When the numbers disagree

If `version` from `/capabilities` is below the contract version above, your installation is older
than this documentation — update the product, or work against the operation list it actually
reports.
