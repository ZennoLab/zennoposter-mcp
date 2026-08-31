[English](../developer-guide.md) · **Русский**

# Руководство разработчика

**См. также:** [index.md](index.md) (индекс) · [errors.md](errors.md) (обработка 401/403/404/409/429/501/503) · [security-model.md](security-model.md) (localhost, модель ключей, аудит) · [versioning-and-deprecation.md](versioning-and-deprecation.md) (политика semver) · [openapi/index.html](../openapi/index.html) (полный справочник OpenAPI в Redoc).

Это руководство показывает по одному характерному примеру на домен — этого достаточно, чтобы поднять интеграцию. Полный список всех операций, методов, путей, тиров и скоупов — в [справочнике операций](../api-reference.md) или в документе OpenAPI.

## Быстрый старт (5 минут, с нуля)

1. Выпустите ключ — см. ниже. Скопируйте необработанный ключ; он показывается ровно один раз.
2. Убедитесь, что ключ работает:

```
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/auth/whoami
```

Ожидайте `200` с отражёнными скоупами/`maxTier` вашего ключа.

3. Проверьте, что вы реально можете делать на этом хосте:

```
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/capabilities
```

Пройдите по массиву `operations` — каждая операция, которую ваш ключ может вызвать прямо сейчас, имеет `isAvailable: true`.

4. Сделайте свой первый безопасный (T0) вызов — без побочных эффектов, например:

```
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/projects/current
```

5. Разберитесь с предсказуемыми сценариями отказа до того, как писать что-то, изменяющее состояние — см. [«Обработка ошибок»](errors.md).

`5299` выше — это порт ProjectMaker по умолчанию; хост ZennoPoster Core привязывает свой порт таким же образом (в обоих случаях только localhost — см. [«Модель безопасности»](security-model.md)).

## Выпуск ключа

Ключи управляются со страницы **API Keys** в настройках ProjectMaker/ZennoPoster , доступной любому, кто запускает продукт на этой машине — отдельного удалённого слоя регистрации нет. Диалог «Add key» собирает:

- **Label** — произвольный текст, для вашего собственного учёта (отображается в `GET /auth/keys`, но никогда не сам ключ).
- **Max tier** — T0–T3 (см. таблицу ниже); по умолчанию T0 (минимум привилегий).
- **Scopes** — чек-лист, сгруппированный по доменам (`project:*`, `task:*`, `instance:*`, `code:*`, `admin`); заранее отмечены только скоупы `*:read`, всё остальное (включая всё изменяющее, запускающее или `code:author`) — опционально.
- **Expiry** — опционально; на wire (`POST /auth/keys`) это абсолютный `expiresAt` — ISO-8601 timestamp, `null`/не задан для бессрочного ключа. Дата в прошлом отклоняется с `400`.

При подтверждении необработанный ключ отображается **один раз** (в стиле GitHub Personal Access Token) — скопируйте его немедленно; повторно получить его нельзя (почему — см. [«Модель безопасности»](security-model.md)). После этого отправляйте его как `Authorization: Bearer <raw-key>` при каждом запросе, кроме `GET /api/v1/ping`.

## Скоупы и тиры

| Тир | Значение | Условие доступа |
|---|---|---|
| T0 | Чтение/просмотр, без побочных эффектов. | Только ApiKey. |
| T1 | Изменяет проект/модель, обратимо, без выполнения. | ApiKey + скоуп. |
| T2 | Выполнение с собственными привилегиями приложения (запустить задачу, завершить сессию). | ApiKey + скоуп + аудит. |
| T3 | Достигает уровня ОС — файловая система/сеть/компиляция+запуск OwnCode, либо администрирование ключей/аудита. Класс RCE. | ApiKey + скоуп + аудит (+ HITL в будущем). |

Скоупы организованы по доменам с пространством имён: `project:read/edit/run/record`, `task:read/edit/control`, `instance:read/control/interact`, `code:read/author`, а также сквозной `admin`. Контентные скоупы (`io:filesystem`, `io:network`, `io:database`, `system:exec`, `code:author`) требуются **в момент исполнения**: запуск задачи/проекта, чьи действия трогают файловую систему, сеть, БД или ОС, требует соответствующего скоупа `io:*`/`system:exec` поверх собственного скоупа операции, а проект с кодом или содержащий OwnCode требует `code:author`. Тело `403` перечисляет, чего именно не хватает (`missingScopes`, `dangerousCategories`, `unknownCategories`).

Каждая операция в каталоге несёт ровно один тир и не более одного требуемого скоупа (`null` для `ping`/`whoami`/`capabilities`, которым нужен лишь валидный ключ, если вообще нужен — `ping` не требует ничего).

## Примеры запросов по доменам

