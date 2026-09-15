**English** · [Русский](ru/install.md)

# Install MCP servers

Requires ZennoPoster 7.9.2 or newer, or ZennoDroid 2.6.1 or newer, on Windows. The MCP server must
be running before you add it to a client: download it, issue an ApiKey and start the server as
described in the [repository README](https://github.com/ZennoLab/zennoposter-mcp#readme).

| Server | Name | Port | Product |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker` | 6207 | ZennoPoster, ZennoDroid |
| MCP.Instance, target ProjectMaker | `instance-pm` | 6208 | ZennoPoster |
| MCP.Instance, target ZennoPoster | `instance-zp` | 6209 | ZennoPoster |
| MCP.ZennoPoster | `zennoposter` | 6210 | ZennoPoster, ZennoDroid |
| MCP.Android | `android` | 6211 | ZennoDroid |

The commands and buttons below use the default ports. If you started a server on another port, use
that port in the client configuration.

## ProjectMaker

```shell
claude mcp add --transport http projectmaker http://localhost:6207
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=projectmaker&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDcifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=projectmaker&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6207%22%7D)

## Instance, target ProjectMaker

```shell
claude mcp add --transport http instance-pm http://localhost:6208
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=instance-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDgifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=instance-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6208%22%7D)

## Instance, target ZennoPoster

```shell
claude mcp add --transport http instance-zp http://localhost:6209
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=instance-zp&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDkifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=instance-zp&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6209%22%7D)

## ZennoPoster

```shell
claude mcp add --transport http zennoposter http://localhost:6210
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=zennoposter&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTAifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=zennoposter&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6210%22%7D)

## Android (ZennoDroid)

```shell
claude mcp add --transport http android http://localhost:6211
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=android&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTEifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=android&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6211%22%7D)

## Configuration files

The same servers written by hand. Keep only the entries you run.

VS Code, `.vscode/mcp.json`:

```json
{
  "servers": {
    "projectmaker": { "type": "http", "url": "http://localhost:6207" },
    "instance-pm": { "type": "http", "url": "http://localhost:6208" },
    "instance-zp": { "type": "http", "url": "http://localhost:6209" },
    "zennoposter": { "type": "http", "url": "http://localhost:6210" },
    "android": { "type": "http", "url": "http://localhost:6211" }
  }
}
```

Cursor, `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "projectmaker": { "url": "http://localhost:6207" },
    "instance-pm": { "url": "http://localhost:6208" },
    "instance-zp": { "url": "http://localhost:6209" },
    "zennoposter": { "url": "http://localhost:6210" },
    "android": { "url": "http://localhost:6211" }
  }
}
```

Claude Code, `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "projectmaker": { "type": "http", "url": "http://localhost:6207" },
    "instance-pm": { "type": "http", "url": "http://localhost:6208" },
    "instance-zp": { "type": "http", "url": "http://localhost:6209" },
    "zennoposter": { "type": "http", "url": "http://localhost:6210" },
    "android": { "type": "http", "url": "http://localhost:6211" }
  }
}
```

Other MCP clients: add a Streamable HTTP server with the URL from the table above. No authorization
header is needed.
