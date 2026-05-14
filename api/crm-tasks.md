# CRM API — Tasks

**Base prefix:** `/api/v1/`
**Authentication:** All endpoints require `Authorization: Bearer <token>`.
**Trailing slashes:** Required on all paths.

---

## Overview

Tasks are the primary work items inside CRM boards. Each task belongs to exactly one board and one column. Tasks have a `position` (1-based order within a column), a priority level, an optional deadline, and an optional assignee. Tasks can be archived without deletion and carry labels, checklists, comments, and file attachments.

There are two overlapping access patterns for tasks:

- **Board-scoped** — `GET /crm/boards/{boardId}/tasks/` and `GET /crm/boards/{boardId}/tasks/{taskId}/` — used by the Kanban board view.
- **Global** — `GET /crm/tasks/`, `GET /crm/tasks/{id}/`, `PATCH /crm/tasks/{id}/`, etc. — used by task detail pages and cross-board views.

Both patterns return the same `CrmTask` shape.

---

## Shared Response Shape — CrmTask

```json
{
  "id": 42,
  "board": {
    "id": 7,
    "name": "Q3 Sprint",
    "description": null,
    "is_archived": false,
    "created_at": "2025-05-14T10:00:00Z",
    "updated_at": "2025-05-14T10:00:00Z",
    "company": 3
  },
  "column_id": 13,
  "title": "Implement login flow",
  "description": "Cover OAuth 2.0 and email/password paths.",
  "priority": "high",
  "deadline": "2025-06-01",
  "assignee": {
    "id": 9,
    "first_name": "Asel",
    "last_name": "Nurova",
    "avatar": "https://storage.example.com/avatars/9.jpg"
  },
  "label_ids": [1, 3],
  "labels": [
    { "id": 1, "name": "Bug", "color": "#ef4444" },
    { "id": 3, "name": "Backend", "color": "#6366f1" }
  ],
  "comments_count": 4,
  "attachments_count": 2,
  "position": 2,
  "is_archived": false,
  "created_at": "2025-05-14T10:05:00Z",
  "checklists": [
    {
      "id": 5,
      "title": "Acceptance criteria",
      "items": [
        { "id": 11, "text": "JWT issued on success", "is_completed": true, "order": 1 },
        { "id": 12, "text": "Refresh token rotated", "is_completed": false, "order": 2 }
      ],
      "checklist_progress": { "total": 2, "completed": 1 }
    }
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Task ID |
| `board` | `CrmBoard` | Full nested board object |
| `column_id` | `integer` | Current column ID |
| `title` | `string` | Max 255 characters |
| `description` | `string \| null` | Free-text |
| `priority` | `"low" \| "medium" \| "high" \| "critical"` | |
| `deadline` | `string \| null` | ISO date `"YYYY-MM-DD"` |
| `assignee` | `{ id, first_name, last_name, avatar? } \| null` | |
| `label_ids` | `integer[]` | IDs of applied labels |
| `labels` | `CrmLabel[]` | Full label objects |
| `comments_count` | `integer` | Denormalized count |
| `attachments_count` | `integer` | Denormalized count |
| `position` | `integer` | 1-based order within the column |
| `is_archived` | `boolean` | |
| `created_at` | `string` | ISO 8601 datetime |
| `checklists` | `CrmChecklist[]` | Embedded; see `crm-task-actions.md` for full shape |

---

## Endpoints

### GET /crm/boards/{boardId}/tasks/

Retrieve all tasks belonging to a specific board, with optional filtering. Used by the Kanban board view.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `boardId` | `integer \| string` | Board ID |

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `board_id` | `integer` | Redundant when using the board-scoped path; ignored if path `boardId` is present. |
| `search` | `string` | Case-insensitive substring match on `title`. |
| `priority` | `string` | One of `low`, `medium`, `high`, `critical`. |
| `deadline` | `string` | One of `overdue`, `today`, `this_week`. |
| `ordering` | `string` | Field to sort by. Prefix with `-` for descending (e.g. `-created_at`). |
| `is_archived` | `boolean` | Pass `true` to return only archived tasks (used by the archive panel). |
| `view` | `string` | `"list"` signals the backend that the request is for the flat list view; may affect serialization. |

**Response** `200 OK` — array or `{ results: CrmTask[], count?: number }`.

The frontend filters out `is_archived: true` tasks client-side for the Kanban view; both active and archived are returned when the query includes `is_archived=true`.

**Note on pagination:** The board task list endpoint may or may not paginate. The frontend handles both a plain array and a `{ results }` envelope. For pagination on a specific board, use `GET /crm/tasks/` with `board_id` and `page`/`page_size` parameters.

---

### GET /crm/boards/{boardId}/tasks/{taskId}/

Retrieve a single task scoped to a board.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `200 OK` — single `CrmTask` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found or does not belong to the specified board. |

---

### GET /crm/tasks/

Global task list. Supports the same filter parameters as the board-scoped endpoint plus explicit `board_id` and pagination parameters.

**Roles:** `company_admin`, `employee`, `superadmin`

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `board_id` | `integer` | Filter by board. Required when fetching tasks for a specific board via this endpoint. |
| `search` | `string` | Substring match on `title`. |
| `priority` | `string` | One of `low`, `medium`, `high`, `critical`. |
| `deadline` | `string` | One of `overdue`, `today`, `this_week`. |
| `ordering` | `string` | Sort field; prefix with `-` for descending. |
| `is_archived` | `boolean` | Include archived tasks only. |
| `page` | `integer` | Page number (1-based). |
| `page_size` | `integer` | Items per page. |

**Response** `200 OK`

```json
{
  "count": 120,
  "next": "https://api.example.com/api/v1/crm/tasks/?page=2",
  "previous": null,
  "results": [ /* CrmTask[] */ ]
}
```

---

### POST /crm/tasks/

Create a new task.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `board_id` | `integer` | Yes | Target board |
| `column_id` | `integer` | Yes | Target column; task is appended at the end of the column |
| `title` | `string` | Yes | Max 255 characters |
| `priority` | `string` | Yes | One of `low`, `medium`, `high`, `critical` |
| `description` | `string` | No | Free-text |
| `deadline` | `string` | No | ISO date `"YYYY-MM-DD"` |
| `assignee_id` | `integer` | No | User ID of the assignee |

```json
{
  "board_id": 7,
  "column_id": 12,
  "title": "Write API documentation",
  "priority": "medium",
  "description": "Cover all CRM endpoints.",
  "deadline": "2025-06-15",
  "assignee_id": 9
}
```

**Response** `201 Created` — the created `CrmTask` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Missing required fields; WIP limit exceeded on the target column. Response body: `{ "detail": "..." }` (contains "wip" when WIP-blocked). |
| `403` | Insufficient role. |
| `404` | Board or column not found. |

---

### GET /crm/tasks/my/

Return tasks assigned to the authenticated user, grouped by board. This endpoint does **not** return a flat paginated list — it returns a structured grouped response.

**Roles:** `company_admin`, `employee`, `superadmin`

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `search` | `string` | Substring match on task `title`. |
| `priority` | `string` | One of `low`, `medium`, `high`, `critical`. |
| `deadline` | `string` | One of `overdue`, `today`, `this_week`. |
| `ordering` | `string` | Sort field (e.g. `-created_at`). Default: `-created_at`. |

**Response** `200 OK`

```json
{
  "groups": [
    {
      "board_id": 7,
      "board_name": "Q3 Sprint",
      "tasks": [ /* CrmTask[] — initial page of tasks for this board */ ],
      "total": 12,
      "has_more": true
    },
    {
      "board_id": 9,
      "board_name": "Support",
      "tasks": [ /* CrmTask[] */ ],
      "total": 3,
      "has_more": false
    }
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `groups` | `MyTaskGroup[]` | One entry per board that has assigned tasks |
| `groups[].board_id` | `integer` | |
| `groups[].board_name` | `string` | |
| `groups[].tasks` | `CrmTask[]` | Initial batch of tasks for this board |
| `groups[].total` | `integer` | Total assigned tasks on this board (across all pages) |
| `groups[].has_more` | `boolean` | Whether additional tasks exist beyond the initial batch |

**Loading more tasks for a group:** Use `GET /crm/tasks/?board_id={board_id}&page=2&page_size=50` — that standard paginated endpoint is used for "load more" per group.

---

### GET /crm/tasks/{id}/

Retrieve a single task by ID. Used by the standalone task detail page.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Response** `200 OK` — single `CrmTask` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

### PATCH /crm/tasks/{id}/

Update one or more fields of an existing task. Only send fields you want to change.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Request Body** `application/json`

| Field | Type | Notes |
|---|---|---|
| `title` | `string` | Max 255 characters |
| `description` | `string \| null` | Pass `null` to clear |
| `priority` | `string` | One of `low`, `medium`, `high`, `critical` |
| `deadline` | `string \| null` | ISO date `"YYYY-MM-DD"`; pass `null` to clear |
| `assignee_id` | `integer \| null` | User ID; pass `null` to unassign |
| `label_ids` | `integer[]` | Complete replacement of the task's labels. Send the full desired set. |

```json
{
  "priority": "critical",
  "deadline": "2025-05-20",
  "assignee_id": null
}
```

**Assigning labels:** Use `label_ids` — the backend replaces all current labels with the provided set.

```json
{
  "label_ids": [1, 3, 5]
}
```

**Response** `200 OK` — updated `CrmTask` object.

**Side effects:** A history entry is created for each changed field (`action: "updated"`, `field_name` identifies the field). Relevant notifications may be emitted.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Validation failure (e.g. blank `title`, unknown `priority`). |
| `403` | Insufficient role. |
| `404` | Task not found. |

---

### DELETE /crm/tasks/{id}/

Permanently delete a task.

**Roles:** `company_admin`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Response** `204 No Content`

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Insufficient role. |
| `404` | Task not found. |

---

### POST /crm/tasks/{id}/move/

Move a task to a different column and/or position within a column. This is the drag-and-drop persistence call — it is called once per completed drag, after the UI has already applied an optimistic update.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `column_id` | `integer` | Yes | Target column ID (may be the same column) |
| `order` | `integer` | Yes | 1-based position within the target column after the move |

```json
{
  "column_id": 13,
  "order": 2
}
```

**Response** `200 OK` — updated `CrmTask` object, or `{}`.

**WIP limit:** If the target column has a WIP limit that would be exceeded by the move, the backend returns `400` with a `detail` or `non_field_errors` message containing "wip". The frontend detects this and rolls back the optimistic UI update.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | WIP limit exceeded on target column. Response: `{ "detail": "..." }` or `{ "non_field_errors": ["..."] }`. |
| `403` | Insufficient role. |
| `404` | Task or column not found. |

---

### POST /crm/tasks/{id}/archive/

Archive a task. Archived tasks are excluded from the Kanban board view but remain accessible via `?is_archived=true` and in the archive panel.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Request Body:** None.

**Response** `200 OK` — updated `CrmTask` object with `"is_archived": true`.

**Side effects:** A history entry is created with `action: "archived"`.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

### POST /crm/tasks/{id}/unarchive/

Restore a task from archive back to its original column.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Task ID |

**Request Body:** None.

**Response** `200 OK` — updated `CrmTask` object with `"is_archived": false`.

**WIP limit:** If the column the task was originally in now has a full WIP limit, the backend should return `400`. The frontend checks the WIP limit client-side before calling this endpoint and blocks the action with a toast message.

**Side effects:** A history entry is created with `action: "unarchived"`.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | WIP limit exceeded on the original column. |
| `404` | Task not found. |
