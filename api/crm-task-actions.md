# CRM API — Task Actions: Comments, History, Attachments, Checklists

**Base prefix:** `/api/v1/`
**Authentication:** All endpoints require `Authorization: Bearer <token>`.
**Trailing slashes:** Required on all paths.

---

## Comments

Comments are threaded discussion entries on a task. All authenticated users of the company can view and post comments. Edit and delete permissions are role-gated: the author can always edit and delete their own comment; `company_admin` can delete any comment; `superadmin` can edit and delete any comment.

---

### GET /crm/tasks/{taskId}/comments/

List all comments for a task, in chronological order.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |

**Response** `200 OK` — array or `{ results: CrmComment[] }`.

```json
[
  {
    "id": 101,
    "text": "Reviewed the spec. Looks good.",
    "author": {
      "id": 9,
      "full_name": "Asel Nurova",
      "avatar": "https://storage.example.com/avatars/9.jpg"
    },
    "created_at": "2025-05-14T11:22:00Z"
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Comment ID |
| `text` | `string` | Comment body |
| `author.id` | `integer` | User ID |
| `author.full_name` | `string` | |
| `author.avatar` | `string \| null` | URL or null |
| `created_at` | `string` | ISO 8601 datetime |

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

### POST /crm/tasks/{taskId}/comments/

Add a new comment to a task.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `text` | `string` | Yes | Non-blank |

```json
{
  "text": "Blocking on the backend endpoint — tracking in #42."
}
```

**Response** `201 Created` — the created `CrmComment` object.

**Side effects:** The task's `comments_count` is incremented. Relevant in-app notifications may be emitted.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `text`. |
| `404` | Task not found. |

---

### PATCH /crm/tasks/{taskId}/comments/{commentId}/

Edit the text of an existing comment.

**Roles:** Author of the comment; or `superadmin` for any comment.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |
| `commentId` | `integer` | Comment ID |

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `text` | `string` | Yes | Non-blank |

```json
{
  "text": "Updated: endpoint is now unblocked."
}
```

**Response** `200 OK` — updated `CrmComment` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `text`. |
| `403` | Caller is not the author and not `superadmin`. |
| `404` | Task or comment not found. |

---

### DELETE /crm/tasks/{taskId}/comments/{commentId}/

Delete a comment.

**Role-based access:**

| Role | Permission |
|---|---|
| `superadmin` | Can delete any comment |
| `company_admin` | Can delete any comment within their company |
| `employee` | Can only delete their own comment |

**Response** `204 No Content`

**Side effects:** The task's `comments_count` is decremented.

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Caller does not have permission to delete this comment. |
| `404` | Task or comment not found. |

---

## History

The history log records all meaningful state changes to a task (creation, field updates, archive actions, label changes, column moves). Entries are immutable — they are never edited or deleted.

---

### GET /crm/tasks/{taskId}/history/

List the change history for a task, with pagination. Entries are returned in reverse chronological order (newest first).

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `page` | `integer` | Page number (1-based). The frontend starts at page 1 and appends subsequent pages as the user clicks "Load more". |

**Response** `200 OK` — standard paginated envelope.

```json
{
  "count": 18,
  "next": "https://api.example.com/api/v1/crm/tasks/42/history/?page=2",
  "previous": null,
  "results": [
    {
      "id": 55,
      "user": {
        "id": 9,
        "full_name": "Asel Nurova",
        "avatar": null
      },
      "action": "updated",
      "field_name": "priority",
      "old_value": "medium",
      "new_value": "high",
      "created_at": "2025-05-14T13:00:00Z"
    },
    {
      "id": 50,
      "user": {
        "id": 9,
        "full_name": "Asel Nurova",
        "avatar": null
      },
      "action": "created",
      "field_name": null,
      "old_value": null,
      "new_value": null,
      "created_at": "2025-05-14T10:05:00Z"
    }
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | History entry ID |
| `user.id` | `integer` | User who performed the action |
| `user.full_name` | `string` | |
| `user.avatar` | `string \| null` | |
| `action` | `string` | See action values table below |
| `field_name` | `string \| null` | Set only when `action` is `"updated"` |
| `old_value` | `string \| null` | String representation of the previous value |
| `new_value` | `string \| null` | String representation of the new value |
| `created_at` | `string` | ISO 8601 datetime |

**Action Values**

| `action` | `field_name` | Description |
|---|---|---|
| `"created"` | `null` | Task was created |
| `"archived"` | `null` | Task was archived |
| `"unarchived"` | `null` | Task was restored from archive |
| `"moved"` | `null` | Task was moved (new_value contains destination description) |
| `"label_added"` | `null` | A label was added; `new_value` is JSON `{"name":"...","color":"..."}` |
| `"label_removed"` | `null` | A label was removed; `old_value` is JSON `{"name":"...","color":"..."}` |
| `"updated"` | `"title"` | Task title changed |
| `"updated"` | `"description"` | Description changed (no values shown in UI) |
| `"updated"` | `"priority"` | Priority changed; values are one of `low / medium / high / critical` |
| `"updated"` | `"deadline"` | Deadline changed; values are ISO date strings or `"null"` |
| `"updated"` | `"assignee"` | Assignee changed; values are full names or `"null"` |
| `"updated"` | `"column"` or `"column_id"` | Task moved between columns; values are column names |

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

## Attachments

File attachments linked to a task. Upload uses `multipart/form-data`. Files are stored server-side and served via a signed URL.

---

### GET /crm/tasks/{taskId}/attachments/

List all attachments for a task.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |

**Response** `200 OK` — array or `{ results: CrmAttachment[] }`.

```json
[
  {
    "id": 31,
    "filename": "spec_v2.pdf",
    "size": 204800,
    "mime_type": "application/pdf",
    "url": "https://storage.example.com/attachments/31/spec_v2.pdf?token=...",
    "uploaded_by": {
      "id": 9,
      "full_name": "Asel Nurova",
      "avatar": null
    },
    "created_at": "2025-05-14T11:00:00Z"
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Attachment ID |
| `filename` | `string` | Original file name |
| `size` | `integer` | File size in bytes |
| `mime_type` | `string` | MIME type (e.g. `"application/pdf"`, `"image/png"`) |
| `url` | `string` | Download URL (may be a signed/temporary URL) |
| `uploaded_by.id` | `integer` | |
| `uploaded_by.full_name` | `string` | |
| `uploaded_by.avatar` | `string \| null` | |
| `created_at` | `string` | ISO 8601 datetime |

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

### POST /crm/tasks/{taskId}/attachments/

Upload a file and attach it to a task.

**Roles:** `company_admin`, `employee`, `superadmin`

**Content-Type:** `multipart/form-data`

**Form Fields**

| Field | Type | Required | Notes |
|---|---|---|---|
| `file` | `File` | Yes | The file binary. Accepted types: `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.png`, `.jpg`, `.jpeg`, `.gif`. |

**Response** `201 Created` — the created `CrmAttachment` object.

**Side effects:** The task's `attachments_count` is incremented.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | File missing; file type not allowed; file too large. Response body: `{ "detail": "..." }` or field-level `{ "file": ["..."] }`. |
| `404` | Task not found. |

---

### DELETE /crm/tasks/{taskId}/attachments/{attachmentId}/

Delete an attachment.

**Role-based access:**

| Role | Permission |
|---|---|
| `company_admin` | Can delete any attachment within their company |
| `employee` | Can only delete attachments they uploaded |
| `superadmin` | Can delete any attachment |

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |
| `attachmentId` | `integer` | Attachment ID |

**Response** `204 No Content`

**Side effects:** The task's `attachments_count` is decremented. The file is deleted from storage.

**Error Codes**

| Code | Condition |
|---|---|
| `403` | Caller does not have permission to delete this attachment. |
| `404` | Task or attachment not found. |

---

## Checklists

A task can have multiple named checklists. Each checklist contains an ordered list of items that can be individually checked off. Progress is tracked as `completed / total`.

---

### GET /crm/tasks/{taskId}/checklists/

List all checklists for a task (including their items and progress).

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `taskId` | `integer` | Task ID |

**Response** `200 OK` — array of `CrmChecklist`.

```json
[
  {
    "id": 5,
    "title": "Acceptance criteria",
    "items": [
      { "id": 11, "text": "JWT issued on success", "is_completed": true, "order": 1 },
      { "id": 12, "text": "Refresh token rotated", "is_completed": false, "order": 2 }
    ],
    "checklist_progress": {
      "total": 2,
      "completed": 1
    }
  }
]
```

| Field | Type | Notes |
|---|---|---|
| `id` | `integer` | Checklist ID |
| `title` | `string` | Max 200 characters |
| `items` | `CrmChecklistItem[]` | Ordered by `item.order` ascending |
| `items[].id` | `integer` | Item ID |
| `items[].text` | `string` | Max 500 characters |
| `items[].is_completed` | `boolean` | |
| `items[].order` | `integer` | 1-based |
| `checklist_progress.total` | `integer` | Total item count |
| `checklist_progress.completed` | `integer` | Count of items with `is_completed: true` |

**Note:** Checklists are also embedded directly inside the `CrmTask` response under the `checklists` key, which avoids a separate round-trip when the full task is loaded.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Task not found. |

---

### POST /crm/tasks/{taskId}/checklists/

Create a new checklist on a task.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | `string` | Yes | Max 200 characters |

```json
{
  "title": "Definition of Done"
}
```

**Response** `201 Created` — the created `CrmChecklist` object (with `items: []` and `checklist_progress: { total: 0, completed: 0 }`).

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `title`. |
| `404` | Task not found. |

---

### GET /crm/checklists/{checklistId}/

Retrieve a single checklist by its own ID (not scoped by task).

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `checklistId` | `integer` | Checklist ID |

**Response** `200 OK` — single `CrmChecklist` object.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Checklist not found. |

---

### PATCH /crm/checklists/{checklistId}/

Update a checklist's title.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | `string` | Yes | Max 200 characters |

```json
{
  "title": "Updated checklist name"
}
```

**Response** `200 OK` — updated `CrmChecklist` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `title`. |
| `404` | Checklist not found. |

---

### DELETE /crm/checklists/{checklistId}/

Delete a checklist and all its items.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `204 No Content`

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Checklist not found. |

---

### GET /crm/checklists/{checklistId}/items/

List all items in a checklist, ordered by `order` ascending.

**Roles:** `company_admin`, `employee`, `superadmin`

**Response** `200 OK` — array of `CrmChecklistItem`.

```json
[
  { "id": 11, "text": "JWT issued on success", "is_completed": true, "order": 1 },
  { "id": 12, "text": "Refresh token rotated", "is_completed": false, "order": 2 }
]
```

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Checklist not found. |

---

### POST /crm/checklists/{checklistId}/items/

Add an item to a checklist. The item is appended at the end.

**Roles:** `company_admin`, `employee`, `superadmin`

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `text` | `string` | Yes | Max 500 characters |

```json
{
  "text": "Unit tests passing"
}
```

**Response** `201 Created` — the created `CrmChecklistItem` object.

```json
{
  "id": 13,
  "text": "Unit tests passing",
  "is_completed": false,
  "order": 3
}
```

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `text`. |
| `404` | Checklist not found. |

---

### PATCH /crm/items/{itemId}/

Update a checklist item's text and/or completion status. Used for both inline text editing and checkbox toggling.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `itemId` | `integer` | Checklist item ID |

**Request Body** `application/json`

| Field | Type | Required | Notes |
|---|---|---|---|
| `text` | `string` | No | Max 500 characters |
| `is_completed` | `boolean` | No | Toggle completion state |

Toggle completion only:
```json
{
  "is_completed": true
}
```

Edit text only:
```json
{
  "text": "Unit tests passing at 90%+ coverage"
}
```

**Response** `200 OK` — updated `CrmChecklistItem` object.

**Error Codes**

| Code | Condition |
|---|---|
| `400` | Blank `text`. |
| `404` | Item not found. |

---

### DELETE /crm/items/{itemId}/

Delete a single checklist item.

**Roles:** `company_admin`, `employee`, `superadmin`

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `itemId` | `integer` | Checklist item ID |

**Response** `204 No Content`

**Side effects:** The parent checklist's `checklist_progress.total` is decremented; if the item was completed, `completed` is also decremented.

**Error Codes**

| Code | Condition |
|---|---|
| `404` | Item not found. |
