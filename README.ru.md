[English](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.md) | **Русский**

# Подключение MCP-серверов ZennoPoster/ProjectMaker к своему LLM-клиенту

MCP-серверы публикуются как self-contained `win-x64` бинарники через GitHub Releases в
общем репозитории **https://github.com/ZennoLab/zennoposter-mcp/releases**. У каждого
сервера своя линия релизов (свой префикс тега):

- **ProjectMaker (editor)** — теги `mcp-projectmaker-v*`, архив `MCP.ProjectMaker-v*-win-x64.zip`
- **Instance (browser control, dual-mount)** — теги `mcp-instance-v*`, архив `MCP.Instance-v*-win-x64.zip`
- **ZennoPoster (task runner)** — теги `mcp-zennoposter-v*`, архив `MCP.ZennoPoster-v*-win-x64.zip`
- **Android (ZennoDroid device)** — теги `mcp-android-v*`, архив `MCP.Android-v*-win-x64.zip`

Документация PublicApi (контракт OpenAPI в Redoc, гайд интегратора, коды ошибок, политика
версий) — **https://zennolab.github.io/zennoposter-mcp/**
(страницы публикуются из папки `docs/` этого репозитория).

Эти серверы работают с **ZennoPoster 7.9.2 и новее** и с **ZennoDroid 2.6.1 и новее**.
`MCP.Instance` применяется только к ZennoPoster, а `MCP.Android` — только к ZennoDroid;
`MCP.ProjectMaker` и `MCP.ZennoPoster` применяются к обоим.
Какая версия сервера сочетается с какой версией продукта и контракта:
**https://zennolab.github.io/zennoposter-mcp/ru/compatibility.html**.

## Модель: свой экземпляр MCP со своим ключом

Продукт **сам** поднимает внутренние MCP-серверы для своего AI-чата — они живут на
внутреннем бэнде портов **6107–6113** (на ZennoDroid всё сдвинуто `+10`), получают от хоста
служебный least-privilege ключ и **игнорируют** `Authorization` входящих запросов. Это внутренняя инфраструктура: внешнее подключение к ней не предполагается
(права там определяются служебным ключом, а не вашим), а **занимать её порты нельзя** — чужой
процесс на порту из этого бэнда не даст стартовать встроенному серверу (продукт напишет об
этом ошибку в лог, но AI-стек останется без этого сервера).

Для своего LLM-клиента вы запускаете **отдельную копию** MCP-сервера из публичного пакета:
она слушает на публичном бэнде **6207–6211**, ходит в тот же PublicApi продукта
(`:5299` ProjectMaker / `:5300` ZennoPoster), но уже с **вашим** ApiKey — с теми scope/tier,
которые вы выбрали при выпуске ключа. Обе копии работают одновременно и не мешают друг другу.

Порты по умолчанию (заданы в `appsettings.json` рядом с exe):

| Сервер | Порт | Ходит в | Ключ в конфиге |
|---|---|---|---|
| `MCP.ProjectMaker` | **6207** | PM PublicApi `:5299` | `NeuroBot:ApiKey` |
| `MCP.Instance` (Target=projectmaker) | **6208** | PM PublicApi `:5299` | `Instance:ApiKey` |
| `MCP.Instance` (Target=zennoposter) | **6209** (конвенция, задать явно) | ZP PublicApi `:5300` | `Instance:ApiKey` |
| `MCP.ZennoPoster` | **6210** | ZP PublicApi `:5300` | `ZennoPosterApi:ApiKey` |
| `MCP.Android` (ZennoDroid) | **6211** | ZDroid PublicApi `:5309` | `Android:ApiKey` |

API продукта, в который ходят эти серверы, слушает в двух продуктах на разных портах:

| Приложение | ZennoPoster | ZennoDroid |
|---|---|---|
| ProjectMaker | 5299 | 5309 |
| ZennoPoster | 5300 | 5310 |

Значения по умолчанию выше и примеры ниже — для ZennoPoster; на ZennoDroid задайте каждому серверу
`BaseUrl` из столбца ZennoDroid, как показано в разделе «Изменить порты».

Всё переопределяется штатной конфигурацией ASP.NET Core: `appsettings.json` рядом с exe,
переменные окружения (`ASPNETCORE_URLS`, `NeuroBot__ApiKey`, …) или аргументы командной
строки (`--urls`, `--NeuroBot:ApiKey=…`, …) — аргументы сильнее окружения, окружение
сильнее `appsettings.json`.

## 1. Скачать нужный сервер

На странице релизов найдите последний релиз нужного сервера (по префиксу тега), скачайте его
`*-win-x64.zip` и распакуйте в любую папку.
Каждый архив — один self-contained `.exe` + `appsettings.json`, дополнительный .NET runtime
не требуется.

## 2. Выпустить ApiKey

