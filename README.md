**English** | [Русский](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.ru.md)

# Connecting the ZennoPoster/ProjectMaker MCP servers to your own LLM client

The MCP servers are published as self-contained `win-x64` binaries via GitHub Releases in
the shared repository **https://github.com/ZennoLab/zennoposter-mcp/releases**. Each
server has its own release line (its own tag prefix):

- **ProjectMaker (editor)** — tags `mcp-projectmaker-v*`, archive `MCP.ProjectMaker-v*-win-x64.zip`
- **Instance (browser control, dual-mount)** — tags `mcp-instance-v*`, archive `MCP.Instance-v*-win-x64.zip`
- **ZennoPoster (task runner)** — tags `mcp-zennoposter-v*`, archive `MCP.ZennoPoster-v*-win-x64.zip`
- **Android (ZennoDroid device)** — tags `mcp-android-v*`, archive `MCP.Android-v*-win-x64.zip`

PublicApi documentation (the OpenAPI contract rendered with Redoc, the integrator guide,
error codes, versioning policy) lives at **https://zennolab.github.io/zennoposter-mcp/**
(the pages are published from the `docs/` folder of this repository).

## The model: your own MCP instance with your own key

The product **itself** starts internal MCP sidecars for its built-in AI chat — they live on
the internal port band **6107–6113** (`ProductPorts.cs`; on ZennoDroid everything is shifted
by `+10`), receive a least-privilege service key from the host via stdin, and **ignore** the
`Authorization` header of incoming requests. This is internal infrastructure: connecting to
it from outside is not supported (permissions there are defined by the service key, not
yours), and **its ports must not be occupied** — a foreign process on a port from this band
prevents the built-in sidecar from starting (the product logs an error, but its AI stack is
left without that server).

For your own LLM client you run a **separate copy** of the MCP server from the public
package: it listens on the public port band **6207–6211** and talks to the same product
PublicApi (`:5299` ProjectMaker / `:5300` ZennoPoster), but with **your** ApiKey — with the
scopes/tier you chose when issuing the key. Both copies run side by side without
interfering with each other.

Default ports (set in the `appsettings.json` next to the exe):

| Server | Port | Talks to | Key in config |
|---|---|---|---|
| `MCP.ProjectMaker` | **6207** | PM PublicApi `:5299` | `NeuroBot:ApiKey` |
| `MCP.Instance` (Target=projectmaker) | **6208** | PM PublicApi `:5299` | `Instance:ApiKey` |
| `MCP.Instance` (Target=zennoposter) | **6209** (convention, set explicitly) | ZP PublicApi `:5300` | `Instance:ApiKey` |
| `MCP.ZennoPoster` | **6210** | ZP PublicApi `:5300` | `ZennoPosterApi:ApiKey` |
| `MCP.Android` (ZennoDroid) | **6211** | ZDroid PublicApi `:5309` | `Android:ApiKey` |

Everything can be overridden through standard ASP.NET Core configuration: the
`appsettings.json` next to the exe, environment variables (`ASPNETCORE_URLS`,
`NeuroBot__ApiKey`, …) or command-line arguments (`--urls`, `--NeuroBot:ApiKey=…`, …) —
arguments override environment variables, environment variables override
`appsettings.json`.

## 1. Download the server you need

On the releases page find the latest release of the server you need (by tag prefix),
download its `*-win-x64.zip` and unpack it into any folder.
Each archive is a single self-contained `.exe` + `appsettings.json`; no additional .NET
runtime is required.

## 2. Issue an ApiKey

In ProjectMaker: **Settings → API Keys** → **Add** — the "Add API key" dialog opens:
1. Set a `Label` (an arbitrary key name, to tell keys apart in the list).
2. Choose the `Max tier` (the minimum sufficient for your tasks — T0 for read-only, higher
   for mutating operations).
3. Tick the `Scopes` you need (only `*:read` are ticked by default; add others as needed).
4. Confirm — the raw key is shown **once**; copy it to a safe place immediately.

More on scopes/tiers — [security-model.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/docs/security-model.md).

## 3. Run the server with your key

The key is set **in the MCP server's own configuration** (not in the MCP client's headers —
the server accepts connections from loopback only and does not read the `Authorization`
header of incoming requests). The simplest way is to put the key into `ApiKey` in the
`appsettings.json` next to the exe and start with no arguments; or pass it via environment
variables / arguments:

```powershell
# ProjectMaker (editor): 6207 -> :5299
.\ZennoLab.AI.MCP.ProjectMaker.exe --NeuroBot:ApiKey=zp_xxx

# Instance for the editor (PM browser): 6208 -> :5299
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# Instance for the runner — a SECOND copy of the same exe: the port and the Target/BaseUrl pair are set explicitly
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx

# ZennoPoster (runner tasks/sessions): 6210 -> :5300
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPosterApi:ApiKey=zp_xxx
```

**Important note on the two `MCP.Instance` copies**: for the Instance server, `Target`
(which instructions it serves to the AI — about ProjectMaker or about ZennoPoster) and
`BaseUrl` (where HTTP requests actually go) are configured only together, as a pair, and
are not linked in code (`Target=projectmaker` → `BaseUrl` at the PM PublicApi `:5299`,
`Target=zennoposter` → `BaseUrl` at the ZP PublicApi `:5300`). On startup the server makes
a best-effort check via the target host's `/capabilities` and logs a warning on mismatch,
but if the target host is unreachable at startup the check is silently skipped — a
mismatch is then not detected, and the AI gets instructions about one host while requests
go to another.

## 4. Configure your LLM client

A ready-made configuration fragment (the format matches Claude Code's `.mcp.json`; use the
equivalent for your MCP client if needed):

```json
{
  "servers": {
    "projectmaker": {
      "type": "http",
      "url": "http://localhost:6207"
    },
    "instance-projectmaker": {
      "type": "http",
      "url": "http://localhost:6208"
    },
    "instance-zennoposter": {
      "type": "http",
      "url": "http://localhost:6209"
    },
    "zennoposter": {
      "type": "http",
      "url": "http://localhost:6210"
    }
  }
}
```

No authorization is needed on this leg: the MCP servers listen on loopback only, and the
permissions are defined by the key the server itself was started with (step 3).

## 5. Verify the connection

With any MCP client (or plain HTTP) call a safe read-only method and make sure it responds
`200 OK` with the expected data — e.g. `get_product_version`/`ping` on the server you need.
If the key is invalid or lacks a scope/tier, the server returns a structured error
(`401 unauthorized` / `403 forbidden` with `required`/`current` fields), not a silent
failure.
