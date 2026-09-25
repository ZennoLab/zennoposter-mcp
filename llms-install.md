# Installation guide for AI agents

This file tells an AI assistant how to install and connect the ZennoPoster / ZennoDroid MCP
servers. Read it fully before acting. Steps that only a human can perform are marked.

This is the short path. The full picture — why the product runs its own internal MCP servers,
how the two `MCP.Instance` and `MCP.Android` copies differ, every configuration override — is in the
[README](https://github.com/ZennoLab/zennoposter-mcp#readme).

## 0. Check the prerequisites first

Confirm all three with the user before downloading anything:

1. **Windows x64.** These are self-contained Windows binaries. They do not run on macOS, Linux
   or in a Linux container.
2. **ZennoPoster 7.9.2+ or ZennoDroid 2.6.1+.** Earlier versions have no PublicApi for the
   servers to talk to. The version is in the program title bar.
3. **The application is running.** The server talks to a live ProjectMaker / ZennoPoster /
   ZennoDroid, not to a project file on disk. For project operations the project must be open.

If any of these is not satisfied, stop and say what is missing. Do not work around it.

## 1. Pick the servers you need

Install only what the task requires. If unsure, start with **MCP.ProjectMaker**. The servers and
ports depend on the product found in step 0.

ZennoPoster:

| Task | Server | Port |
|---|---|---|
| Read and edit a project: actions, connections, variables, lists, tables | MCP.ProjectMaker | 6207 |
| Drive the browser opened inside ProjectMaker | MCP.Instance | 6208 |
| Drive the browser inside ZennoPoster tasks | MCP.Instance (second copy) | 6209 |
| Manage tasks: run, stop, threads, tries, settings | MCP.ZennoPoster | 6210 |

ZennoDroid:

| Task | Server | Port |
|---|---|---|
| Read and edit a project: actions, connections, variables, lists, tables | MCP.ProjectMaker | 6217 |
| Manage tasks: run, stop, threads, tries, settings | MCP.ZennoPoster | 6220 |
| Control the device attached to ProjectMaker | MCP.Android | 6211 |
| Control the devices of running tasks | MCP.Android (second copy) | 6212 |

The two sets use different ports, so a machine with both products can run both at once.

Do not bind anything to ports **6107-6113** (**6117-6123** on ZennoDroid): that band belongs to the
product's own internal MCP infrastructure, and a foreign process there stops the built-in AI chat
from starting.

## 2. Ask the user to issue an ApiKey — human step

The key cannot be created from the command line. Ask the user to open ProjectMaker,
**Settings -> Api-Keys -> Add new API key**, and then:

- set a `Label`, for example the name of the AI client being connected;
- set the `Max tier`: `T0` for read-only, higher to allow modifications;
- tick the `Scopes` needed; only `*:read` are ticked by default;
- press **Generate API Key** and copy the key immediately — it is shown once and cannot be
  retrieved later.

Recommend starting with a `T0` read-only key; once the connection works, the user can issue a
second key with write permissions. The key looks like `zp_...`. Never print it back in full and
never commit it. Scopes and tiers:
[security model](https://zennolab.github.io/zennoposter-mcp/security-model.html).

## 3. Download and unpack the server

Releases: https://github.com/ZennoLab/zennoposter-mcp/releases

Each server has its own release line, so take the latest release with the matching tag prefix:
`mcp-projectmaker-v*`, `mcp-instance-v*`, `mcp-zennoposter-v*`, `mcp-android-v*`. The asset is
`MCP.<Server>-v<version>-win-x64.zip`. Unpack each server into its own folder, for example
`C:\ZennoMCP\ProjectMaker\`. The archive is one self-contained `.exe` plus `appsettings.json`;
no .NET runtime installation is needed.

## 4. Start the server with the key

Run from the folder with the unpacked server, substituting the user's key for `zp_xxx`. Start only
the servers picked in step 1, from the block of the user's product.

ZennoPoster:

```powershell
# ProjectMaker, port 6207
.\ZennoLab.AI.MCP.ProjectMaker.exe --ProjectMaker:ApiKey=zp_xxx

# Browser in ProjectMaker, port 6208
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# Browser inside ZennoPoster tasks, port 6209: a second copy of the Instance server
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx

# ZennoPoster tasks, port 6210
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPoster:ApiKey=zp_xxx
```

ZennoDroid: its API listens on 5309 (ProjectMaker) and 5310 (the runner), and the two servers that
serve both products need the listen port and `BaseUrl` set explicitly:

```powershell
# ProjectMaker, port 6217
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:6217 `
  --ProjectMaker:BaseUrl=http://localhost:5309/api/v1 --ProjectMaker:ApiKey=zp_xxx

# ZennoDroid tasks, port 6220
.\ZennoLab.AI.MCP.ZennoPoster.exe --urls http://localhost:6220 `
  --ZennoPoster:BaseUrl=http://localhost:5310/api/v1 --ZennoPoster:ApiKey=zp_xxx

# Android device attached to ProjectMaker, port 6211
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx

# Devices of running tasks, port 6212: a second copy of the Android server
.\ZennoLab.AI.MCP.Android.exe --urls http://localhost:6212 `
  --Android:Target=zennoposter --Android:BaseUrl=http://localhost:5310/api/v1 `
  --Android:ApiKey=zp_xxx
```

A second copy is the same exe started once more, with `Target` and `BaseUrl` set together as a
pair. `Target` selects the instructions the server gives the assistant; for the Android server, one
editor device whose id may stay 0, or one device per task thread addressed by an id from
`list_devices`.

The config section is named after the server: `ProjectMaker`, `Instance`, `ZennoPoster`,
`Android`. Versions up to 0.3.0 of `MCP.ProjectMaker` and `MCP.ZennoPoster` used `NeuroBot` and
`ZennoPosterApi`; those names still work and the server logs a warning at startup.

The key can also be written into `ApiKey` in the `appsettings.json` next to the exe, or passed
through an environment variable such as `ProjectMaker__ApiKey`. Command-line arguments override
environment variables, which override `appsettings.json`.

The process must keep running while the assistant is connected. Tell the user to leave the
console window open, or to put the key into `appsettings.json` so the server can be started by
double-clicking it.

## 5. Configure the MCP client

Transport is **streamable HTTP on localhost**, not stdio. There is no npm package to run, and no
`Authorization` header is needed on this leg: the servers listen on loopback only and the
permissions come from the key the server itself was started with.

Claude Code, one command per server:

```sh
claude mcp add --transport http projectmaker http://localhost:6207
```

VS Code and GitHub Copilot read `servers` from `.vscode/mcp.json`:

```json
{
  "servers": {
    "projectmaker": { "type": "http", "url": "http://localhost:6207" },
    "zennoposter": { "type": "http", "url": "http://localhost:6210" }
  }
}
```

Cursor and LM Studio (0.3.17 or newer) read `mcpServers`, with a `url` per entry:

```json
{
  "mcpServers": {
    "projectmaker": { "url": "http://localhost:6207" },
    "zennoposter": { "url": "http://localhost:6210" }
  }
}
```

Cline reads `mcpServers` too and needs the transport named: without `"type": "streamableHttp"` it
falls back to the older SSE transport.

```json
{
  "mcpServers": {
    "projectmaker": { "type": "streamableHttp", "url": "http://localhost:6207" },
    "zennoposter": { "type": "streamableHttp", "url": "http://localhost:6210" }
  }
}
```

On ZennoDroid the entries have names of their own, so both products can be configured in one
client. In the VS Code format:

```json
{
  "servers": {
    "projectmaker-droid": { "type": "http", "url": "http://localhost:6217" },
    "zennodroid": { "type": "http", "url": "http://localhost:6220" },
    "android-pm": { "type": "http", "url": "http://localhost:6211" },
    "android-zd": { "type": "http", "url": "http://localhost:6212" }
  }
}
```

Restart the client after editing the configuration: it is read at startup. Every entry for VS Code,
Cursor and Claude Code, with one-click install buttons:
https://zennolab.github.io/zennoposter-mcp/install.html

## 6. Verify

List the available tools, then call a read-only one that needs the key: `get_product_version` on
MCP.ProjectMaker, `tasks_list` on MCP.ZennoPoster, `list_devices` on MCP.Android, `get_all_tabs` on
MCP.Instance with a browser open. `ping` answers without checking the key. A working ProjectMaker
connection also exposes `get_project_structure`. If no tools appear:

- the server process is not running, or
- the configuration was written for a different client, or
- the client was not restarted.

Errors come back structured, not as silent failures; MCP.Android does so since 0.4.0, earlier
versions report only that a call failed.

| Code | Meaning | Fix |
|---|---|---|
| `401` | Key missing or invalid | Check the key was passed in full; ask the user to issue a new one if lost |
| `403` | Key lacks a scope or tier | The body names `required` and `current`; ask the user for a key with those scopes |
| `409 project_not_open` | No project is open in ProjectMaker | Ask the user to open one, or call `open_project` |
| `409 failed_precondition` | The project's state refused the operation, e.g. closing a project with unsaved changes | Read the message: save first, or opt in explicitly (`discardUnsavedChanges` on `close_project`) |

Full list: https://zennolab.github.io/zennoposter-mcp/errors.html

## Notes for working with projects

- `get_project_structure` is tiered: no arguments gives a group overview, `groupId` drills into
  one group, `full=true` dumps the whole graph. Start with the overview — a full dump of a large
  template costs tens of thousands of tokens.
- `find_path(from, to)` and `get_action_connections(actionId)` answer routing questions without
  dumping the project; `get_action_details` reads one action's parameter values.
- `save_project` and `open_project` return `fileHash`, `fileSizeBytes` and `lastWriteTimeUtc`, so
  you can confirm what is actually on disk.
- Advise the user to keep a copy of the project before making edits.

## Reference

- Documentation: https://zennolab.github.io/zennoposter-mcp/
- OpenAPI contract, error codes, versioning policy: same site
- Version matrix (which server version works with which product):
  https://zennolab.github.io/zennoposter-mcp/compatibility.html
