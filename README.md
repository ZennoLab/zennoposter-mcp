**English** | [Русский](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.ru.md)

# Connecting the ZennoPoster and ZennoDroid MCP servers to your own LLM client

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

These servers talk to **ZennoPoster 7.9.3 and newer** and to **ZennoDroid 2.6.1 and newer**.
`MCP.Instance` applies to ZennoPoster only and `MCP.Android` to ZennoDroid only; `MCP.ProjectMaker`
and `MCP.ZennoPoster` apply to both.
Which server version goes with which product and contract version:
**https://zennolab.github.io/zennoposter-mcp/compatibility.html**.

Install buttons for Cursor and VS Code and the Claude Code commands for every server:
**https://zennolab.github.io/zennoposter-mcp/install.html**.

## The model: your own MCP instance with your own key

The product **itself** starts internal MCP servers for its built-in AI chat — they live on
the internal port band **6107–6113** (on ZennoDroid everything is shifted by `+10`), receive a
least-privilege service key from the host, and **ignore** the `Authorization` header of
incoming requests. This is internal infrastructure: connecting to
it from outside is not supported (permissions there are defined by the service key, not
yours), and **its ports must not be occupied** — a foreign process on a port from this band
prevents the built-in server from starting (the product logs an error, but its AI stack is
left without that server).

Those internal servers have no console of their own, so the product copies everything they
write to their output into its own log, each line prefixed with the process name
(`MCP.ProjectMaker`, `Orchestrator`, …). Ordinary start-up chatter is tagged `Debug` and never
reaches the log at all; only what the server itself reports as a warning or a failure, and the
exit code of a server that dies unprompted, actually get written. That is the place to look
when the built-in chat loses one of its servers.

Those internal servers also expect a per-machine secret that the product hands only to its own
AI processes, so being able to reach the port is no longer enough to use one: a call without it
is **refused with `401`**, and the refusal is written to the product's log as `ingress refused`.
The same goes for the built-in chat's own API on 6113. If you were calling those ports from your
own client, run your own copy on the public band instead, as described below.

For your own LLM client you run a **separate copy** of the MCP server from the public
package: it listens on a public port (the tables below) and talks to the same product
PublicApi, but with **your** ApiKey — with the scopes/tier you chose when issuing the key.
Both copies run side by side without interfering with each other.

On **ZennoDroid** the two servers that serve both products listen `+10` higher — `MCP.ProjectMaker`
on **6217**, `MCP.ZennoPoster` on **6220** — the same shift the product applies to its internal band.
That way a machine with both products installed can run both sets at once. `MCP.Android` exists only
on ZennoDroid and keeps **6211**: nothing on ZennoPoster occupies it.

Default ports, one table per product. A server takes its listen port and `BaseUrl` from the
`appsettings.json` next to its exe; the ports marked ¹ have no such defaults and are passed on the
command line together with the matching `BaseUrl` (and `Target`), as in the examples of step 3.

**ZennoPoster**

| Server | Port | Talks to (PublicApi) | Key in config |
|---|---|---|---|
| `MCP.ProjectMaker` | **6207** | ProjectMaker `:5299` | `ProjectMaker:ApiKey` |
| `MCP.Instance`, `Target=projectmaker` (editor browser) | **6208** | ProjectMaker `:5299` | `Instance:ApiKey` |
| `MCP.Instance`, `Target=zennoposter` (task browsers) | **6209** ¹ | ZennoPoster `:5300` | `Instance:ApiKey` |
| `MCP.ZennoPoster` | **6210** | ZennoPoster `:5300` | `ZennoPoster:ApiKey` |

**ZennoDroid**

| Server | Port | Talks to (PublicApi) | Key in config |
|---|---|---|---|
| `MCP.ProjectMaker` | **6217** ¹ | ProjectMaker `:5309` | `ProjectMaker:ApiKey` |
| `MCP.ZennoPoster` | **6220** ¹ | ZennoPoster `:5310` | `ZennoPoster:ApiKey` |
| `MCP.Android`, `Target=projectmaker` (editor device) | **6211** | ProjectMaker `:5309` | `Android:ApiKey` |
| `MCP.Android`, `Target=zennoposter` (task devices) | **6212** ¹ | ZennoPoster `:5310` | `Android:ApiKey` |

¹ Not a built-in default: set it explicitly (`--urls` plus the matching `BaseUrl`; the commands are in step 3).

Everything can be overridden through standard ASP.NET Core configuration: the
`appsettings.json` next to the exe, environment variables (`ASPNETCORE_URLS`,
`ProjectMaker__ApiKey`, …) or command-line arguments (`--urls`, `--ProjectMaker:ApiKey=…`, …) —
arguments override environment variables, environment variables override
`appsettings.json`.

