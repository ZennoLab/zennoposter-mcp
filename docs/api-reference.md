# ZennoLab PublicApi — operation reference

> **Generated file — do not edit by hand.** It is produced from the same contract as
> `openapi.v1.json`, so it never drifts from the API it describes.

All paths are relative to the version base path `/api/v1`. Every request is
authorized by the shared gate: a missing/insufficient key is `401`/`403`; error bodies use
the unified `ApiError` shape. Risk tiers: **T0** read-only · **T1** edit · **T2** control ·
**T3** execution (RCE class). See `security-model.md` and `errors.md` for the full model.

**How to read the `Required` column.** In a **request body**, `yes` means the field must be
provided. In a **success body**, `yes` means the field is always present and never `null`
(a non-nullable value type on the wire) — it says nothing about what you must send. Fields
marked `no` in a response may be `null` or absent.

## Domains

- [ProjectMaker — editor automation](#projectmaker--editor-automation)
- [ZennoPoster — task & session control](#zennoposter--task--session-control)
- [Instance — browser instances, tabs & elements](#instance--browser-instances-tabs--elements)
- [Android — ZennoDroid device](#android--zennodroid-device)
- [Cross-cutting — auth, audit & capabilities](#cross-cutting--auth-audit--capabilities)
- [Declared, not implemented](#declared-not-implemented)

## Declared, not implemented

The following operations are part of the contract but have **no implementation yet** —
calling any of them returns `501 not_implemented`. They are listed in the capabilities
manifest with `implemented: false`.

- `GET /confirmations` (`confirmations_list`) — Pending HITL confirmations.
- `POST /confirmations/{id}/approve` (`confirmation_approve`) — Approve an operation (UI only).
- `POST /confirmations/{id}/reject` (`confirmation_reject`) — Reject an operation (UI only).

## ProjectMaker — editor automation

**Base URL:** `http://localhost:5299/api/v1 (ProjectMaker editor)`

### `GET /ping`

`ping` · tier **T0** · scope `—`

Liveness check of the integration.

**Responses:** `200` success · `400` bad_request · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /product/version`

`product_version` · tier **T0** · scope `project:read`

Product name and version.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `productVariant` | string | no | Product edition name, e.g. "ZennoPoster Pro" or "ZennoDroid Enterprise" — lets the client adapt to browser vs Android capabilities. |
| `productVersion` | string | no | Full product version string (e.g. "7.7.7.0"). |

### `GET /projects/current`

`project_get_current` · tier **T0** · scope `project:read`

Info about the open project.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `zennoPosterVersion` | object | no | Version of the ZennoPoster/ProjectMaker product currently running. |
| `projectName` | string | no | Name of the currently open project (active tab). |
| `projectPath` | string | no | Full path of the project's .zp file on disk; null when the project has not been saved yet. |
| `projectId` | string | no | Stable unique id stored inside the project. |
| `projectMinVersion` | object | no | The project's UpdateVersion — the product version that last wrote the .zp; treat it as the minimum product version expected to open the project. |
| `lastModified` | string | yes | Last modification time (UTC) of the project; default (zero) when unknown. |
| `isZDroid` | boolean | yes | True when the host product is ZennoDroid (Android automation) rather than ZennoPoster. |
| `isRealPhone` | boolean | yes | True on ZennoDroid Enterprise, which drives real phones instead of emulators. Always false outside ZennoDroid. |
| `hasUnsavedChanges` | boolean | yes | True when the in-memory project has edits that have NOT yet been written to its .zp file — i.e. the file on disk is stale relative to what is open in ProjectMaker. Call SaveProject before running the project in ZennoPoster. For a brand-new project with no file yet, this is false only if there is also no edit history (nothing to save). |
| `fileHash` | string | no | Freshness token: lowercase hex SHA-256 of the .zp file bytes currently on disk. This describes the SAVED file, not the in-memory edits — when HasUnsavedChanges is true this hash still reflects the previous save. Matches the FileHash returned by SaveProject/OpenProject for the same unchanged file. Null if the project has no file yet or the file could not be read. |
| `fileSizeBytes` | integer | no | Size in bytes of the .zp file on disk. Null if no file / unreadable. |
| `lastWriteTimeUtc` | string | no | Last write time (UTC) of the .zp file on disk. Null if no file / unreadable. |

### `POST /projects`

`project_create` · tier **T1** · scope `project:edit`

Create a new project.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `projectName` | string | no | Name of the newly created project. |
| `projectPath` | string | no | Full path on disk, or null for a freshly created project that has not been saved yet (use SaveProject with a path to persist it). |
| `projectId` | string | no | Unique id assigned to the project. |

### `POST /projects/open`

`project_open` · tier **T1** · scope `project:edit`

Open a project by path.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | string | no | Full path to the project file to open (e.g. *.zp / *.xmlz). |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `projectName` | string | no | Name of the opened project. |
| `projectPath` | string | no | Full path of the opened .zp file. |
| `projectId` | string | no | Stable unique id stored inside the project. |
| `fileHash` | string | no | Freshness token: lowercase hex SHA-256 of the opened .zp file bytes on disk. Matches the FileHash returned by SaveProject for the same unchanged file. Null if the file could not be read back after opening. |
| `fileSizeBytes` | integer | no | Size in bytes of the opened .zp file. Null if the file could not be read back. |
| `lastWriteTimeUtc` | string | no | Last write time (UTC) of the opened .zp file. Null if the file could not be read back. |
| `projectVersion` | string | no | Version stamped into the opened .zp (the project's UpdateVersion). Part of the freshness token alongside the hash; matches the value SaveProject returned for the same file. |

### `POST /projects/current/save`

`project_save` · tier **T1** · scope `project:edit`

Save the open project.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | string | no | Optional target file path ("save as"). When null/empty the project is saved in place to its existing file. A path is REQUIRED the first time a freshly created project is saved (it has no file yet). |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` failed_precondition · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `projectName` | string | no | Name of the saved project. |
| `projectPath` | string | no | Full path the project was saved to. |
| `fileHash` | string | no | Freshness token: lowercase hex SHA-256 of the saved .zp file bytes on disk, computed by reading the file back AFTER the save completed. Null if the file could not be read back (the save itself still succeeded). OpenProject returns the identical hash for the same unchanged file. |
| `fileSizeBytes` | integer | no | Size in bytes of the saved .zp file. Null if the file could not be read back. |
| `lastWriteTimeUtc` | string | no | Last write time (UTC) of the saved .zp file. Null if the file could not be read back. |
| `projectVersion` | string | no | Version stamped into the saved .zp (the project's UpdateVersion — the product version that wrote the file). Part of the freshness token alongside the hash; stored inside the file, so OpenProject on the same file returns the same value. |

### `POST /projects/current/close`

`project_close` · tier **T1** · scope `project:edit`

Close the open project.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `discardUnsavedChanges` | boolean | yes | Close the project even when it has unsaved changes, discarding them. When false (the default) a changed project is not closed: the call fails with 409 failed_precondition so no work is lost silently — save first (project_save) or opt into discarding explicitly. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` failed_precondition · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `projectName` | string | no | Name of the project that was closed. |
| `projectPath` | string | no | Path of the closed project's file; null for a never-saved project. |

### `POST /projects/current/start`

`project_start` · tier **T3** · scope `project:run`

Run the open project — executes project code incl. OwnCode (RCE class).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `started` | boolean | yes | True when the project run was triggered. Execution happens asynchronously — poll GetExecutionLogs / GetLastError / GetProjectStructure (action ExecutionState) to observe progress and completion. |

### `POST /projects/current/stop`

`project_stop` · tier **T2** · scope `project:run`

Stop the running project.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /actions/catalog`

`actions_catalog_list` · tier **T0** · scope `project:read`

Catalog of action types.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `categories` | array of object | no | All visible catalog categories with their action types and variants; hidden actions are excluded. |

### `GET /actions/catalog/search`

`actions_catalog_search` · tier **T0** · scope `project:read`

Search actions (query/category/maxResults).

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `totalMatches` | integer | yes | Total variants that matched, before the maxResults cut. |
| `truncated` | boolean | yes | True when more variants matched than were returned — narrow the query/category. |
| `results` | array of object | no | Matched action variants, best matches first, capped by maxResults. |

### `GET /actions/catalog/{type}/{action}/schema`

`actions_catalog_schema` · tier **T0** · scope `project:read`

Parameter/result schema of a variant.

**Path parameters:** `type`, `action`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | no | Echo of the requested action type (catalog category tag, e.g. "WebBrowser", "Logic"). |
| `action` | string | no | Echo of the requested action tag. |
| `description` | string | no | Human-readable description of this variant. |
| `actionTypeDescription` | string | no | Description of the owning action type (shared by all its variants). |
| `schemaVersion` | integer | yes | Version of the variant's schema definition; bumps when the parameter surface changes. |
| `parameters` | array of object | no | Parameter definitions of the variant: kind, required-ness (including conditional RequiredWhen rules), allowed values, defaults. |
| `results` | array of object | no | Result variables the variant can emit (e.g. OutputVariable). |
| `bindingDiscriminator` | string | no | Name of the discriminator field inside finder JSON (e.g. FinderType), when the category uses bindings. |
| `acceptedBindings` | array of string | no | Names of the binding kinds this variant accepts; null when it takes no finder. |
| `bindings` | array of object | no | Resolved definitions of the accepted bindings (field shapes); null when it takes no finder. |

### `GET /projects/current/structure`

`project_structure_get` · tier **T0** · scope `project:read`

List actions (without parameters).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `groupId` | string | no | Optional. When set, scope the returned nodes/edges to this single group — useful to keep the payload small on very large templates. Empty/null returns the whole graph. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actions` | array of object | no | Graph nodes — one per placed cube (see ActionInfo). |
| `links` | array of object | no | Graph edges — every outgoing branch of every cube. Each ActionLink is a source (ActionId/GroupId) → target (TargetActionId, a bare action id) hop on a branch (OnSuccess/OnError/Default/Case:N); IsImplicit marks the next-in-group OnSuccess fallback. |
| `startActionId` | string | no | Entry cube the Start block points at, or null when unwired/empty project. |
| `startGroupId` | string | no | Group of the entry cube, or null when unwired/empty project. |

### `GET /projects/current/actions/{groupId}/{actionId}`

`action_get` · tier **T0** · scope `project:read`

Action details (params/results/finder/links).

**Path parameters:** `groupId`, `actionId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Id of the inspected action. |
| `groupId` | string | no | Id of the group the action belongs to. |
| `type` | string | no | Action type (catalog category tag, e.g. "WebBrowser") — ready to pass to UpdateAction. |
| `action` | string | no | Action tag within the type (e.g. "CMD_STARTINSTANCE") — ready to pass to UpdateAction. |
| `parameters` | array of object | no | Current parameter values, keyed by the schema parameter name. Only parameters declared in the catalog variant are included; absent values are returned as empty strings so the caller sees the full schema-defined surface. |
| `results` | array of object | no | Current result-binding values (e.g. OutputVariable → variable macro). |
| `finderType` | string | no | For actions that use a structured element finder (HtmlElement and similar): short name of the active finder — e.g. XPathFinder, DomFinder, IntelliSearch, ImageFinder, RawCoordinatesFinder. null for actions that do not use finders. |
| `finder` | array of object | no | Generic field map of the active finder's sub-elements. Plain leaf elements become strings; repeated children with the same tag become arrays (e.g. DomFinder's SearchCondition list). null when the action has no finder. |
| `links` | array of object | no | Outgoing branches of the action: OnSuccess and OnError. Each entry's IsImplicit is true when the target is computed from the next-action-in-group rule rather than stored explicitly. null targetActionId means the action is the terminal step on that branch (no successor at all). |

### `POST /projects/current/actions`

`action_add` · tier **T1** · scope `project:edit`

Add an action (OwnCode/CSharp category requires code:author, T3).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | no | Action type (catalog category tag, e.g. "WebBrowser", "TextProcessing"). Discover values via the actions catalog (GetActionsCatalog / FindActions). |
| `action` | string | no | Action tag within the type (e.g. "CMD_STARTINSTANCE", "ToUpper"). Together with Type it uniquely addresses one catalog variant. |
| `parameters` | array of object | no | Parameter values keyed by the schema parameter name of the selected variant (see GetActionSchema). Validated against the catalog schema; missing required parameters are rejected with validation errors. |
| `results` | array of object | no | Result bindings keyed by the schema result name (e.g. OutputVariable). The value is the project variable name to store the result into; variables that do not exist yet are created. |
| `bindings` | object | no | Optional structured-binding payload (e.g. ElementFinder / XPathFinder). When non-null, the engine applies it to action.Parameters after writing flat parameters and before binding-validation. |
| `succesLink` | string | no | Reserved — currently ignored by the engine; wire the OnSuccess branch via SetActionLinks instead. |
| `failLink` | string | no | Reserved — currently ignored by the engine; wire the OnError branch via SetActionLinks instead. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Id assigned to the newly created action. |
| `groupId` | string | no | Id of the group the action was placed into. |
| `validationErrors` | array of object | no | Per-parameter validation failures when the input was rejected (ResultCode RESULT_INVALID_PARAMS); null on success. |

### `PUT /projects/current/actions/{groupId}/{actionId}`

`action_update` · tier **T1** · scope `project:edit`

Update an action.

**Path parameters:** `groupId`, `actionId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | no | Action type (catalog category tag) the action should carry — pass the value returned by GetActionDetails to keep the current type. |
| `action` | string | no | Action tag within the type — pass the value returned by GetActionDetails to keep the current action. |
| `parameters` | array of object | no | Parameter values keyed by the schema parameter name of the selected variant (see GetActionSchema). Validated against the catalog schema. |
| `results` | array of object | no | Result bindings keyed by the schema result name (e.g. OutputVariable). The value is the project variable name to store the result into; missing variables are created. |
| `bindings` | object | no | Optional structured-binding payload — see Bindings. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `validationErrors` | array of object | no | Per-parameter validation failures when the input was rejected (ResultCode RESULT_INVALID_PARAMS); null on success. |

### `DELETE /projects/current/actions/{groupId}/{actionId}`

`action_delete` · tier **T1** · scope `project:edit`

Delete an action.

**Path parameters:** `groupId`, `actionId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

_(empty object)_

### `POST /projects/current/actions/move`

`actions_move` · tier **T1** · scope `project:edit`

Move actions into a group.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `actions` | array of object | no | Actions to move, identified by (GroupId, ActionId). Order matters — the actions land in this order. |
| `targetGroupId` | string | no | Target group ID. Leave null/empty to create a NEW group and put the actions there. |
| `afterActionId` | string | no | Within the target group, insert the moved actions immediately AFTER this action. Null/empty means "insert at the start of the target group" (or "at the end" if the target is a fresh group). Must reference an action that exists in the target group BEFORE the move. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `targetGroupId` | string | no | ID of the destination group (the existing one, or the freshly created group when none was supplied). |
| `movedActions` | array of object | no | Updated (GroupId, ActionId) pairs after the move — actions retain their IDs but now report the new GroupId. |

### `POST /projects/current/actions/links`

`action_links_set` · tier **T1** · scope `project:edit`

Set OnSuccess/OnError links (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `links` | array of object | no | Branch assignments to apply. Each item names the source action, the branch ("OnSuccess" or "OnError") and the target action; a null/empty target clears the explicit link. Processed as a batch — see the per-item outcomes in the result. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-link outcomes in input order (Name carries the source action id). Failed items do not prevent the other links from being applied. |

### `GET /projects/current/cursor`

`cursor_get` · tier **T0** · scope `project:read`

Cursor position.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Id of the action the ProjectMaker cursor (current selection) is on. ResultCode is RESULT_NOT_FOUND when nothing is selected. |
| `groupId` | string | no | Id of the group containing the selected action. |

### `PUT /projects/current/cursor`

`cursor_set` · tier **T1** · scope `project:edit`

Set the cursor.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Id of the action to select (move the ProjectMaker cursor to). The action is resolved by this id alone. |
| `groupId` | string | no | Id of the group containing the action (accompanies ActionId; not used for the lookup). |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

_(empty object)_

### `POST /projects/current/actions/{groupId}/{actionId}/execute`

`action_execute` · tier **T3** · scope `project:run`

Execute a single action — RCE class.

**Path parameters:** `groupId`, `actionId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `successActionId` | string | no | Id of the action execution continues with on success: the explicit OnSuccess link, or the implicit next action in the group when no link is set. Null when the action is terminal. |
| `failActionId` | string | no | Id of the action execution continues with on error — only when an explicit OnError link is set on this action. Null means no OnError branch. |
| `executeStatus` | enum(NotStarted \| Executing \| Success \| Fail) | yes | Execution status of the action sampled right after the run was started: NotStarted \| Executing \| Success \| Fail. When still Executing, poll the execute-status operation until it reaches a terminal state. |

### `GET /projects/current/actions/{groupId}/{actionId}/execute/status`

`action_execute_status` · tier **T0** · scope `project:read`

Execution status of an action.

**Path parameters:** `groupId`, `actionId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Echo of the requested action id, so parallel status polls stay attributable. |
| `groupId` | string | no | Echo of the requested group id. |
| `successActionId` | string | no | Id of the action execution continues with on success: the explicit OnSuccess link, or the implicit next action in the group when no link is set. Null when the action is terminal. |
| `failActionId` | string | no | Id of the action execution continues with on error — only when an explicit OnError link is set on this action. Null means no OnError branch: the error propagates upward (there is no implicit next-in-group fallback for OnError). |
| `executeStatus` | enum(NotStarted \| Executing \| Success \| Fail) | yes | Current execution status of the action: NotStarted (never ran) \| Executing \| Success \| Fail. |
| `executionError` | string | no | Message of the error that failed this action — set only when ExecuteStatus is Fail and the project's last error belongs to this action. |

### `GET /projects/current/last-error`

`last_error_get` · tier **T0** · scope `project:read`

Last execution error.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `actionId` | string | no | Id of the action that raised the project's last execution error. |
| `groupId` | string | no | Id of the group containing that action. |
| `lastError` | string | no | Message of the last execution error. ResultCode is RESULT_NOT_FOUND when the project has no recorded error. |

### `GET /projects/current/logs`

`logs_get` · tier **T0** · scope `project:read`

Execution logs (filter by level).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `skipCount` | integer | yes | Paging offset: number of matching log entries to skip (query parameter "skip"; default 0). |
| `maxCount` | integer | yes | Maximum number of entries to return (query parameter "take"; the server defaults to 100 when omitted or non-positive). |
| `logLevel` | enum(Info \| Warning \| Error) | yes | Severity filter — a flags combination of Info (1), Warning (2), Error (4). 0/omitted means all levels. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `logs` | array of object | no | Matching execution-log entries after the skip/take paging window is applied. |

### `GET /projects/current/variables`

`variables_list` · tier **T0** · scope `project:read`

List variables.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `variables` | array of object | no | All project variables with their full definitions and current values. |
| `totalCount` | integer | yes | Total number of variables in the project. |

### `POST /projects/current/variables`

`variables_add` · tier **T1** · scope `project:edit`

Add variables (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `variables` | array of object | no | Definitions of the variables to create. Each definition is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `PUT /projects/current/variables`

`variables_update` · tier **T1** · scope `project:edit`

Update variables (batch; rename via newName).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `variables` | array of object | no | Definitions of the variables to update, matched by Name; null fields are left unchanged. Each definition is processed independently. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `DELETE /projects/current/variables`

`variables_delete` · tier **T1** · scope `project:edit`

Delete variables by name.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `names` | array of string | no | Names of the variables to remove. Each name is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input name; a failed item does not fail the rest of the batch. |

### `GET /projects/current/variables/{name}/value`

`variable_value_get` · tier **T0** · scope `project:read`

Runtime + default value.

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | no | Variable name echoed from the request. |
| `value` | string | no | Current (runtime) value of the variable. |
| `defaultValue` | string | no | Design-time default value the variable starts with on each project run. |

### `PUT /projects/current/variables/{name}/value`

`variable_value_set` · tier **T1** · scope `project:edit`

Set runtime value.

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `value` | string | no | New current (runtime) value for the variable. Does not change the variable's design-time default value. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /projects/current/lists`

`lists_list` · tier **T0** · scope `project:read`

List the project's Lists.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `lists` | array of object | no | All project lists with their full definitions. |

### `POST /projects/current/lists`

`lists_add` · tier **T1** · scope `project:edit`

Add lists (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `lists` | array of object | no | Definitions of the lists to create. Each definition is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `PUT /projects/current/lists`

`lists_update` · tier **T1** · scope `project:edit`

Update lists (batch; rename via newName).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `lists` | array of object | no | Definitions of the lists to update, matched by Name; null fields are left unchanged. Each definition is processed independently. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `DELETE /projects/current/lists`

`lists_delete` · tier **T1** · scope `project:edit`

Delete lists by name.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `names` | array of string | no | Names of the lists to remove. Each name is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input name; a failed item does not fail the rest of the batch. |

### `GET /projects/current/lists/{name}/items`

`list_items_get` · tier **T0** · scope `project:read`

Runtime list items (skip/take).

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `skip` | integer | yes | Number of items to skip from the start of the list. Default is 0. |
| `take` | integer | yes | Maximum number of items to return. Default is 1000. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | no | List name echoed from the request. |
| `items` | array of string | no | Requested page of list items, in list order. |
| `totalCount` | integer | yes | Total number of items in the list (before Skip/Take paging). |

### `POST /projects/current/lists/{name}/items`

`list_items_add` · tier **T1** · scope `project:edit`

Add list items.

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of string | no | Items to append to the end of the list, one entry per item. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /projects/current/tables`

`tables_list` · tier **T0** · scope `project:read`

List tables.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tables` | array of object | no | All project tables with their full definitions. |

### `POST /projects/current/tables`

`tables_add` · tier **T1** · scope `project:edit`

Add tables (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `tables` | array of object | no | Definitions of the tables to create. Each definition is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-input outcome, in the same order as the input items. |

### `PUT /projects/current/tables`

`tables_update` · tier **T1** · scope `project:edit`

Update tables (batch; rename via newName).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `tables` | array of object | no | Definitions of the tables to update, matched by Name; null fields are left unchanged. Each definition is processed independently. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `DELETE /projects/current/tables`

`tables_delete` · tier **T1** · scope `project:edit`

Delete tables by name.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `names` | array of string | no | Names of the tables to remove. Each name is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input name; a failed item does not fail the rest of the batch. |

### `GET /projects/current/tables/{name}/columns`

`table_columns_get` · tier **T0** · scope `project:read`

Table columns.

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `columns` | array of string | no | Column names in table order. Table columns are index-addressed and unnamed, so these are Excel-style letters (A, B, C, ...). |
| `hasHeaders` | boolean | yes | True when the table's first row holds column header text (IsFirstRowContainsHeaders). |
| `columnHeaders` | array of string | no | Header text taken from the first row, one entry per column. Empty when HasHeaders is false. |

### `POST /projects/current/tables/{name}/columns`

`table_column_add` · tier **T1** · scope `project:edit`

Add a column (optional header).

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `header` | string | no | Optional header text for the new column, written into the first (header) row. Allowed only when the table is marked as having a header row — otherwise the request fails with 400. On an empty table the header materializes the header row. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `columnName` | string | no | Excel-style name (A, B, C, ...) assigned to the newly added column, derived from its index — table columns are index-addressed, not named. |

### `GET /projects/current/tables/{name}/rows`

`table_rows_get` · tier **T0** · scope `project:read`

Table rows (skip/take).

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `skip` | integer | yes | Number of rows to skip from the start of the table. Default is 0. |
| `take` | integer | yes | Maximum number of rows to return. Default is 1000. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `rows` | array of array of string | no | Requested page of rows; each row is an array of cell values in the same order as Columns. When the table has a header row it is stored as part of the data and returned as the first row. |
| `columns` | array of string | no | Excel-style column names (A, B, C, ...) giving the order of cells within each row — table columns are index-addressed, not named. |
| `totalCount` | integer | yes | Total number of rows in the table (before Skip/Take paging). |

### `POST /projects/current/tables/{name}/rows`

`table_row_add` · tier **T1** · scope `project:edit`

Add a row.

**Path parameters:** `name`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `values` | array of string | no | Cell values of the row appended to the table, in column order. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /projects/current/google-spreadsheets`

`google_spreadsheets_list` · tier **T0** · scope `project:read`

List Google Spreadsheets.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `googleSpreadsheets` | array of object | no | All GoogleSpreadsheet statics of the project with their full definitions. |

### `POST /projects/current/google-spreadsheets`

`google_spreadsheets_add` · tier **T1** · scope `project:edit`

Add Google Spreadsheets (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `googleSpreadsheets` | array of object | no | Definitions of the GoogleSpreadsheet statics to create. Each definition is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-input outcome, in the same order as the input items. |

### `PUT /projects/current/google-spreadsheets`

`google_spreadsheets_update` · tier **T1** · scope `project:edit`

Update Google Spreadsheets (batch).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `googleSpreadsheets` | array of object | no | Definitions of the GoogleSpreadsheet statics to update, matched by Name; null fields are left unchanged. Each definition is processed independently. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input definition; a failed item does not fail the rest of the batch. |

### `DELETE /projects/current/google-spreadsheets`

`google_spreadsheets_delete` · tier **T1** · scope `project:edit`

Delete Google Spreadsheets by name.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `names` | array of string | no | Names of the GoogleSpreadsheet statics to remove. Each name is processed independently; per-item outcomes are returned. |
| `session` | string | no | Optional legacy session identifier kept for wire compatibility. Ignored by v1 hosts — no host reads it; authentication is carried by the API key header instead. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `items` | array of object | no | Per-item outcomes, one per input name; a failed item does not fail the rest of the batch. |

### `GET /projects/current/recording`

`recording_get` · tier **T0** · scope `project:read`

Recorder state.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `isRecording` | boolean | yes | True when ProjectMaker is currently in recording mode (user/browser actions are captured as cubes). |

### `PUT /projects/current/recording`

`recording_set` · tier **T2** · scope `project:record`

Toggle action recording.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `isRecording` | boolean | yes | Pass true to start recording, false to stop it. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /code-api`

`code_api_get` · tier **T0** · scope `code:read`

C# API surface for OwnCode (optional ?typeName=).

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `sources` | array of object | no | Index mode: the assemblies the digest was generated from. |
| `types` | array of object | no | Index mode: compact list of available types (no members). |
| `type` | object | no | Type mode: the requested type with its full member list. |

## ZennoPoster — task & session control

**Base URL:** `http://localhost:5300/api/v1 (ZennoPoster runner)`

### `GET /tasks`

`tasks_list` · tier **T0** · scope `task:read`

List tasks.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tasks` | array of object | no | All tasks currently present in the ZennoPoster task list, one summary entry each. |

### `GET /tasks/{id}`

`task_get` · tier **T0** · scope `task:read`

Task info.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | no | Task GUID. |
| `name` | string | no | Task display name. |
| `projectPath` | string | no | Path to the referenced project file. |
| `isSchedulerOwned` | boolean | yes | True when the task is owned by a scheduler job (created via the scheduler wizard): such tasks cannot be deleted or edited directly through the API. |
| `createTime` | string | no | Task creation time (ISO-8601). |
| `settingsType` | string | no | Settings block kind of the referenced project: "BotUI" \| "InputSettings" \| "None" . GET/PUT /tasks/{id}/config serves BOTH kinds (the unified settings view). Can be empty until the project has been scanned at least once. |
| `browserType` | string | no | Browser type stored on the task, or null when unset. |

### `POST /tasks`

`task_create` · tier **T3** · scope `task:edit`

Add a task — imports a project whose code (incl. OwnCode) runs on start (RCE class).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | string | no | Absolute path to an existing project file (.zp/.zpproj/.vsproj). |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | no | GUID assigned to the newly imported task; use it to address the task in all other endpoints. |
| `name` | string | no | Display name of the created task, taken from the imported project. |

### `DELETE /tasks/{id}`

`task_delete` · tier **T1** · scope `task:edit`

Remove a task (interrupts running threads first).

**Path parameters:** `id`.

**Responses:** `204` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` task_scheduler_owned · `500` internal_error

### `POST /tasks/{id}/start`

`task_start` · tier **T3** · scope `task:control`

Start a task — executes project code incl. OwnCode (RCE class).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | string | no | Outcome of the control operation: "started" \| "stopped" \| "interrupted". |
| `id` | string | no | GUID of the task the operation acted on. |

### `POST /tasks/{id}/stop`

`task_stop` · tier **T2** · scope `task:control`

Stop a task.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | string | no | Outcome of the control operation: "started" \| "stopped" \| "interrupted". |
| `id` | string | no | GUID of the task the operation acted on. |

### `POST /tasks/{id}/interrupt`

`task_interrupt` · tier **T2** · scope `task:control`

Interrupt a task.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | string | no | Outcome of the control operation: "started" \| "stopped" \| "interrupted". |
| `id` | string | no | GUID of the task the operation acted on. |

### `GET /tasks/{id}/threads/max`

`task_threads_max_get` · tier **T0** · scope `task:read`

Thread limit (LimitOfThreads).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `maxThreads` | integer | yes | The task's thread limit, always >= 1. On PUT this is the value actually applied, re-read after the commit. |

### `PUT /tasks/{id}/threads/max`

`task_threads_max_set` · tier **T1** · scope `task:edit`

Set the thread limit.

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `maxThreads` | integer | yes | New thread limit for the task. Must be >= 1; rejected with 400 otherwise. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `maxThreads` | integer | yes | The task's thread limit, always >= 1. On PUT this is the value actually applied, re-read after the commit. |

### `GET /tasks/{id}/tries`

`task_tries_get` · tier **T0** · scope `task:read`

Queued tries (NumberOfTries).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tries` | integer | yes | The queued-tries counter, always >= 0. On PUT this is the value actually applied, re-read after the commit. |

### `POST /tasks/{id}/tries/add`

`task_tries_add` · tier **T1** · scope `task:edit`

Add tries — starts a ready task (gated like task_start when it would start).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `count` | integer | yes | Number of run tries to add to the task's queue. Must be >= 1 (use PUT /tasks/{id}/tries to lower the queue); rejected with 400 otherwise. Adding tries to a ready task starts it. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `totalTries` | integer | yes | The queued-tries counter after the add, re-read from the task (actually applied, not an echo). |
| `addedTries` | integer | yes | Number of tries this call added (echoes the request's count). |

### `PUT /tasks/{id}/tries`

`task_tries_set` · tier **T1** · scope `task:edit`

Set tries — starts a ready task (gated like task_start when it would start).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `tries` | integer | yes | New value for the queued-tries counter. Must be >= 0; rejected with 400 otherwise. Setting a positive value on a ready task starts it. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tries` | integer | yes | The queued-tries counter, always >= 0. On PUT this is the value actually applied, re-read after the commit. |

### `GET /tasks/{id}/settings/execution`

`task_execution_get` · tier **T0** · scope `task:read`

Execution settings.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `limitOfThreads` | integer | no | Thread limit for the task, at least 1. |
| `numberOfTries` | integer | no | Number of tries to queue, 0 or more. A non-zero value on a ready task starts it — gated like task_start. |
| `priority` | integer | no | Priority: 10 low, 50 medium, 100 high, 100000 critical. |
| `proxy` | string | no | Proxy usage mode, one of: DoNotUseProxy, IfPossible, UseProxyWithoutRemove, UseProxy. |
| `proxyLabels` | string | no | Comma-separated proxy labels. |
| `groupLabels` | string | no | Comma-separated group labels. |
| `maxNumOfSuccessStop` | integer | no | Stop after N successes: -1 = unlimited, otherwise at least 1. |
| `maxNumOfFailStop` | integer | no | Stop after N consecutive failures: -1 = unlimited, otherwise at least 1. |
| `timeout` | integer | no | Per-try timeout in minutes: -1 = none, otherwise at least 1. |
| `traceTask` | boolean | no | Trace the task. |
| `performBadEndOnInterrupt` | boolean | no | Run the BadEnd branch on project interrupt. |

### `PUT /tasks/{id}/settings/execution`

`task_execution_set` · tier **T1** · scope `task:edit`

Update execution settings (merge semantics; may start a ready task — gated like task_start).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `limitOfThreads` | integer | no | Thread limit for the task, at least 1. |
| `numberOfTries` | integer | no | Number of tries to queue, 0 or more. A non-zero value on a ready task starts it — gated like task_start. |
| `priority` | integer | no | Priority: 10 low, 50 medium, 100 high, 100000 critical. |
| `proxy` | string | no | Proxy usage mode, one of: DoNotUseProxy, IfPossible, UseProxyWithoutRemove, UseProxy. |
| `proxyLabels` | string | no | Comma-separated proxy labels. |
| `groupLabels` | string | no | Comma-separated group labels. |
| `maxNumOfSuccessStop` | integer | no | Stop after N successes: -1 = unlimited, otherwise at least 1. |
| `maxNumOfFailStop` | integer | no | Stop after N consecutive failures: -1 = unlimited, otherwise at least 1. |
| `timeout` | integer | no | Per-try timeout in minutes: -1 = none, otherwise at least 1. |
| `traceTask` | boolean | no | Trace the task. |
| `performBadEndOnInterrupt` | boolean | no | Run the BadEnd branch on project interrupt. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | no | Short outcome word for the operation, e.g. "imported", "updated" or "cleared". |
| `id` | string | no | GUID of the affected task when the operation targets one; null otherwise. |

### `GET /tasks/{id}/settings/scheduler`

`task_scheduler_get` · tier **T0** · scope `task:read`

Scheduler settings.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `isActive` | boolean | yes | Scheduler enabled. Required. Activating the schedule is a deferred start of the project's code, so isActive:true runs the same content-classification gate as task_start and requires a real executePeriod (400 on NULL). |
| `executePeriod` | string | no | Execution period name: OneTime \| EveryDay \| EveryWeek \| EveryMonth \| OnDemand \| NULL (no schedule). Required. |
| `daysOfWeek` | array of string | no | Day names for EveryWeek: English day names, "Monday" … "Sunday" (other casings are normalized; unknown names are rejected with 400). |
| `daysOfMonth` | string | no | Days of month for EveryMonth, in the scheduler's human-view form (e.g. "1,5,10"). |
| `fileName` | string | no | Trigger file path for OnDemand. |
| `isNeedDeleteFile` | boolean | yes | Delete the trigger file after firing, for OnDemand. |
| `startDateType` | string | no | Start-date type: Immediately \| OnDate. |
| `startDate` | string | no | Start date when StartDateType == OnDate: an invariant-culture date-time string in the MACHINE'S LOCAL TIME, no time zone (e.g. "08/30/2026 09:00:00") — a deliberate exception to the API-wide absolute-UTC convention, because the scheduler runs on the machine's clock. |
| `attemptsRange` | string | no | Attempts per run: "n" or "a-b" (SchedulerAttempts.AttemptsRange human view). Required. |
| `isClearSuccess` | boolean | yes | Clear success counter each run. Required. |
| `intervals` | array of string | no | Time-of-day intervals as "HH:mm-HH:mm". An end of "00:00" means 24:00 (end of day); a single time ("09:30") is a point interval. Optional. |
| `stopExecutionOutsideOfIntervals` | boolean | yes | Stop execution outside the intervals. |
| `repeatType` | string | no | Repeat type: Continued \| ContinuedWithPause \| Regularly \| Count. |
| `repeatCountDayRange` | string | no | Per-day repeat count "n" or "a-b"; required unless Continued. |
| `endDateType` | string | no | End-date type: Infinity \| OnDate \| Count. |
| `endDate` | string | no | End date when EndDateType == OnDate — same format and local-time semantics as StartDate. |
| `repeatCountTotalRange` | string | no | Total repeat count "n" or "a-b" when EndDateType == Count. |

### `PUT /tasks/{id}/settings/scheduler`

`task_scheduler_set` · tier **T1** · scope `task:edit`

Update scheduler settings (activation gated like task_start).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `isActive` | boolean | yes | Scheduler enabled. Required. Activating the schedule is a deferred start of the project's code, so isActive:true runs the same content-classification gate as task_start and requires a real executePeriod (400 on NULL). |
| `executePeriod` | string | no | Execution period name: OneTime \| EveryDay \| EveryWeek \| EveryMonth \| OnDemand \| NULL (no schedule). Required. |
| `daysOfWeek` | array of string | no | Day names for EveryWeek: English day names, "Monday" … "Sunday" (other casings are normalized; unknown names are rejected with 400). |
| `daysOfMonth` | string | no | Days of month for EveryMonth, in the scheduler's human-view form (e.g. "1,5,10"). |
| `fileName` | string | no | Trigger file path for OnDemand. |
| `isNeedDeleteFile` | boolean | yes | Delete the trigger file after firing, for OnDemand. |
| `startDateType` | string | no | Start-date type: Immediately \| OnDate. |
| `startDate` | string | no | Start date when StartDateType == OnDate: an invariant-culture date-time string in the MACHINE'S LOCAL TIME, no time zone (e.g. "08/30/2026 09:00:00") — a deliberate exception to the API-wide absolute-UTC convention, because the scheduler runs on the machine's clock. |
| `attemptsRange` | string | no | Attempts per run: "n" or "a-b" (SchedulerAttempts.AttemptsRange human view). Required. |
| `isClearSuccess` | boolean | yes | Clear success counter each run. Required. |
| `intervals` | array of string | no | Time-of-day intervals as "HH:mm-HH:mm". An end of "00:00" means 24:00 (end of day); a single time ("09:30") is a point interval. Optional. |
| `stopExecutionOutsideOfIntervals` | boolean | yes | Stop execution outside the intervals. |
| `repeatType` | string | no | Repeat type: Continued \| ContinuedWithPause \| Regularly \| Count. |
| `repeatCountDayRange` | string | no | Per-day repeat count "n" or "a-b"; required unless Continued. |
| `endDateType` | string | no | End-date type: Infinity \| OnDate \| Count. |
| `endDate` | string | no | End date when EndDateType == OnDate — same format and local-time semantics as StartDate. |
| `repeatCountTotalRange` | string | no | Total repeat count "n" or "a-b" when EndDateType == Count. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | no | Short outcome word for the operation, e.g. "imported", "updated" or "cleared". |
| `id` | string | no | GUID of the affected task when the operation targets one; null otherwise. |

### `POST /tasks/{id}/success/clear`

`task_success_clear` · tier **T1** · scope `task:edit`

Reset success counter.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | no | Short outcome word for the operation, e.g. "imported", "updated" or "cleared". |
| `id` | string | no | GUID of the affected task when the operation targets one; null otherwise. |

### `POST /tasks/{id}/fails/clear`

`task_fails_clear` · tier **T1** · scope `task:edit`

Reset fail counter.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | no | Short outcome word for the operation, e.g. "imported", "updated" or "cleared". |
| `id` | string | no | GUID of the affected task when the operation targets one; null otherwise. |

### `GET /tasks/{id}/state`

`task_state_get` · tier **T0** · scope `task:read`

Runtime state.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | no | Task GUID. |
| `status` | string | no | Task status name, one of: Newbie (never ran), Complete, NotComplete, Schedule (waiting for the scheduler), WaitPerform (queued to run), Perform (running), WaitProxy, Stopping, Stop, Fail. |
| `doneSuccessfully` | integer | yes | Successful completions so far. |
| `doneAll` | integer | yes | Total completions so far. |
| `numberOfTries` | integer | yes | Remaining tries queued. |
| `numOfFailStop` | integer | yes | Consecutive failures counter. |
| `countOfThreads` | integer | yes | Currently running threads for the task. |
| `limitOfThreads` | integer | yes | The user-set thread limit — same value as GET /tasks/{id}/threads/max. |
| `maxAllowOfThreads` | integer | yes | Read-only system thread ceiling: written by the core on every project scan (and by ZennoBox for purchased templates), not settable through the API. |
| `failurePoints` | array of object | no | SNAPSHOT of the last observed failure per worker thread/instance (hard stop or soft OnError-continued) — NOT an event queue: reading is non-destructive and repeated GETs return the same entries. An entry is overwritten in place when its thread fails again (keyed by taskId+threadId), expires after 2 hours, is capped at 64 entries per task, and is cleared only when the task is deleted. Deduplicate client-side by (threadId, timestamp) or uowId. The variables snapshot inside may contain sensitive values (readable at T0 task:read). |

### `GET /tasks/{id}/config`

`task_config_get` · tier **T0** · scope `task:read`

User settings (unified: InputSettings and BotUI).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `settings` | array of object | no | The task's user settings. On PUT this is a batch merge by setting name: names that match a setting of the task are applied, unknown names are skipped (see InputSettingsApplyResponse). |

### `PUT /tasks/{id}/config`

`task_config_set` · tier **T1** · scope `task:edit`

Update user settings (reports applied/skipped names).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `settings` | array of object | no | The task's user settings. On PUT this is a batch merge by setting name: names that match a setting of the task are applied, unknown names are skipped (see InputSettingsApplyResponse). |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `applied` | array of string | no | Setting names (with their resolved variable) that matched and were applied. |
| `skipped` | array of string | no | Setting names that matched no setting of the task and were skipped. |

### `GET /sessions`

`sessions_list` · tier **T0** · scope `task:read`

Open WaitForUserAction windows.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `sessions` | array of object | no | The requested page of open windows, ordered by open time. |
| `total` | integer | yes | Total number of open windows matching the filter, before paging. |

### `GET /sessions/{id}`

`session_get` · tier **T0** · scope `task:read`

Window details.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` session_not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `sessionId` | string | no | Opaque session id, minted when the window opens. |
| `taskId` | string | no | Task GUID the window belongs to. |
| `projectName` | string | no | Project name of the running task. |
| `instanceId` | integer | yes | Instance port, the id used to address the Instance contract. |
| `started` | string | no | UTC timestamp (ISO-8601) the window opened. |
| `prompt` | string | no | Comment the project passed to WaitForUserAction. Empty when none. |
| `timeoutSeconds` | integer | yes | Timeout in seconds set on the action. Zero when there is no timeout. |
| `deadline` | string | no | UTC timestamp (ISO-8601) when the window expires. Empty when there is no timeout. |

### `GET /sessions/events`

`sessions_events` · tier **T0** · scope `task:read`

Long-poll: window open/close events (?timeoutSeconds=N).

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `events` | array of object | no | Events collected during this poll, oldest first. Empty when the poll timed out. |
| `timedOut` | boolean | yes | True when the timeout elapsed with no events (the events list is empty). |

### `POST /sessions/{id}/complete`

`session_complete` · tier **T2** · scope `task:control`

Complete the action, resume thread (success branch).

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `summary` | string | no | Optional free-text summary of what was done, for the audit trail. Does not affect how the thread resumes. |
| `metadata` | object | no | Optional arbitrary key/value payload accompanying the completion, for the audit trail. Does not affect how the thread resumes. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` session_not_found · `409` no_active_interaction · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `sessionId` | string | no | Id of the session that was completed. |
| `status` | string | no | Always "completed" — the only terminal state this endpoint produces. |
| `threadResumedOn` | string | no | Branch the paused task thread resumed on. Always "success": there is no fail/cancel branch yet. |

## Instance — browser instances, tabs & elements

**Base URL:** `http://localhost:5299 or :5300/api/v1 (the host process that owns the browser)`

### `GET /instances`

`instances_list` · tier **T0** · scope `instance:read`

List active instances.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `instances` | array of object | no | The running browser instances the host exposes (a single entry on a ProjectMaker host). |

### `POST /instances`

`instance_create` · tier **T2** · scope `instance:control`

Start/get an instance.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `browserType` | string | no | Browser engine to launch — a BrowserType enum name, case-insensitive (e.g. "Chromium", "Firefox45"). Null/empty falls back to the host default (Chromium); an unrecognized value is rejected with 400. |
| `browserArgs` | string | no | Extra command-line arguments passed to the browser process. Null/empty for none. |
| `isolated` | boolean | yes | Launch the browser in its own dedicated base process instead of sharing one with other instances — the same "isolated process" option a project's static settings offer. Only Firefox45 can actually share a base process, so this flag matters for Firefox45 alone; every other browser type (Chromium included) is single-instance and gets a dedicated process regardless of this flag. Honored by the ZennoPoster runner; the ProjectMaker host's singleton editor instance ignores it. |
| `ownerUowId` | string | no | SERVER-SET: the unit of work the creating caller acts for, taken from the X-Zenno-Uow-Id header by the Instance host (a client-supplied value is overwritten). The ZennoPoster runner records the new browser as owned by this unit of work, so the AI action's self-started browsers get the same cross-task protection as task browsers; null (no header) leaves the browser unowned, as before. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `browserType` | string | no | Browser engine of the instance — a BrowserType enum name, e.g. "Chromium" or "Firefox45". |
| `id` | integer | yes | Port of the browser instance; the single ProjectMaker instance has a real port too. |
| `profileDir` | string | no | Path to the browser profile the instance runs with. |

### `GET /instances/{id}`

`instance_get` · tier **T0** · scope `instance:read`

Instance metadata.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `browserType` | string | no | Browser engine of the instance — a BrowserType enum name, e.g. "Chromium" or "Firefox45". |
| `id` | integer | yes | Port of the browser instance; the single ProjectMaker instance has a real port too. |
| `profileDir` | string | no | Path to the browser profile the instance runs with. |

### `POST /instances/{id}/show`

`instance_show` · tier **T1** · scope `instance:control`

Show the window (409 when the view is protected).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` instance_view_protected · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `POST /instances/{id}/hide`

`instance_hide` · tier **T1** · scope `instance:control`

Hide the window.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `DELETE /instances/{id}`

`instance_release` · tier **T1** · scope `instance:control`

Release the instance back to the pool (the pool decides restart-vs-close; not a forced close).

**Path parameters:** `id`.

**Responses:** `204` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` instance_busy · `500` internal_error

### `GET /instances/{id}/tabs`

`tabs_list` · tier **T0** · scope `instance:read`

List tabs.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tabs` | array of object | no | All open tabs of the instance. |

### `POST /instances/{id}/tabs`

`tab_create` · tier **T1** · scope `instance:interact`

New tab.

**Path parameters:** `id`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `tabName` | string | no | Name for the new tab. |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tabName` | string | no | Display name of the tab. |
| `tabId` | integer | yes | Tab id. 0 is reserved as the "active tab" sentinel. |

### `POST /instances/{id}/tabs/{tabId}/activate`

`tab_activate` · tier **T1** · scope `instance:interact`

Activate tab.

**Path parameters:** `id`, `tabId`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `DELETE /instances/{id}/tabs/{tabId}`

`tab_close` · tier **T1** · scope `instance:interact`

Close tab.

**Path parameters:** `id`, `tabId`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `POST /instances/{id}/tabs/{tabId}/navigate`

`tab_navigate` · tier **T1** · scope `instance:interact`

Navigate to URL.

**Path parameters:** `id`, `tabId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `url` | string | no | Absolute URL to navigate the tab to. |
| `timeout` | integer | yes | Page-load timeout in seconds applied to this navigation (sets the tab's NavigateTimeout). |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `GET /instances/{id}/tabs/{tabId}/dom`

`tab_dom_get` · tier **T0** · scope `instance:read`

DOM text (tag filter, pagination).

**Path parameters:** `id`, `tabId`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `409` dom_unavailable · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `content` | string | no | The requested chunk of the (optionally tag-filtered) DOM text, starting at the requested startIndex. |
| `totalLength` | integer | yes | Total length in characters of the full filtered text, across all chunks. |
| `nextIndex` | integer | no | Offset to pass as startIndex on the next call, or null if there is no more content. |
| `isTruncated` | boolean | yes | True when more content remains after this chunk — continue from NextIndex. |

### `GET /instances/{id}/tabs/{tabId}/elements/attribute`

`element_attribute_get` · tier **T0** · scope `instance:read`

Get attribute by XPath.

**Path parameters:** `id`, `tabId`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `value` | string | no | Current value of the requested attribute; null when the element has no such attribute. |

### `PUT /instances/{id}/tabs/{tabId}/elements/attribute`

`element_attribute_set` · tier **T1** · scope `instance:interact`

Set attribute by XPath.

**Path parameters:** `id`, `tabId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `xPath` | string | no | XPath selecting the target element. |
| `attributeName` | string | no | Attribute to set. "value" (case-insensitive) is special: the text is typed via real keyboard emulation, firing trusted keydown/input/change events and interpreting tokens like {ENTER}; any other attribute is set directly on the element. |
| `attributeValue` | string | no | Value to assign — the text to type, when the attribute is "value". |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

### `POST /instances/{id}/tabs/{tabId}/elements/event`

`element_event` · tier **T1** · scope `instance:interact`

Raise a DOM event (click/input/submit/...).

**Path parameters:** `id`, `tabId`.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `xPath` | string | no | XPath selecting the target element. |
| `eventName` | string | no | DOM event to raise on the element, e.g. "click", "input", "change". A click additionally requires the element to be actually rendered (visible, non-zero size). |

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the mutation was performed (errors surface as non-2xx ApiError responses instead of false). |

## Android — ZennoDroid device

**Base URL:** `http://localhost:5300/api/v1 (ZennoDroid runner)`

### `GET /android/devices`

`devices_list` · tier **T0** · scope `instance:read`

List the devices this process drives.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `devices` | array of object | no | The devices this process drives (a single entry on a ProjectMaker host). |

### `POST /android/devices/{id}/start`

`device_start` · tier **T2** · scope `instance:control`

Start/attach the device and wait until it is ready.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `success` | boolean | yes | True when the device came up and is ready for work. |

### `GET /android/devices/{id}`

`device_get` · tier **T0** · scope `instance:read`

Device metadata (id, name, running state).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Device id on the wire — the UI port used by the device instance window. |
| `name` | string | no | Device name that uniquely identifies the device, e.g. "emulator-5554" or the phone's serial. |
| `title` | string | no | Display title of the device as shown in Device Manager. |
| `isRunning` | boolean | yes | True when the device is ready for work: the screen is being captured and the device is started and connected (emulator) or connected (real phone). |

### `GET /android/devices/{id}/screen`

`device_screen_get` · tier **T0** · scope `instance:read`

Real screen resolution in pixels.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `width` | integer | yes | Screen width in physical pixels. |
| `height` | integer | yes | Screen height in physical pixels. |

### `GET /android/devices/{id}/source`

`device_source_get` · tier **T0** · scope `instance:read`

Current UI tree (raw page source).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `source` | string | no | Raw XML page source; empty when the device exposes none right now. |

### `GET /android/devices/{id}/apps`

`device_apps_get` · tier **T0** · scope `instance:read`

Applications installed on the device.

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `applications` | string | no | Driver-reported application listing, verbatim. |

## Cross-cutting — auth, audit & capabilities

**Base URL:** `served by every host on its own PublicApi port/api/v1`

### `POST /auth/keys`

`auth_keys_create` · tier **T3** · scope `admin`

Issue an ApiKey (raw key shown once).

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `label` | string | no | Optional human-readable label to identify the key in listings and the UI. |
| `scopes` | array of string | no | Scopes to grant, e.g. "task:read", "project:edit", "instance:interact" (see the ApiScope constants). A request exceeding the host's policy ceiling is rejected with 403 policy_violation. |
| `maxTier` | enum(T0 \| T1 \| T2 \| T3) | yes | Highest risk tier the key may invoke: T0 (read) .. T3 (host/RCE). Defaults to T0. |
| `expiresAt` | string | no | Absolute expiry (ISO-8601 with offset on the wire); null = non-expiring. A value in the past is rejected with 400. |

**Responses:** `201` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (201):**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | no | Stable key id used to address the key in later calls (list/revoke/enable); not a secret. |
| `label` | string | no | Human-readable label copied from the issue request; null when none was given. |
| `rawKey` | string | no | The secret API key itself. Returned only in this response — it is not stored and cannot be retrieved again. |
| `scopes` | array of string | no | Scopes granted to the key, e.g. "task:read" (see the ApiScope constants). |
| `maxTier` | enum(T0 \| T1 \| T2 \| T3) | yes | Highest risk tier the key may invoke: T0 (read) .. T3 (host/RCE). |
| `expiresAt` | string | no | Absolute expiry of the key; null for a non-expiring key. |
| `enabled` | boolean | yes | True when the key can authenticate; a freshly issued key is always enabled. |

### `GET /auth/keys`

`auth_keys_list` · tier **T3** · scope `admin`

List keys (no raw key).

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

_(empty object)_

### `DELETE /auth/keys/{id}`

`auth_keys_revoke` · tier **T3** · scope `admin`

Revoke a key (immediate).

**Path parameters:** `id`.

**Responses:** `204` success · `400` bad_request · `401` unauthorized · `403` forbidden · `404` not_found · `500` internal_error

### `GET /auth/whoami`

`auth_whoami` · tier **T0** · scope `—`

Scopes/tier of the current key.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `keyId` | string | no |  |
| `label` | string | no |  |
| `scopes` | array of string | no |  |
| `maxTier` | integer | yes |  |
| `expiresAt` | string | no |  |

### `GET /audit`

`audit_get` · tier **T0** · scope `admin`

Audit log of calls.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `403` forbidden · `500` internal_error

**Success body (200):**

_(empty object)_

### `GET /capabilities`

`capabilities` · tier **T0** · scope `—`

Manifest: operations, tiers, required scopes.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `500` internal_error

**Success body (200):**

| Field | Type | Required | Description |
|---|---|---|---|
| `host` | string | no | Host identity, e.g. "projectmaker" / "zennoposter". |
| `version` | string | no | Contract semver (ContractVersion). |
| `currentScopes` | array of string | no | Scopes granted to the current key. |
| `currentMaxTier` | integer | yes | Highest tier the current key may invoke (0..3). |
| `operations` | array of object | no | Every operation of the host's domains, one entry each — including operations the current key cannot invoke (see IsAvailable). |
| `tierLegend` | array of object | no | Static legend of the tier taxonomy for client display. |

### `GET /confirmations`

`confirmations_list` · tier **T0** · scope `—` · **not implemented (501)**

Pending HITL confirmations.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `500` internal_error

### `POST /confirmations/{id}/approve`

`confirmation_approve` · tier **T0** · scope `—` · **not implemented (501)**

Approve an operation (UI only).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `500` internal_error

### `POST /confirmations/{id}/reject`

`confirmation_reject` · tier **T0** · scope `—` · **not implemented (501)**

Reject an operation (UI only).

**Path parameters:** `id`.

**Responses:** `200` success · `400` bad_request · `401` unauthorized · `500` internal_error
