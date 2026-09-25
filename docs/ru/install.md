[English](../install.md) · **Русский**

# Установка MCP-серверов

Нужен ZennoPoster 7.9.2 или новее либо ZennoDroid 2.6.1 или новее, Windows. Клиент подключается к запущенному MCP-серверу: скачайте его, выпустите ApiKey и запустите, как описано в [README репозитория](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.ru.md).

Команды и кнопки ниже используют порты из таблиц README. Если сервер запущен на другом порту, укажите этот порт в конфигурации клиента.

## Cursor и VS Code

ZennoPoster:

| Сервер | Имя, порт | Cursor | VS Code |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker`, 6207 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=projectmaker&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDcifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=projectmaker&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6207%22%7D) |
| MCP.Instance, цель ProjectMaker | `instance-pm`, 6208 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDgifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6208%22%7D) |
| MCP.Instance, цель ZennoPoster | `instance-zp`, 6209 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=instance-zp&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMDkifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=instance-zp&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6209%22%7D) |
| MCP.ZennoPoster | `zennoposter`, 6210 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=zennoposter&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTAifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=zennoposter&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6210%22%7D) |

ZennoDroid:

| Сервер | Имя, порт | Cursor | VS Code |
|---|---|---|---|
| MCP.ProjectMaker | `projectmaker-droid`, 6217 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=projectmaker-droid&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTcifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=projectmaker-droid&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6217%22%7D) |
| MCP.ZennoPoster | `zennodroid`, 6220 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=zennodroid&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMjAifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=zennodroid&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6220%22%7D) |
| MCP.Android, цель ProjectMaker | `android-pm`, 6211 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=android-pm&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTEifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=android-pm&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6211%22%7D) |
| MCP.Android, цель ZennoPoster | `android-zd`, 6212 | [![Add to Cursor](https://img.shields.io/badge/Cursor-Add_server-000000?style=for-the-badge&logo=cursor&logoColor=white)](https://cursor.com/en/install-mcp?name=android-zd&config=eyJ1cmwiOiJodHRwOi8vbG9jYWxob3N0OjYyMTIifQ%3D%3D) | [![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_server-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=android-zd&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22http%3A%2F%2Flocalhost%3A6212%22%7D) |

MCP.ProjectMaker и MCP.ZennoPoster обслуживают оба продукта; на ZennoDroid они слушают порты, сдвинутые на +10, и носят собственные имена, чтобы оба продукта можно было настроить в одном клиенте. MCP.Instance есть только на ZennoPoster, MCP.Android — только на ZennoDroid.

## Claude Code

Выполните строку для каждого сервера, который используете:

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

Без `--scope` сервер добавляется только для текущего проекта; с `--scope user` он доступен во всех проектах.

## Файлы конфигурации

Те же серверы, записанные вручную, — отдельный блок на каждый продукт. Оставьте только те записи, которые запускаете; если на машине оба продукта, записи обоих блоков помещаются в один файл.

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

Claude Code, `.mcp.json` в корне проекта:

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

Claude Code, `.mcp.json` в корне проекта:

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

Другие MCP-клиенты: добавьте сервер Streamable HTTP с URL из таблиц выше (Cline нужен `"type": "streamableHttp"` в записи). Заголовок авторизации не нужен.

<!-- translated-from: install.md eb1452bb3f2a1256c1f293eaca89e1d976a00b75 -->
