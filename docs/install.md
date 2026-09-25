**English** · [Русский](ru/install.md)

# Install MCP servers

Requires ZennoPoster 7.9.2 or newer, or ZennoDroid 2.6.1 or newer, on Windows. The client connects to
a running MCP server: download it, issue an ApiKey and start it as described in the
[repository README](https://github.com/ZennoLab/zennoposter-mcp#readme).

The commands and buttons below use the ports from the README tables. If you started a server on
another port, use that port in the client configuration.

## Cursor and VS Code

ZennoPoster:

| Server | Name, port | Cursor | VS Code |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker`, 6207 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=projectmaker&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDcifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=projectmaker&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6207%22%7D) |
| MCP.Instance, target ProjectMaker | `instance-pm`, 6208 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDgifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6208%22%7D) |
| MCP.Instance, target ZennoPoster | `instance-zp`, 6209 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-zp&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDkifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-zp&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6209%22%7D) |
| MCP.ZennoPoster | `zennoposter`, 6210 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=zennoposter&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTAifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=zennoposter&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6210%22%7D) |

ZennoDroid:

| Server | Name, port | Cursor | VS Code |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker-droid`, 6217 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=projectmaker-droid&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTcifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=projectmaker-droid&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6217%22%7D) |
| MCP.ZennoPoster | `zennodroid`, 6220 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=zennodroid&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMjAifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=zennodroid&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6220%22%7D) |
| MCP.Android, target ProjectMaker | `android-pm`, 6211 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=android-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTEifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=android-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6211%22%7D) |
| MCP.Android, target ZennoPoster | `android-zd`, 6212 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=android-zd&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTIifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=android-zd&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6212%22%7D) |

MCP.ProjectMaker and MCP.ZennoPoster serve both products; on ZennoDroid they listen on ports shifted by
+10 and go under names of their own, so both products can be configured in one client. MCP.Instance
exists on ZennoPoster only, MCP.Android on ZennoDroid only.

## Claude Code

Run the line for each server you use:

```shell
# ZennoPoster
claude mcp add --transport http projectmaker http://localhost:6207
claude mcp add --transport http instance-pm http://localhost:6208
claude mcp add --transport http instance-zp http://localhost:6209
claude mcp add --transport http zennoposter http://localhost:6210

# ZennoDroid
claude mcp add --transport http projectmaker-droid http://localhost:6217
claude mcp add --transport http zennodroid http://localhost:6220
claude mcp add --transport http android-pm http://localhost:6211
claude mcp add --transport http android-zd http://localhost:6212
```

Without `--scope`, a server is added for the current project only; add `--scope user` to make it
available in every project.

## Configuration files

The same servers written by hand, one block per product. Keep only the entries you run; with both
products on one machine, the entries of both blocks go into the same file.

### ZennoPoster

VS Code, `.vscode/mcp.json`:

```json
{
  "servers": {
    "projectmaker": { "type": "http", "url": "http://localhost:6207" },
    "instance-pm": { "type": "http", "url": "http://localhost:6208" },
    "instance-zp": { "type": "http", "url": "http://localhost:6209" },
    "zennoposter": { "type": "http", "url": "http://localhost:6210" }
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
    "zennoposter": { "url": "http://localhost:6210" }
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
    "zennoposter": { "type": "http", "url": "http://localhost:6210" }
  }
}
```

### ZennoDroid

VS Code, `.vscode/mcp.json`:

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

Cursor, `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "projectmaker-droid": { "url": "http://localhost:6217" },
    "zennodroid": { "url": "http://localhost:6220" },
    "android-pm": { "url": "http://localhost:6211" },
    "android-zd": { "url": "http://localhost:6212" }
  }
}
```

Claude Code, `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "projectmaker-droid": { "type": "http", "url": "http://localhost:6217" },
    "zennodroid": { "type": "http", "url": "http://localhost:6220" },
    "android-pm": { "type": "http", "url": "http://localhost:6211" },
    "android-zd": { "type": "http", "url": "http://localhost:6212" }
  }
}
```

Other MCP clients: add a Streamable HTTP server with the URL from the tables above (Cline needs
`"type": "streamableHttp"` in the entry). No authorization header is needed.