Все пути ниже указаны относительно `/api/v1`. Каждый ответ использует общий конверт (`resultCode` для домена ProjectMaker, либо специфичное для домена тело — точную схему по каждой операции см. в документе OpenAPI; это руководство показывает один показательный пример на домен, чтобы дать ориентир, а не полный набор).

### ProjectMaker (`project:*`) — хост `:5299`

```
GET  /projects/current           (T0, project:read)   — информация об открытом проекте
POST /projects/current/actions   (T1, project:edit)   — добавить действие в проект
POST /projects/current/actions/{groupId}/{actionId}/execute
                                  (T3, project:run)   — выполнить одно действие (класс RCE)
```

Добавление действия, чей `type` — `OwnCode`, дополнительно требует `code:author` (T3) сверх `project:edit` — авторство нового исполняемого кода и запуск существующего проекта разграничены по скоупам: ключ с низкими привилегиями может запустить доверенный проект, но не может внедрить и выполнить новый код тем же ключом.

### Задачи ZennoPoster (`task:*`) — хост = ZennoPoster Core

```
GET  /tasks                (T0, task:read)     — список задач
POST /tasks/{id}/start      (T3, task:control)  — запустить задачу (контент-классификация, см. выше)
PUT  /tasks/{id}/config     (T1, task:edit)     — обновить пользовательские настройки задачи
```

`POST /tasks` (добавить задачу) — это T3/`task:edit`: либо ссылается на файл проекта, уже лежащий на диске (`application/json`-тело `{ "path": "C:\\bots\\my.zp" }`), либо загружает байты проекта (`application/octet-stream`, `?name=my.zp`). Принимаются только `.zp`, `.zpproj` и `.vsproj`. В любом варианте считайте это рискованной операцией: импортированный проект сам может содержать OwnCode.

**Какие «start-образные» вызовы гейтятся.** Всё, что может запустить код проекта, проходит тот же гейт контент-классификации, что и `/start`: добавление/установка ненулевого числа попыток на READY-задаче (`Newbie`/`Complete`/`NotComplete` — семантика ZP7, общая с кнопками «+N» в UI), `PUT /settings/execution` с итоговым ненулевым счётчиком попыток на готовой задаче и `PUT /settings/scheduler` с `isActive: true` (активное расписание — это отложенный запуск). На остановленной/выполняющейся/запланированной задаче те же вызовы — обычные T1-правки.

#### Настройки задачи

`GET /tasks/{id}` возвращает `settingsType` — где живут пользовательские настройки задачи: **`InputSettings`** — проект объявляет сырой блок input-settings; **`BotUI`** — проект использует форму BotUI; **`None`** — пользовательских настроек нет. В любом случае `GET`/`PUT /tasks/{id}/config` — ЕДИНСТВЕННАЯ пара настроек: она обслуживает унифицированное представление обоих видов блоков. Поле `settingsType` может быть пустым, пока проект не был просканирован хотя бы раз (первый запуск или открытие в UI).

Настройки — пары имя+значение, привязанные к переменным проекта:

```json
// GET /tasks/{id}/config
{ "settings": [ { "name": "Login", "variable": "login", "value": "user@example.com" } ] }

// PUT /tasks/{id}/config — отправляйте только то, что меняете; сопоставление по name
// (+variable для устранения неоднозначности дублей). Ответ говорит, что реально совпало:
{ "applied": ["Login"], "skipped": ["Loginn"] }
```

Имя в `skipped` не совпало ни с одной настройкой задачи (опечатка, либо настройка принадлежит другому виду блока) — семантика импорта пропускает его, а не роняет весь батч.

#### Настройки выполнения (merge-семантика)

`PUT /tasks/{id}/settings/execution` мёржит: каждое поле опционально, пропущенное поле сохраняет текущее значение, поэтому `{ "priority": 50 }` меняет приоритет и ничего больше. `GET` возвращает полный текущий набор. Валидация строгая (иначе `400`): `limitOfThreads >= 1`; `numberOfTries >= 0`; `priority` ∈ 10 (low), 50 (medium), 100 (high), 100000 (critical); `maxNumOfSuccessStop`/`maxNumOfFailStop` = `-1` (без лимита) или `>= 1`; `timeout` = `-1` (нет) или минуты `>= 1`; `proxy` ∈ `DoNotUseProxy`, `IfPossible`, `UseProxyWithoutRemove`, `UseProxy`. Системный потолок `maxAllowOfThreads` не записываемый — читайте его в `GET /tasks/{id}/state`.

#### Потоки и попытки

