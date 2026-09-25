[English](https://github.com/ZennoLab/zennoposter-mcp/blob/main/README.md) | **Русский**

# Подключение MCP-серверов ZennoPoster и ZennoDroid к своему LLM-клиенту

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

Кнопки установки для Cursor и VS Code и команды Claude Code для каждого сервера:
**https://zennolab.github.io/zennoposter-mcp/ru/install.html**.

## Модель: свой экземпляр MCP со своим ключом

Продукт **сам** поднимает внутренние MCP-серверы для своего AI-чата — они живут на
внутреннем бэнде портов **6107–6113** (на ZennoDroid всё сдвинуто `+10`), получают от хоста
служебный least-privilege ключ и **игнорируют** `Authorization` входящих запросов. Это внутренняя инфраструктура: внешнее подключение к ней не поддерживается
(права там определяются служебным ключом, а не вашим), а **занимать её порты нельзя** — чужой
процесс на порту из этого бэнда не даст стартовать встроенному серверу (продукт напишет об
этом ошибку в лог, но AI-стек останется без этого сервера).

Для своего LLM-клиента вы запускаете **отдельную копию** MCP-сервера из публичного пакета:
она слушает публичный порт (таблицы ниже) и ходит в тот же PublicApi продукта, но уже с
**вашим** ApiKey — с теми scope/tier, которые вы выбрали при выпуске ключа. Обе копии работают
одновременно и не мешают друг другу.

На **ZennoDroid** два сервера, которые обслуживают оба продукта, слушают на `+10` выше:
`MCP.ProjectMaker` — **6217**, `MCP.ZennoPoster` — **6220**. Это тот же сдвиг, который продукт
применяет к своему внутреннему бэнду, поэтому на машине с обоими продуктами оба набора работают
одновременно. `MCP.Android` есть только на ZennoDroid и остаётся на **6211**: на ZennoPoster этот
порт никем не занят.

Порты по умолчанию — своя таблица на каждый продукт. Порт прослушивания и `BaseUrl` сервер берёт из
`appsettings.json` рядом со своим exe; у портов, помеченных ¹, таких значений нет — их задают явно в
командной строке вместе с парным `BaseUrl` (и `Target`), как в примерах шага 3.

**ZennoPoster**

| Сервер | Порт | Ходит в (PublicApi) | Ключ в конфиге |
|---|---|---|---|
| `MCP.ProjectMaker` | **6207** | ProjectMaker `:5299` | `ProjectMaker:ApiKey` |
| `MCP.Instance`, `Target=projectmaker` (браузер редактора) | **6208** | ProjectMaker `:5299` | `Instance:ApiKey` |
| `MCP.Instance`, `Target=zennoposter` (браузеры задач) | **6209** ¹ | ZennoPoster `:5300` | `Instance:ApiKey` |
| `MCP.ZennoPoster` | **6210** | ZennoPoster `:5300` | `ZennoPoster:ApiKey` |

**ZennoDroid**

| Сервер | Порт | Ходит в (PublicApi) | Ключ в конфиге |
|---|---|---|---|
| `MCP.ProjectMaker` | **6217** ¹ | ProjectMaker `:5309` | `ProjectMaker:ApiKey` |
| `MCP.ZennoPoster` | **6220** ¹ | ZennoPoster `:5310` | `ZennoPoster:ApiKey` |
| `MCP.Android`, `Target=projectmaker` (устройство редактора) | **6211** | ProjectMaker `:5309` | `Android:ApiKey` |
| `MCP.Android`, `Target=zennoposter` (устройства задач) | **6212** ¹ | ZennoPoster `:5310` | `Android:ApiKey` |

¹ Не значение по умолчанию — задаётся явно (`--urls` и парный `BaseUrl`; команды приведены в шаге 3).

Всё переопределяется штатной конфигурацией ASP.NET Core: `appsettings.json` рядом с exe,
переменные окружения (`ASPNETCORE_URLS`, `ProjectMaker__ApiKey`, …) или аргументы командной
строки (`--urls`, `--ProjectMaker:ApiKey=…`, …) — аргументы сильнее окружения, окружение
сильнее `appsettings.json`.

## 1. Скачать нужный сервер

На странице релизов найдите последний релиз нужного сервера (по префиксу тега), скачайте его
`*-win-x64.zip` и распакуйте в любую папку.
Каждый архив — один self-contained `.exe` + `appsettings.json`, дополнительный .NET runtime
не требуется.

## 2. Выпустить ApiKey

В ProjectMaker: **Настройки → Api-Keys** → **Добавить новый API-ключ** — откроется одноимённый
диалог:
1. Задайте `Название` (произвольное имя ключа, для отличия в списке).
2. Выберите `Макс. уровень` (минимально достаточный для ваших задач — T0 для read-only, выше для
   изменяющих операций).
3. Отметьте нужные `Разрешения` (по умолчанию отмечены только `*:read`; остальные — по
   необходимости).
4. Нажмите **Сгенерировать ключ** — сырой ключ показывается **один раз**, сразу скопируйте его в
   надёжное место.

Подробнее про scopes/tiers — [security-model.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/docs/ru/security-model.md).

## 3. Запустить сервер со своим ключом

Ключ задаётся **в конфигурации самого MCP-сервера** (не в заголовках MCP-клиента — сервер
принимает соединения только с loopback и не читает `Authorization` входящих запросов).
Сервер, у порта которого в таблицах выше нет ¹, может взять ключ из `ApiKey` в `appsettings.json`
рядом со своим exe и запуститься без аргументов; порты, помеченные ¹, всегда задаются в командной
строке вместе с парным `BaseUrl` (и `Target`). Ключ можно передать и через окружение или аргумент,
как ниже. Запускайте только нужные серверы — из блока своего продукта.

**ZennoPoster**

```powershell
# ProjectMaker (редактор): 6207 -> :5299
.\ZennoLab.AI.MCP.ProjectMaker.exe --ProjectMaker:ApiKey=zp_xxx

# Instance для редактора (браузер PM): 6208 -> :5299
.\ZennoLab.AI.MCP.Instance.exe --Instance:ApiKey=zp_xxx

# Instance для раннера — ВТОРАЯ копия того же exe: порт и пара Target/BaseUrl задаются явно
.\ZennoLab.AI.MCP.Instance.exe --urls http://localhost:6209 `
  --Instance:Target=zennoposter --Instance:BaseUrl=http://localhost:5300/api/v1 `
  --Instance:ApiKey=zp_xxx

# ZennoPoster (задачи/сессии раннера): 6210 -> :5300
.\ZennoLab.AI.MCP.ZennoPoster.exe --ZennoPoster:ApiKey=zp_xxx
```

**ZennoDroid**

```powershell
# ProjectMaker (редактор): 6217 -> :5309
.\ZennoLab.AI.MCP.ProjectMaker.exe --urls http://localhost:6217 `
  --ProjectMaker:BaseUrl=http://localhost:5309/api/v1 --ProjectMaker:ApiKey=zp_xxx

# ZennoDroid (задачи/сессии раннера): 6220 -> :5310
.\ZennoLab.AI.MCP.ZennoPoster.exe --urls http://localhost:6220 `
  --ZennoPoster:BaseUrl=http://localhost:5310/api/v1 --ZennoPoster:ApiKey=zp_xxx

# Android (устройство, подключённое к ProjectMaker): 6211 -> :5309
.\ZennoLab.AI.MCP.Android.exe --Android:ApiKey=zp_xxx

# Android для раннера — ВТОРАЯ копия того же exe, устройства выполняющихся задач:
# порт и пара Target/BaseUrl задаются явно
.\ZennoLab.AI.MCP.Android.exe --urls http://localhost:6212 `
  --Android:Target=zennoposter --Android:BaseUrl=http://localhost:5310/api/v1 `
  --Android:ApiKey=zp_xxx
```

`MCP.Android` монтируется дважды по той же причине, что и `MCP.Instance` на ZennoPoster: домен
Android обслуживают оба хоста — PublicApi ProjectMaker (устройство, которое вы видите в редакторе)
и PublicApi раннера (устройства, которыми управляют его задачи). Как и у `MCP.Instance`, у него
есть `Target` — `projectmaker` для устройства редактора, `zennoposter` для устройств раннера, —
который выбирает инструкции, отдаваемые ИИ при initialize (одно устройство, id которого можно
оставить 0, либо по устройству на рабочий поток с id из `list_devices`); набор инструментов в обоих
случаях один и тот же. `Target` и `BaseUrl` задаются только вместе, как пара — ровно как у
`MCP.Instance` (см. замечание ниже). `Target` появился в `MCP.Android` 0.4.0; более ранние версии
различаются только `--urls` и `BaseUrl`.

**Важно про два экземпляра `MCP.Instance` и два экземпляра `MCP.Android`**: у этих серверов
`Target` (какие инструкции для ИИ они отдают — про редактор ProjectMaker или про раннер) и
`BaseUrl` (куда реально уходят HTTP-запросы) — задаются только вместе, как пара, и ничем не
связаны в коде (`Target=projectmaker` → `BaseUrl` на PM PublicApi `:5299`, `Target=zennoposter`
→ `BaseUrl` на ZP PublicApi `:5300`; на ZennoDroid — `:5309` и `:5310`). При старте сервер
делает best-effort проверку через `/capabilities` целевого хоста и пишет предупреждение в лог при
несовпадении, но если целевой хост в момент старта недоступен, проверка молча пропускается —
рассинхронизация в этом случае не детектируется, и ИИ получит инструкции про один хост, а запросы
будут уходить на другой.

## 4. Настроить LLM-клиент

Готовый фрагмент конфигурации в формате `.vscode/mcp.json` VS Code. Те же записи для Cursor,
Claude Code и других клиентов, а также кнопки установки в один клик — на
[странице установки](https://zennolab.github.io/zennoposter-mcp/ru/install.html).

**ZennoPoster**

```json
{
  "servers": {
    "projectmaker": {
      "type": "http",
      "url": "http://localhost:6207"
    },
    "instance-pm": {
      "type": "http",
      "url": "http://localhost:6208"
    },
    "instance-zp": {
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

**ZennoDroid** — имена отличаются от ZennoPoster, поэтому оба продукта настраиваются в одном
клиенте:

```json
{
  "servers": {
    "projectmaker-droid": {
      "type": "http",
      "url": "http://localhost:6217"
    },
    "zennodroid": {
      "type": "http",
      "url": "http://localhost:6220"
    },
    "android-pm": {
      "type": "http",
      "url": "http://localhost:6211"
    },
    "android-zd": {
      "type": "http",
      "url": "http://localhost:6212"
    }
  }
}
```

Авторизация на этом плече не нужна: MCP-серверы слушают только loopback, а права
определяются ключом, с которым запущен сам сервер (шаг 3).

## 5. Проверить подключение

Выведите список инструментов сервера в MCP-клиенте, затем вызовите read-only инструмент, которому
нужен ключ:

| Сервер | Инструмент |
|---|---|
| `MCP.ProjectMaker` | `get_product_version` |
| `MCP.ZennoPoster` | `tasks_list` |
| `MCP.Instance` | `get_all_tabs`, при открытом браузере |
| `MCP.Android` | `list_devices` |

`ping` отвечает, не проверяя ключ, поэтому показывает только, что сервер запущен. `capabilities` у
`MCP.ProjectMaker` и `MCP.ZennoPoster` показывает, что разрешено ключу, — его `currentScopes` и
`currentMaxTier`; пустой `currentScopes` значит, что ключ не принят или не даёт ни одного scope.

Если ключ невалиден или ему не хватает scope/tier, вызов завершится структурированной ошибкой
(`unauthorized` / `forbidden`, у второй — с полями `required`/`current`). `MCP.Android` возвращает
её начиная с 0.4.0; более ранние версии сообщают только о том, что вызов не удался.

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

Не занимайте **6107–6113** (**6117–6123** на ZennoDroid): это порты встроенных серверов продукта, чужой процесс на любом из них
не даст встроенному серверу стартовать.

**Порт API продукта.** Задаётся в `BaseUrl` секции этого сервера — аргументом вида
`--ProjectMaker:BaseUrl=http://localhost:<порт>/api/v1`, переменной окружения вида
`ProjectMaker__BaseUrl` или значением `BaseUrl` в `appsettings.json` рядом с exe. Именно так
устроены команды ZennoDroid в шаге 3.

Секция называется по имени сервера: `ProjectMaker`, `Instance`, `ZennoPoster`, `Android`.
В версиях до 0.3.0 включительно `MCP.ProjectMaker` и `MCP.ZennoPoster` использовали `NeuroBot` и
`ZennoPosterApi`; эти имена работают и в более новых версиях, при старте сервер пишет предупреждение в лог.

## Лицензия

Файлы этого репозитория (документация и спецификация OpenAPI) распространяются по
[лицензии MIT](https://github.com/ZennoLab/zennoposter-mcp/blob/main/LICENSE). Бинарные сборки
MCP-серверов на странице Releases проприетарные, условия их использования — в
[TERMS.ru.md](https://github.com/ZennoLab/zennoposter-mcp/blob/main/TERMS.ru.md).
