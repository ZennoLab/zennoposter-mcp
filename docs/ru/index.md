[English](../index.md) · **Русский**

# ZennoLab PublicApi — документация для разработчиков

Локальный HTTP-API ProjectMaker и ZennoPoster: что он предоставляет, как выпускается и ограничивается ключ, что означают ошибки и как развивается контракт. API доступен только с loopback — см. модель безопасности.

Русский перевод — вспомогательный; при расхождении верной считается [английская версия](../index.md).

Этот сайт описывает ZennoPoster **7.9.3 и новее** и ZennoDroid **2.6.1 и новее**, версию контракта **1.4.0**, MCP-серверы **0.5.0** (MCP.ProjectMaker, MCP.ZennoPoster), **0.4.0** (MCP.Android) и **0.3.0** (MCP.Instance). Какой сервер к какому продукту и как узнать версии у своей установки — в [compatibility.md](compatibility.md).

- **[developer-guide.md](developer-guide.md)** — начните здесь: быстрый старт, выпуск ключа, скоупы/тиры, примеры запросов по доменам, `/capabilities`.
- **[versioning-and-deprecation.md](versioning-and-deprecation.md)** — политика semver (что считается breaking change), окно устаревания и контракт заголовков `Deprecation`/`Sunset`.
- **[compatibility.md](compatibility.md)** — какая версия продукта, версия контракта и версия MCP-серверов сочетаются между собой и как узнать все три у работающей установки.
- **[errors.md](errors.md)** — каждый HTTP-статус и код ошибки, который возвращает этот API, и как его обрабатывать.
- **[data-collection.md](data-collection.md)** — какая обезличенная телеметрия собирается по использованию PublicApi и выпуску ключей, что не собирается и как данные обрабатываются.
- **[security-model.md](security-model.md)** — что защищает `ApiKey`, а что нет, доступ только с localhost, аудит.

Только на английском:

- **[api-reference.md](../api-reference.md)** — полный справочник операций: каждый метод по доменам, с глаголом, путём, уровнем риска, требуемым скоупом, полями тела запроса и кодами ответа.
- **[openapi/openapi.v1.json](../openapi/openapi.v1.json)** — машиночитаемый документ OpenAPI 3.0.3; **[openapi/index.html](../openapi/index.html)** отображает его через Redoc.

Подключение LLM-клиента к MCP-серверам описано отдельно — в [README репозитория](https://github.com/ZennoLab/zennoposter-mcp). Кнопки установки для Cursor и VS Code и команды Claude Code — в **[install.md](install.md)**.

<!-- translated-from: index.md 40230fc0e88fa0275a17ece2da6d365c9ce8f83f -->