## 1. Download the server you need

On the releases page find the latest release of the server you need (by tag prefix),
download its `*-win-x64.zip` and unpack it into any folder.
Each archive is a single self-contained `.exe` + `appsettings.json`; no additional .NET
runtime is required. For Claude Desktop take the `.mcpb` bundle of the same release instead, see
[Claude Desktop: install from an .mcpb bundle](#claude-desktop-install-from-an-mcpb-bundle).

## 2. Issue an ApiKey

In ProjectMaker: **Settings → Api-Keys** → **Add new API key** — a dialog of the same name opens:
1. Set a `Label` (an arbitrary key name, to tell keys apart in the list).
2. Choose the `Max tier` (the minimum sufficient for your tasks — T0 for read-only, higher
   for mutating operations).
3. Tick the `Scopes` you need (only `*:read` are ticked by default; add others as needed).
4. Press **Generate API Key** — the raw key is shown **once**; copy it to a safe place
   immediately.

More on scopes/tiers — [security-model.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/docs/security-model.md).

## 3. Run the server with your key

The key is set **in the MCP server's own configuration** (not in the MCP client's headers —
the server accepts connections from loopback only and does not read the `Authorization`
header of incoming requests). A server whose port in the tables above has no ¹ can take the
key from `ApiKey` in the `appsettings.json` next to its exe and start with no arguments; the
ports marked ¹ are always set on the command line, together with the matching `BaseUrl` (and
`Target`). The key can also be passed as an environment variable or an argument, as below. Run
only the servers you need, from the block of your product.

**ZennoPoster**

```powershell
# ProjectMaker (editor): 6207 -> :5299
.\ZennoLab.AI.MCP.ProjectMaker.exe --ProjectMaker:ApiKey=zp_xxx

# Instance for the editor (PM browser): 6208 -> :5299
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# Instance for the runner — a SECOND copy of the same exe: the port and the Target/BaseUrl pair are set explicitly
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx

# ZennoPoster (runner tasks/sessions): 6210 -> :5300
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPoster:ApiKey=zp_xxx
```

**ZennoDroid**

```powershell
# ProjectMaker (editor): 6217 -> :5309
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:6217 `
  --ProjectMaker:BaseUrl=http://localhost:5309/api/v1 --ProjectMaker:ApiKey=zp_xxx

# ZennoDroid (runner tasks/sessions): 6220 -> :5310
.\ZennoLab.AI.MCP.ZennoPoster.exe --urls http://localhost:6220 `
  --ZennoPoster:BaseUrl=http://localhost:5310/api/v1 --ZennoPoster:ApiKey=zp_xxx

# Android (the device attached to ProjectMaker): 6211 -> :5309
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx

# Android for the runner — a SECOND copy of the same exe, for the devices of running tasks:
# the port and the Target/BaseUrl pair are set explicitly
.\ZennoLab.AI.MCP.Android.exe --urls http://localhost:6212 `
  --Android:Target=zennoposter --Android:BaseUrl=http://localhost:5310/api/v1 `
  --Android:ApiKey=zp_xxx
