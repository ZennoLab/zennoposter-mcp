[English](../index.md) · **Русский**

# ZennoLab PublicApi — документация для разработчиков

Локальный HTTP-API ProjectMaker и ZennoPoster: что он предоставляет, как выпускается и ограничивается ключ, что означают ошибки и как развивается контракт. API доступен только с loopback — см. модель безопасности.

Русский перевод — вспомогательный; при расхождении верной считается [английская версия](../index.md).

Этот сайт описывает ZennoPoster **7.9.2 и новее** и ZennoDroid **2.6.1 и новее**, версию контракта **1.1.0**, MCP-серверы **0.2.0**. Какой сервер к какому продукту и как узнать версии у своей установки — в [compatibility.md](compatibility.md).

- **[developer-guide.md](developer-guide.md)** — начните здесь: быстрый старт, выпуск ключа, скоупы/тиры, примеры запросов по доменам, `/capabilities`.
- **[versioning-and-deprecation.md](versioning-and-deprecation.md)** — политика semver (что считается breaking change), окно устаревания и контракт заголовков `Deprecation`/`Sunset`.
- **[compatibility.md](compatibility.md)** — какая версия продукта, версия контракта и версия MCP-серверов сочетаются между собой и как узнать все три у работающей установки.
- **[errors.md](errors.md)** — каждый HTTP-статус и код ошибки, который возвращает этот API, и как его обрабатывать.
- **[security-model.md](security-model.md)** — что защищает `ApiKey`, а что нет, доступ только с localhost, аудит.

Только на английском:

- **[api-reference.md](../api-reference.md)** — полный справочник операций: каждый метод по доменам, с глаголом, путём, уровнем риска, требуемым скоупом, полями тела запроса и кодами ответа.
- **[openapi/openapi.v1.json](../openapi/openapi.v1.json)** — машиночитаемый документ OpenAPI 3.0.3; **[openapi/index.html](../openapi/index.html)** отображает его через Redoc.

Подключение LLM-клиента к MCP-серверам описано отдельно — в [README репозитория](https://github.com/ZennoLab/zennoposter-mcp).

<!-- translated-from: index.md ed58c76745b20258076217f7b5d839e224f7d282 -->
