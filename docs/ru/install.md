[English](../install.md) · **Русский**

# Установка MCP-серверов

Нужен ZennoPoster 7.9.2 или новее либо ZennoDroid 2.6.1 или новее, Windows. MCP-сервер должен быть запущен до добавления в клиент: скачайте его, выпустите ApiKey и запустите сервер, как описано в [README репозитория](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.ru.md).

| Сервер | Имя | Порт | Продукт |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker` | 6207 | ZennoPoster, ZennoDroid |
| MCP.Instance, цель ProjectMaker | `instance-pm` | 6208 | ZennoPoster |
| MCP.Instance, цель ZennoPoster | `instance-zp` | 6209 | ZennoPoster |
| MCP.ZennoPoster | `zennoposter` | 6210 | ZennoPoster, ZennoDroid |
| MCP.Android | `android` | 6211 | ZennoDroid |

Команды и кнопки ниже используют порты по умолчанию. Если сервер запущен на другом порту, укажите этот порт в конфигурации клиента.

## ProjectMaker

```shell
claude mcp add --transport http projectmaker http://localhost:6207
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=projectmaker&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDcifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=projectmaker&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6207%22%7D)

## Instance, цель ProjectMaker

```shell
claude mcp add --transport http instance-pm http://localhost:6208
```

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=instance-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDgifQ%3D%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=instance-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6208%22%7D)

## Instance, цель ZennoPoster

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

## Файлы конфигурации

Те же серверы, записанные вручную. Оставьте только те записи, которые запускаете.

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

Claude Code, `.mcp.json` в корне проекта:

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

Другие MCP-клиенты: добавьте сервер Streamable HTTP с URL из таблицы выше. Заголовок авторизации не нужен.

<!-- translated-from: install.md 94a2f21bd2591f3db3b162ff880eaa3034c4b110 -->