```

`MCP.Android` mounts twice for the same reason `MCP.Instance` does on ZennoPoster: the Android
domain is served both by ProjectMaker's PublicApi (the device you see in the editor) and by the
runner's (the devices its tasks are driving). Like `MCP.Instance` it takes a `Target` —
`projectmaker` for the editor's device, `zennoposter` for the runner's — which selects the
instructions the server hands to the AI at initialize (one device whose id may be left at 0, versus
one device per worker thread with the id taken from `list_devices`); the tool set is the same
either way. `Target` and `BaseUrl` are set together as a pair, exactly as for `MCP.Instance` (see
the note below). `Target` appeared in `MCP.Android` 0.4.0; earlier versions differ only by `--urls`
and `BaseUrl`.

**Important note on the two copies of `MCP.Instance` and of `MCP.Android`**: for these servers,
`Target` (which instructions they serve to the AI — about the ProjectMaker editor or about the
runner) and `BaseUrl` (where HTTP requests actually go) are configured only together, as a pair,
and are not linked in code (`Target=projectmaker` → `BaseUrl` at the PM PublicApi `:5299`,
`Target=zennoposter` → `BaseUrl` at the ZP PublicApi `:5300`; on ZennoDroid `:5309` and `:5310`).
On startup the server makes a best-effort check via the target host's `/capabilities` and logs a
warning on mismatch, but if the target host is unreachable at startup the check is silently
skipped — a mismatch is then not detected, and the AI gets instructions about one host while
requests go to another.

## stdio: the server as a subprocess of your client

Besides listening on a port, every server can speak MCP over its own stdin and stdout. Start the exe
with `--stdio` and the client that launched it owns the connection: there is no port to pick, nothing
else on the machine can reach the server, and several clients can each run their own copy. This is
the mode MCPB bundles and Claude Desktop use; Claude Code and MCP Inspector support it as well. The
mode is available from server versions 0.5.0 (ProjectMaker, ZennoPoster, Android) and 0.3.0 (Instance).

Only JSON-RPC goes to stdout; every log line goes to stderr. The key is read from the configuration as
usual, which for a subprocess means the environment variable `<Section>__ApiKey` (`ProjectMaker__ApiKey`,
`Instance__ApiKey`, `ZennoPoster__ApiKey`, `Android__ApiKey`); the `--<Section>:ApiKey=` argument and
`appsettings.json` next to the exe work too. The Instance and Android copies take their `Target` and
`BaseUrl` pair the same way (`Instance__Target`, `Instance__BaseUrl`, and so on).

```powershell
# Claude Code: -e stores the value in the server's own config entry, not in the launch command
# line - without it the key only reaches the server in this one session, from this one shell.
$env:ProjectMaker__ApiKey = "<your key>"
claude mcp add projectmaker --transport stdio -e "ProjectMaker__ApiKey=$env:ProjectMaker__ApiKey" -- "C:\mcp\ZennoLab.AI.MCP.ProjectMaker.exe" --stdio
```

The same entries for a client that reads a `mcpServers` configuration (Claude Desktop, Cursor):

```json
{
  "mcpServers": {
    "projectmaker": {
      "command": "C:\\mcp\\ZennoLab.AI.MCP.ProjectMaker.exe",
      "args": ["--stdio"],
      "env": { "ProjectMaker__ApiKey": "<your key>" }
    },
    "instance-zp": {
      "command": "C:\\mcp\\ZennoLab.AI.MCP.Instance.exe",
      "args": ["--stdio"],
      "env": {
        "Instance__ApiKey": "<your key>",
        "Instance__Target": "zennoposter",
        "Instance__BaseUrl": "http://localhost:5300/api/v1"
      }
    }
  }
}
```

The process ends when the client closes its stdin. Without `--stdio` nothing changes: the server
listens on its port as described above, and the embedded AI chat of the products keeps using HTTP.

## Claude Desktop: install from an .mcpb bundle

Every release of a server also carries its MCPB bundles, one per client name from the tables above,
named `MCP.<server>-<client name>-v<version>-win-x64.mcpb` (for example
`MCP.Instance-instance-zp-v0.3.0-win-x64.mcpb`). A bundle is the same exe as in the zip, already set
up for stdio with the right `Target` and `BaseUrl`: Claude Desktop starts it itself and asks you
only for the key.

| Bundle | Server | Works with |
|---|---|---|
| `projectmaker` | MCP.ProjectMaker | ZennoPoster ProjectMaker, `:5299` |
| `instance-pm` | MCP.Instance | the browser of ZennoPoster ProjectMaker, `:5299` |
| `instance-zp` | MCP.Instance | the browsers of running ZennoPoster tasks, `:5300` |
| `zennoposter` | MCP.ZennoPoster | ZennoPoster tasks, `:5300` |
| `projectmaker-droid` | MCP.ProjectMaker | ZennoDroid ProjectMaker, `:5309` |
| `zennodroid` | MCP.ZennoPoster | ZennoDroid tasks, `:5310` |
| `android-pm` | MCP.Android | the device attached to ZennoDroid ProjectMaker, `:5309` |
| `android-zd` | MCP.Android | the devices of running ZennoDroid tasks, `:5310` |

1. Download the bundle you need from the release of its server.
2. In Claude Desktop open **Settings → Extensions** and drag the `.mcpb` file into the window.
3. Enter the ApiKey issued in step 2. The field is required: without a key the installation does
   not finish. Claude Desktop keeps the key with its own settings; the bundle itself holds no key.
4. Start the product the bundle works with. The tools of the server appear in the chat.

To replace the key, open the extension in **Settings → Extensions** and change it there. A bundle
listens on no port, so it never collides with a copy of the same server you started from the zip.

## 4. Configure your LLM client

A ready-made configuration fragment in the format of VS Code's `.vscode/mcp.json`. The same
entries for Cursor, Claude Code and other clients, and one-click install buttons, are on the
[install page](https://zennolab.github.io/zennoposter-mcp/install.html).

**ZennoPoster**

```json
{
  "servers": {
    "projectmaker": {
      "type": "http",
      "url": "http://localhost:6207"
    },
    "instance-pm": {
      "type": "http",
      "url": "http://localhost:6208"
    },
    "instance-zp": {
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

**ZennoDroid** — the names differ from the ZennoPoster ones, so both products can be configured in
one client:

```json
{
  "servers": {
    "projectmaker-droid": {
      "type": "http",
      "url": "http://localhost:6217"
    },
    "zennodroid": {
      "type": "http",
      "url": "http://localhost:6220"
    },
    "android-pm": {
      "type": "http",
      "url": "http://localhost:6211"
    },
    "android-zd": {
      "type": "http",
      "url": "http://localhost:6212"
    }
  }
}
```

No authorization is needed on this leg: the MCP servers listen on loopback only, and the
permissions are defined by the key the server itself was started with (step 3).

## 5. Verify the connection

List the server's tools in your MCP client, then call a read-only tool that needs the key:

| Server | Tool |
|---|---|
| `MCP.ProjectMaker` | `get_product_version` |
| `MCP.ZennoPoster` | `tasks_list` |
| `MCP.Instance` | `get_all_tabs`, with a browser open |
| `MCP.Android` | `list_devices` |

`ping` answers without checking the key, so it only shows that the server is running.
`capabilities` on `MCP.ProjectMaker` and `MCP.ZennoPoster` shows what the key may do — its
`currentScopes` and `currentMaxTier`; an empty `currentScopes` means the key was not accepted or
grants no scope.

If the key is invalid or lacks a scope/tier, the call fails with a structured error (`unauthorized` /
`forbidden`, the latter with `required`/`current` fields). `MCP.Android` returns it since 0.4.0;
earlier versions report only that the call failed.

## Identifying your client: the `X-ZP-Client` header

Every HTTP call to the PublicApi can carry the `X-ZP-Client` header to identify the calling
client. The value uses `name/version` format:

```
X-ZP-Client: mcp-zennoposter/0.5.0
```

The four official MCP sidecars each set it automatically (once ZP-6170 ships):

| Sidecar | Value |
|---|---|
| `MCP.ProjectMaker` | `mcp-projectmaker/<version>` |
| `MCP.Instance` | `mcp-instance/<version>` |
| `MCP.ZennoPoster` | `mcp-zennoposter/<version>` |
| `MCP.Android` | `mcp-android/<version>` |

**For third-party integrators:** choose a stable lower-case identifier that names your product —
for example `acme-bot/1.0`. Avoid generic names (`client`, `test`, `my-app`) that clash with
other integrators. The header is free-form; there is no registration step.

Calls that arrive without the header (direct HTTP scripts, curl, custom clients that do not set
it) are attributed to the built-in client name **`direct-http`**.

The header is used for **aggregated, anonymized analytics only** — see
[data-collection.md](https://zennolab.github.io/zennoposter-mcp/data-collection.html) for
exactly what is and is not collected. It has no effect on routing, authorization or rate limiting.

## Changing ports

Each server has two ports: the one it listens on for your LLM client, and the product API port it
calls.

**Listen port.** Any of the three, arguments winning over environment, environment over the file:

```powershell
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:7207

$env:ASPNETCORE_URLS = "http://localhost:7207"
.\ZennoLab.AI.MCP.ProjectMaker.exe
```

or `"Urls": "http://localhost:7207"` in the `appsettings.json` next to the exe. After moving a
listen port, update the matching URL in the client configuration from step 4.

Stay off **6107–6113** (**6117–6123** on ZennoDroid): those belong to the product's built-in servers,
and a foreign process on one of them prevents the built-in server from starting.

**Product API port.** Set `BaseUrl` in that server's own section — an argument such as
`--ProjectMaker:BaseUrl=http://localhost:<port>/api/v1`, an environment variable such as
`ProjectMaker__BaseUrl`, or `BaseUrl` in the `appsettings.json` next to the exe. The ZennoDroid
commands in step 3 do exactly this.

The section is named after the server: `ProjectMaker`, `Instance`, `ZennoPoster`, `Android`.
Versions up to 0.3.0 of `MCP.ProjectMaker` and `MCP.ZennoPoster` used `NeuroBot` and `ZennoPosterApi`;
those names still work in later versions and the server logs a warning at startup.

## License

The files in this repository (documentation and the OpenAPI specification) are licensed under the
[MIT License](https://github.com/ZennoLab/zennoposter-mcp/blob/main/LICENSE). The MCP server
binaries on the Releases page are proprietary; their terms are in
[TERMS.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/TERMS.md).