`/tasks/{id}/threads/max` (GET/PUT) — пользовательский **лимит** потоков (`limitOfThreads`); `/tasks/{id}/tries` (GET/PUT) и `/tries/add` (POST) управляют счётчиком поставленных в очередь попыток. Число **выполняющихся сейчас** потоков — другое значение: `countOfThreads` в `GET /tasks/{id}/state`. PUT-ответы возвращают значение, реально применённое после коммита, а не эхо запроса.

#### Состояние времени выполнения и точки отказа

`GET /tasks/{id}/state` возвращает счётчики плюс `failurePoints`: **снапшот последнего отказа по каждому рабочему потоку**, а НЕ очередь событий. Чтение неразрушающее — повторные GET возвращают те же записи; запись перезаписывается на месте, когда её поток падает снова (ключ taskId+threadId), истекает через 2 часа, ограничена 64 записями на задачу и полностью очищается только при удалении задачи. Дедуплицируйте на клиенте по `(threadId, timestamp)` или `uowId`. Внимание: снапшот `variables` внутри точки отказа может содержать чувствительные значения и читается со скоупом `task:read` (T0) — скоупируйте ключи соответственно.

#### Настройки планировщика

`GET`/`PUT /tasks/{id}/settings/scheduler`. Форматы: списки — через запятую; `daysOfWeek` — английские имена дней с заглавной буквы (`Monday` … `Sunday` — другие регистры принимаются и нормализуются, неизвестные имена — `400`); элементы `intervals` — `"HH:mm-HH:mm"` (конец `"00:00"` означает 24:00 — конец дня; одиночное время вроде `"09:30"` — точечный интервал); `attemptsRange`, `repeatCountDayRange` и `repeatCountTotalRange` — `"n"` или `"a-b"`; **даты (`startDate`/`endDate`) — invariant-culture строки в ЛОКАЛЬНОМ ВРЕМЕНИ МАШИНЫ, без часового пояса** — единственное исключение из общей конвенции API «абсолютное время», потому что сам планировщик работает по часам машины. `isActive: true` требует реального `executePeriod` (`400` на `NULL`) и гейтится как `/start` (см. выше). Четыре типичных тела:

```json
// 1. Разовый запуск в конкретную дату/время
{ "isActive": true, "executePeriod": "OneTime", "startDateType": "OnDate",
  "startDate": "08/30/2026 09:00:00", "attemptsRange": "1", "isClearSuccess": false }

// 2. Каждый день, с 09:00 до 18:00, с регулярным повторением в течение дня
{ "isActive": true, "executePeriod": "EveryDay", "startDateType": "Immediately",
  "attemptsRange": "1", "isClearSuccess": true,
  "intervals": ["09:00-18:00"], "stopExecutionOutsideOfIntervals": true,
  "repeatType": "Regularly", "repeatCountDayRange": "10", "endDateType": "Infinity" }

// 3. По выбранным дням недели
{ "isActive": true, "executePeriod": "EveryWeek", "daysOfWeek": ["Monday", "Wednesday", "Friday"],
  "startDateType": "Immediately", "attemptsRange": "1-3", "isClearSuccess": false,
  "repeatType": "Continued", "endDateType": "Infinity" }

// 4. По появлению файла-триггера (файл удаляется после срабатывания)
{ "isActive": true, "executePeriod": "OnDemand", "fileName": "C:\\triggers\\run.txt",
  "isNeedDeleteFile": true, "attemptsRange": "1", "isClearSuccess": false }
```

### Сессии взаимодействия (`task:*`) — хост = ZennoPoster Core

Сессия — это активное окно `WaitForUserAction` у выполняющейся задачи/инстанса.

```
GET  /sessions                  (T0, task:read)     — список открытых окон
POST /sessions/{id}/complete    (T2, task:control)   — завершить окно, продолжить поток
```

`{ "summary": "approved", "metadata": {} }` — типичное тело для `complete`. Если окно уже закрылось (по таймауту, либо его завершил другой вызывающий), вы получите `409 no_active_interaction` — см. [«Обработка ошибок»](errors.md). Эндпоинта `cancel`/ветки-отказа в этой версии нет.

`started` сессии — UTC ISO-8601 момент открытия окна `WaitForUserAction`; вместе с `deadline` (присутствует, когда у действия есть таймаут) это даёт оставшееся время.

**Наблюдение за окнами:** `GET /sessions/events?timeoutSeconds=N` (1–60, дефолт 25) — ограниченный long-poll: блокируется, пока не придёт хотя бы одно событие открытия/закрытия (всплески батчатся) или не истечёт окно, затем возвращает `{ "events": [...], "timedOut": bool }`. Это живое наблюдение, а не долговременный журнал: бэклога нет, события между двумя поллами теряются — переполлируйте в цикле, а для снимка на момент времени используйте `GET /sessions`. Конкурентные поллы на хост ограничены (дефолт 32; сверх — `429 rate_limited`).

