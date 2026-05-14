# CRM API — Boards, Columns, and Labels

**Base prefix:** `/api/v1/`
**Authentication:** All endpoints require `Authorization: Bearer <token>` unless stated otherwise.
**Trailing slashes:** Required on all paths (Django convention).

---

## Boards

A board is the top-level container for a CRM workflow. Each board belongs to one company and holds an ordered set of columns and tasks. Boards can be soft-archived rather than deleted.

---

### GET /crm/boards/

List boards for the authenticated user's company.

**Roles:** `company_admin`, `employee`, `superadmin`

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `include_archived` | `boolean` | When `true`, response includes both active and archived boards. Default: omit (active only). |
| `company_id` | `integer` | Superadmin only. Filter boards by a specific company ID. |

**Response** `200 OK`

Returns an array or paginated object. The frontend normalises both shapes:

```json
[
  {
    "id": 1,
    "name": "Product Development",
    "description": "Main dev board",
    "is_archived": false,
    "created_at": "2025-01-15T09:00:00Z",
    "updated_at": "2025-05-10T14:22:00Z",
    "company": 3
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Board ID |
| `name` | `string` | Max 100 characters |
| `description` | `string \| null` | Optional, max 500 characters |
| `is_archived` | `boolean` | `true` means the board is archived |
| `created_at` | `string` | ISO 8601 datetime |
| `updated_at` | `string` | ISO 8601 datetime |
| `company` | `integer` | Company ID (foreign key) |

---

### POST /crm/boards/

Create a new board.

**Roles:** `company_admin`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | Yes | Max 100 characters |
| `description` | `string` | No | Max 500 characters |

```json
{
  "name": "Q3 Sprint",
  "description": "Third quarter sprint board"
}
```

**Response** `201 Created` — returns the created `CrmBoard` object.

```json
{
  "id": 7,
  "name": "Q3 Sprint",
  "description": "Third quarter sprint board",
  "is_archived": false,
  "created_at": "2025-05-14T10:00:00Z",
  "updated_at": "2025-05-14T10:00:00Z",
  "company": 3
}
```

**Error Codes**

| Code | Condition |
|---|---|
| `400` | `name` missing or blank; board limit for the company's plan has been reached. Response body contains `detail` or `non_field_errors[0]` with a human-readable message. |
| `403` | Caller is not `company_admin` or `superadmin`. |

---

### GET /crm/boards/{id}/

Retrieve a single board by ID.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Board ID |

**Response** `200 OK` — single `CrmBoard` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Board does not exist or belongs to a different company. |

---

### PATCH /crm/boards/{id}/

Update board fields. Partial updates accepted — send only the fields you want to change.

**Roles:** `company_admin`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Board ID |

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | No | Max 100 characters |
| `description` | `string \| null` | No | Pass `null` to clear |

```json
{
  "name": "Q3 Sprint — Revised",
  "description": null
}
```

**Response** `200 OK` — updated `CrmBoard` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Validation failure (e.g. blank `name`). |
| `403` | Insufficient role. |
| `404` | Board not found. |

---

### DELETE /crm/boards/{id}/

Permanently delete a board and all its columns and tasks.

**Roles:** `company_admin`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Board ID |

**Response** `204 No Content`

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Insufficient role. |
| `404` | Board not found. |

---

### POST /crm/boards/{id}/archive/

Archive a board. Archived boards are hidden from the default board list but remain accessible with `?include_archived=true`.

**Roles:** `company_admin`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Board ID |

**Request Body:** None (empty POST).

**Response** `200 OK` — updated `CrmBoard` object with `"is_archived": true`.

```json
{
  "id": 7,
  "name": "Q3 Sprint",
  "is_archived": true,
  "updated_at": "2025-05-14T12:00:00Z",
  "company": 3,
  "description": null,
  "created_at": "2025-05-14T10:00:00Z"
}
```

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Insufficient role. |
| `404` | Board not found. |

---

### POST /crm/boards/{id}/unarchive/

Restore a board from archive.

**Roles:** `company_admin`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `id` | `integer` | Board ID |

**Request Body:** None.

**Response** `200 OK` — updated `CrmBoard` object with `"is_archived": false`.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Company's board limit would be exceeded by restoring this board. Response body: `{ "detail": "..." }` or `{ "non_field_errors": ["..."] }`. |
| `403` | Insufficient role. |
| `404` | Board not found. |

---

## Columns

Columns represent workflow stages within a board (e.g. "To Do", "In Progress", "Done"). Each column has an integer `order` (1-based, ascending left-to-right) and an optional WIP limit.

---

### GET /crm/boards/{boardId}/columns/

List all columns for a board, ordered by `order` ascending.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `boardId` | `integer` | Board ID |

**Response** `200 OK` — array or paginated object of `CrmColumn`.

```json
[
  {
    "id": 12,
    "name": "To Do",
    "order": 1,
    "board": 7,
    "wip_limit": null
  },
  {
    "id": 13,
    "name": "In Progress",
    "order": 2,
    "board": 7,
    "wip_limit": 3
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Column ID |
| `name` | `string` | Max 100 characters |
| `order` | `integer` | 1-based position within the board |
| `board` | `integer` | Parent board ID |
| `wip_limit` | `integer \| null` | Maximum active tasks allowed; `null` = unlimited |

---

### POST /crm/boards/{boardId}/columns/

Create a new column on a board. The column is appended at the end (highest `order`).

**Roles:** `company_admin`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | Yes | Max 100 characters |
| `wip_limit` | `integer \| null` | No | Min 0; omit or `null` for no limit |

```json
{
  "name": "Review",
  "wip_limit": 5
}
```

**Response** `201 Created` — the created `CrmColumn` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `name`; `wip_limit` negative. |
| `403` | Insufficient role. |
| `404` | Board not found. |

---

### GET /crm/boards/{boardId}/columns/{columnId}/

Retrieve a single column.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `200 OK` — single `CrmColumn` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Column not found or does not belong to the specified board. |

---

### PATCH /crm/boards/{boardId}/columns/{columnId}/

Update a column's name, WIP limit, or position.

**Roles:** `company_admin`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | No | Max 100 characters |
| `wip_limit` | `integer \| null` | No | Must be `>= current task count` in the column; `null` to remove limit |
| `position` | `integer` | No | 1-based desired position; backend reorders other columns accordingly |

```json
{
  "name": "In Review",
  "wip_limit": 4,
  "position": 3
}
```

**Response** `200 OK` — updated `CrmColumn` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | `wip_limit` is lower than the current task count in the column; blank `name`. |
| `403` | Insufficient role. |
| `404` | Column not found. |

---

### DELETE /crm/boards/{boardId}/columns/{columnId}/

Delete a column. The frontend does not delete columns that contain tasks — validate this before calling.

**Roles:** `company_admin`, `superadmin`

**Response** `204 No Content`

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Column still has tasks (backend enforced). |
| `403` | Insufficient role. |
| `404` | Column not found. |

---

### POST /crm/boards/{boardId}/columns/reorder/

Reorder all columns on a board in a single atomic operation. Called after a drag-and-drop interaction. The backend assigns `order` values based on the position of each ID in the provided array.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `column_ids` | `integer[]` | Yes | Ordered array of all column IDs for this board. Position in the array (0-indexed) determines the new `order` (1-indexed). Must include every column on the board. |

```json
{
  "column_ids": [13, 12, 15, 14]
}
```

**Response** `200 OK`

```json
{}
```

The frontend invalidates the `['crm', 'columns', boardId]` cache after success and restores the previous order on error.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Array is missing IDs or contains unknown IDs. |
| `403` | Insufficient role. |
| `404` | Board not found. |

---

## Labels

Labels are color-coded tags scoped to the company. A task can carry multiple labels. Labels are managed separately from boards and reused across all boards within the same company.

---

### GET /crm/labels/

List all labels for the authenticated user's company.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `200 OK` — array or paginated object of `CrmLabel`.

```json
[
  {
    "id": 1,
    "name": "Bug",
    "color": "#ef4444"
  },
  {
    "id": 2,
    "name": "Feature",
    "color": "#22c55e"
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Label ID |
| `name` | `string` | Max 50 characters |
| `color` | `string` | Hex color string, e.g. `"#ef4444"` |

---

### POST /crm/labels/

Create a new label.

**Roles:** `company_admin`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | Yes | Max 50 characters |
| `color` | `string` | Yes | Hex color string, e.g. `"#6366f1"` |

```json
{
  "name": "Urgent",
  "color": "#f97316"
}
```

**Response** `201 Created` — the created `CrmLabel` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `name`; malformed `color`. |
| `403` | Insufficient role. |

---

### GET /crm/labels/{id}/

Retrieve a single label.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `200 OK` — single `CrmLabel` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Label not found. |

---

### PATCH /crm/labels/{id}/

Update a label's name or color.

**Roles:** `company_admin`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | `string` | No | Max 50 characters |
| `color` | `string` | No | Hex color string |

```json
{
  "color": "#a855f7"
}
```

**Response** `200 OK` — updated `CrmLabel` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Validation failure. |
| `403` | Insufficient role. |
| `404` | Label not found. |

---

### DELETE /crm/labels/{id}/

Delete a label. The label is removed from all tasks it was attached to.

**Roles:** `company_admin`, `superadmin`

**Response** `204 No Content`

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Insufficient role. |
| `404` | Label not found. |