В ProjectMaker: **Settings → API Keys** → **Add** — откроется диалог "Add API key":
1. Задайте `Label` (произвольное имя ключа, для отличия в списке).
2. Выберите `Max tier` (минимально достаточный для ваших задач — T0 для read-only, выше для
   изменяющих операций).
3. Отметьте нужные `Scopes` (по умолчанию отмечены только `*:read`; остальные — по необходимости).
4. Подтвердите — сырой ключ показывается **один раз**, сразу скопируйте его в надёжное место.

Подробнее про scopes/tiers — [security-model.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/docs/ru/security-model.md).

## 3. Запустить сервер со своим ключом

Ключ задаётся **в конфигурации самого MCP-сервера** (не в заголовках MCP-клиента — сервер
принимает соединения только с loopback и не читает `Authorization` входящих запросов).
Проще всего — вписать ключ в `ApiKey` в `appsettings.json` рядом с exe и запустить без
аргументов; или передать через окружение/аргументы:

```powershell
# ProjectMaker (редактор): 6207 -> :5299
.\ZennoLab.AI.MCP.ProjectMaker.exe --NeuroBot:ApiKey=zp_xxx

# Instance для редактора (браузер PM): 6208 -> :5299
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# Instance для раннера — ВТОРАЯ копия того же exe: порт и пара Target/BaseUrl задаются явно
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx

# ZennoPoster (задачи/сессии раннера): 6210 -> :5300
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPosterApi:ApiKey=zp_xxx

# Android (устройство ZennoDroid): 6211 -> :5309
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx
```

**Важно про два экземпляра `MCP.Instance`**: у сервера Instance `Target` (какие инструкции
для ИИ он отдаёт — про ProjectMaker или про ZennoPoster) и `BaseUrl` (куда реально уходят
HTTP-запросы) — задаются только вместе, как пара, и ничем не связаны в коде
(`Target=projectmaker` → `BaseUrl` на PM PublicApi `:5299`, `Target=zennoposter` → `BaseUrl`
на ZP PublicApi `:5300`). При старте сервер делает best-effort проверку через `/capabilities`
целевого хоста и пишет предупреждение в лог при несовпадении, но если целевой хост в момент
старта недоступен, проверка молча пропускается — рассинхронизация в этом случае не
детектируется, и ИИ получит инструкции про один хост, а запросы будут уходить на другой.

## 4. Настроить LLM-клиент

Готовый фрагмент конфигурации (формат — как в `.mcp.json` Claude Code, используйте
эквивалент для вашего MCP-клиента при необходимости):

```json
{
  "servers": {
    "projectmaker": {
      "type": "http",
      "url": "http://localhost:6207"
    },
    "instance-projectmaker": {
      "type": "http",
      "url": "http://localhost:6208"
    },
    "instance-zennoposter": {
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

На ZennoDroid записи будут `projectmaker`, `zennoposter` и `android` (порт 6211); монтирования
`instance-*` там нет.

Авторизация на этом плече не нужна: MCP-серверы слушают только loopback, а права
определяются ключом, с которым запущен сам сервер (шаг 3).

## 5. Проверить подключение

Любым MCP-клиентом (или напрямую HTTP) вызовите безопасный read-only метод и убедитесь, что он
отвечает `200 OK` с ожидаемыми данными, например `get_product_version`/`ping` на нужном сервере.
Если ключ невалиден или не хватает scope/tier — сервер вернёт структурированную ошибку
(`401 unauthorized` / `403 forbidden` с полями `required`/`current`), а не молчаливый сбой.

## Изменить порты

У каждого сервера два порта: тот, на котором он слушает ваш LLM-клиент, и порт API продукта, в
который он ходит.

**Порт прослушивания.** Любым из трёх способов, аргументы сильнее окружения, окружение сильнее
файла:

```powershell
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:7207

$env:ASPNETCORE_URLS = "http://localhost:7207"
.\ZennoLab.AI.MCP.ProjectMaker.exe
```

или `"Urls": "http://localhost:7207"` в `appsettings.json` рядом с exe. После смены порта поправьте
соответствующий URL в конфигурации клиента из шага 4.

Не занимайте **6107–6113**: это порты встроенных серверов продукта, чужой процесс на любом из них
не даст встроенному серверу стартовать.

**Порт API продукта.** Задаётся в `BaseUrl` секции этого сервера — именно это меняется на
ZennoDroid:

```powershell
# API ProjectMaker на 5309 вместо 5299
.\ZennoLab.AI.MCP.ProjectMaker.exe --NeuroBot:BaseUrl=http://localhost:5309/api/v1 --NeuroBot:ApiKey=zp_xxx

# API ZennoPoster на 5310 вместо 5300
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPosterApi:BaseUrl=http://localhost:5310/api/v1 --ZennoPosterApi:ApiKey=zp_xxx
```

Имя секции у каждого сервера своё: `NeuroBot` у `MCP.ProjectMaker`, `Instance` у `MCP.Instance`,
`ZennoPosterApi` у `MCP.ZennoPoster`, `Android` у `MCP.Android`.
