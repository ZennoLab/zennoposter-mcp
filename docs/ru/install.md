[English](../install.md) · **Русский**

# Установка MCP-серверов

Нужен ZennoPoster 7.9.2 или новее либо ZennoDroid 2.6.1 или новее, Windows. MCP-сервер должен быть запущен до добавления в клиент: скачайте его, выпустите ApiKey и запустите сервер, как описано в [README репозитория](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.ru.md).

Команды и кнопки ниже используют порты по умолчанию. Если сервер запущен на другом порту, укажите этот порт в конфигурации клиента.

## Cursor и VS Code

| Сервер | Имя, порт | Cursor | VS Code |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker`, 6207 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=projectmaker&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDcifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=projectmaker&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6207%22%7D) |
| MCP.Instance, цель ProjectMaker | `instance-pm`, 6208 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDgifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6208%22%7D) |
| MCP.Instance, цель ZennoPoster | `instance-zp`, 6209 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-zp&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDkifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-zp&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6209%22%7D) |
| MCP.ZennoPoster | `zennoposter`, 6210 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=zennoposter&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTAifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=zennoposter&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6210%22%7D) |
| MCP.Android | `android`, 6211 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=android&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTEifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=android&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6211%22%7D) |

MCP.Instance работает только с ZennoPoster, MCP.Android — только с ZennoDroid; MCP.ProjectMaker и MCP.ZennoPoster — с обоими.

## Claude Code

Выполните строку для каждого сервера, который используете:

```shell
claude mcp add --transport http projectmaker http://localhost:6207
claude mcp add --transport http instance-pm http://localhost:6208
claude mcp add --transport http instance-zp http://localhost:6209
claude mcp add --transport http zennoposter http://localhost:6210
claude mcp add --transport http android http://localhost:6211
```

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

<!-- translated-from: install.md be17ec579f37e412bae8eb0d0ee9ffe4a2559d7e -->