### Instance (браузер) — `instance:*`

Общий контракт, два хоста: ProjectMaker предоставляет свой единственный debug-инстанс; раннер ZennoPoster предоставляет один инстанс на каждую выполняющуюся задачу/порт. Управление вкладками и DOM — обычная автоматизация: вызову `instance:interact` не нужно открытое окно сессии — он ограничен только скоупом `instance:interact` и тиром самой операции. (Окно `WaitForUserAction` — это просто частный случай, когда задача ждёт, что вы будете ей управлять; это не предпосылка.)

```
GET  /instances                                     (T0, instance:read)
POST /instances/{id}/tabs/{tabId}/navigate          (T1, instance:interact)  — { "url": "...", "timeout": 5 }
POST /instances/{id}/tabs/{tabId}/elements/event    (T1, instance:interact)  — { "xpath": "...", "eventName": "click" }
```

**Release ≠ close.** `DELETE /instances/{id}` мягко возвращает запущенный через API браузер в **пул**, и пул сам решает — освободить или перезапустить: окно может остаться жить для переиспользования; это by design, а не утечка. Порты, принадлежащие рабочему потоку выполняющейся задачи, защищены: `409 instance_busy`. `POST /instances/{id}/show` отвечает `409 instance_view_protected`, когда вид браузера защищён (защита вида включена, открытого окна `WaitForUserAction` нет) — `200` всегда означает, что окно реально показано.

### Сквозные (`admin`, либо без скоупа)

```
GET  /auth/whoami      (T0, без скоупа)  — собственные скоупы/тир вашего ключа
GET  /capabilities     (T0, без скоупа)  — манифест ниже
POST /auth/keys        (T3, admin)      — выпустить ключ (та же операция, которую вызывает UI)
GET  /audit            (T0, admin)      — журнал вызовов
```

**`GET /audit` — журнал ВСЕХ HTTP-вызовов PublicApi на хосте**, а не только операций с ключами. Каждая запись несёт временную метку, id/label ключа (никогда сам токен), метод, путь и код статуса; фильтруйте `?since=`/`?until=` (ISO-8601), `?keyId=`, `?method=`, `?statusCode=`, пагинация `?skip=`/`?take=`.

**`GET /auth/keys`** перечисляет каждую запись ключа с `systemProvisioned` (автовыпущен хостом для его встроенных MCP-сайдкаров — вы их не создавали, и UI «My keys» их скрывает) и `status` (`active`, `expired` или `disabled`). Истёкший/отключённый ключ уже не проходит аутентификацию, но остаётся в списке до отзыва — отзыв удаляет запись физически и необратим. Фильтры: `?includeSystemProvisioned=false` и `?status=` (`active`, `expired` или `disabled`).

**`GET /code-api`** (хост ProjectMaker, `code:read`) отдаёт поверхность C# API, доступную кубикам OwnCode, из встроенного code-api-дайджеста продукта. Быстрый рецепт:

```bash
# индекс доступных типов
curl -H "Authorization: Bearer <api-key>" http://localhost:5299/api/v1/code-api
# члены одного типа
curl -H "Authorization: Bearer <api-key>" "http://localhost:5299/api/v1/code-api?typeName=IZennoTable"
```

## `/capabilities`

`GET /api/v1/capabilities` возвращает для ключа из запроса (или анонимное представление, если вызвано без ключа — см. ниже):

```json
{
  "host": "zennoposter",
  "version": "1.0.0",
  "currentScopes": ["task:read", "task:control"],
  "currentMaxTier": 2,
  "operations": [
    {
      "operationId": "session_complete",
      "method": "POST",
      "path": "/api/v1/sessions/{id}/complete",
      "tier": 2,
      "requiredScope": "task:control",
      "isAvailable": true
    }
  ],
  "tierLegend": { "T0": "read", "T1": "edit", "T2": "execute+audit", "T3": "host/RCE" }
}
```

`host` показывает, с какой поверхностью вы общаетесь (`zennoposter` обслуживает операции ZennoPoster + Instance + сквозные; ProjectMaker обслуживает ProjectMaker + Instance + сквозные — два хоста не предоставляют одинаковый набор операций). `isAvailable` равно `true` только тогда, когда операция и подключена на этом хосте (`IsImplemented`), и разрешена для вашего ключа (скоуп + тир) — значение `false` может означать как «не ваша операция для вызова», так и «ещё не реализована на этом хосте»; сравните `requiredScope`/`tier` с собственными `currentScopes`/`currentMaxTier`, чтобы понять, что именно. Вызов `/capabilities` перед попыткой вызова — самый дешёвый способ избежать предсказуемых 403/501.

<!-- translated-from: developer-guide.md 415de58ea72623c47cb95cb16371858273a434fb -->
