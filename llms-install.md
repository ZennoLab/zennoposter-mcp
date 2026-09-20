# Installation guide for AI agents

This file tells an AI assistant how to install and connect the ZennoPoster / ZennoDroid MCP
servers. Read it fully before acting. Steps that only a human can perform are marked.

This is the short path. The full picture — why the product runs its own internal MCP servers,
how the two `MCP.Instance` copies differ, every configuration override — is in the
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

Install only what the task requires. If unsure, start with **MCP.ProjectMaker**.

| Task | Server | Port | Works with |
|---|---|---|---|
| Read and edit a project: actions, connections, variables, lists, tables | MCP.ProjectMaker | 6207 | both products |
| Drive the browser opened inside ProjectMaker | MCP.Instance | 6208 | ZennoPoster only |
| Drive the browser inside ZennoPoster tasks | MCP.Instance (second copy) | 6209 | ZennoPoster only |
| Manage tasks: run, threads, stop, logs | MCP.ZennoPoster | 6210 | both products |
| Control the device attached to ProjectMaker | MCP.Android | 6211 | ZennoDroid only |
| Control the devices of running tasks | MCP.Android (second copy) | 6212 | ZennoDroid only |

On ZennoDroid the two servers that serve both products listen `+10` higher - MCP.ProjectMaker on
**6217** and MCP.ZennoPoster on **6220** - so a machine with both products can run both sets at once.
MCP.Android keeps 6211; nothing on ZennoPoster uses that port.

Do not bind anything to ports **6107-6113**: that band belongs to the product's own internal MCP
infrastructure, and a foreign process there stops the built-in AI chat from starting.

## 2. Ask the user to issue an ApiKey — human step

The key cannot be created from the command line. Ask the user to open ProjectMaker,
**Settings -> API Keys -> Add**, and then:

- set a `Label`, for example the name of the AI client being connected;
- set the `Max tier`: `T0` for read-only, higher to allow modifications;
- tick the `Scopes` needed; only `*:read` are ticked by default;
- copy the key immediately — it is shown once and cannot be retrieved later.

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

Run from the folder with the unpacked server, substituting the user's key for `zp_xxx`.

```powershell
# ProjectMaker, port 6207
.\ZennoLab.AI.MCP.ProjectMaker.exe --ProjectMaker:ApiKey=zp_xxx

# Browser in ProjectMaker, port 6208
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# ZennoPoster tasks, port 6210
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPoster:ApiKey=zp_xxx

# Android device attached to ProjectMaker, port 6211
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx
```

The devices of running tasks are a second copy of the Android server, exactly as the browser inside
tasks is a second copy of the Instance server:

```powershell
.\ZennoLab.AI.MCP.Android.exe --urls http://localhost:6212 `
  --Android:BaseUrl=http://localhost:5310/api/v1 --Android:ApiKey=zp_xxx
```

The browser inside ZennoPoster tasks (port 6209) is a second copy of the Instance server,
unpacked into a separate folder and started with `Target` and `BaseUrl` set together as a pair:

```powershell
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx
```

**On ZennoDroid everything shifts by +10**: the product API (ProjectMaker 5309, ZennoPoster 5310)
and the two servers that serve both products. The commands above are the ZennoPoster ones; on
ZennoDroid start these instead, with both the listen port and the `BaseUrl` set explicitly:

```powershell
# ProjectMaker on ZennoDroid: 6217 -> :5309
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:6217 `
  --ProjectMaker:BaseUrl=http://localhost:5309/api/v1 --ProjectMaker:ApiKey=zp_xxx

# ZennoPoster tasks on ZennoDroid: 6220 -> :5310
.\ZennoLab.AI.MCP.ZennoPoster.exe --urls http://localhost:6220 `
  --ZennoPoster:BaseUrl=http://localhost:5310/api/v1 --ZennoPoster:ApiKey=zp_xxx

# Android: no shift needed, ZennoDroid-only server
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx
```

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

Claude Code:

```sh
claude mcp add --transport http projectmaker http://localhost:6207
```

Cline, Cursor, VS Code, GitHub Copilot and other clients that use `mcp.json`:

```json
{
  "servers": {
    "projectmaker": { "type": "http", "url": "http://localhost:6207" },
    "zennoposter": { "type": "http", "url": "http://localhost:6210" }
  }
}
```

On ZennoDroid, with the shifted ports and names of their own, so both products can be configured
in one client:

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

LM Studio uses a different key name and needs version 0.3.17 or newer:

```json
{
  "mcpServers": {
    "projectmaker": { "url": "http://localhost:6207" }
  }
}
```

Restart the client after editing the configuration: it is read at startup. One-click install
buttons for Cursor and VS Code: https://zennolab.github.io/zennoposter-mcp/install.html

## 6. Verify

List the available tools, or call a safe read-only one such as `ping` / `get_product_version`.
A working ProjectMaker connection also exposes `get_project_structure`. If no tools appear:

- the server process is not running, or
- the configuration was written for a different client, or
- the client was not restarted.

Errors come back structured, not as silent failures:

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
