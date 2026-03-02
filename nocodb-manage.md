# NocoDB Table & Column Management

Manage NocoDB tables, columns, views, form fields, and records via the API.

## Setup

1. Read these variables from `local.env` in the current project root:
   - `NOCO_HOST` — NocoDB instance URL (e.g. `https://my-nocodb.example.com`)
   - `NOCO_API_TOKEN` — API token for authentication
   - `NOCO_BASE_ID` — Base/project ID (prefix `p...`)
2. All API calls use header: `xc-token: <NOCO_API_TOKEN>` (NOT `xc-auth`)
3. All endpoints are relative to `NOCO_HOST`

## Workflow

1. **Before changes**: Inspect current state and present a summary to the user
2. **Make changes**: Use the verified endpoints below
3. **After changes**: Re-fetch and verify
4. **Destructive operations** (delete): Always confirm with the user first

---

## ID Prefixes

| ID             | Prefix   | Description                    |
| -------------- | -------- | ------------------------------ |
| Base ID        | `p...`   | Database/project               |
| Table ID       | `m...`   | Table                          |
| View ID        | `vw...`  | Grid/Form/Gallery/Kanban view  |
| Column ID      | `c...`   | Table column (schema level)    |
| View Column ID | `nc...`  | Column in a grid view          |
| Form Column ID | `fvc...` | Column in a form view          |

View types: 1=Form, 3=Grid, 4=Gallery, 5=Kanban

---

## Meta API — Structure Management

### Read

| Purpose                    | Method | Endpoint                              |
| -------------------------- | ------ | ------------------------------------- |
| List tables in base        | GET    | `/api/v2/meta/bases/{baseId}/tables`  |
| Get table details + fields | GET    | `/api/v2/meta/tables/{tableId}`       |
| List views for a table     | GET    | `/api/v2/meta/tables/{tableId}/views` |
| Get columns in a view      | GET    | `/api/v2/meta/views/{viewId}/columns` |
| Get form view config       | GET    | `/api/v2/meta/forms/{viewId}`         |

### Create

| Purpose             | Method | Endpoint                                |
| ------------------- | ------ | --------------------------------------- |
| Create table        | POST   | `/api/v2/meta/bases/{baseId}/tables`    |
| Create column       | POST   | `/api/v2/meta/tables/{tableId}/columns` |

Create column body: `{"column_name":"Name","title":"Name","uidt":"SingleLineText"}`
Always provide both `column_name` and `title`.

### Update

| Purpose                     | Method | Endpoint                                             |
| --------------------------- | ------ | ---------------------------------------------------- |
| Rename table                | PATCH  | `/api/v2/meta/tables/{tableId}`                      |
| Rename view                 | PATCH  | `/api/v2/meta/views/{viewId}`                        |
| Update column (rename/opts) | PATCH  | `/api/v2/meta/columns/{columnId}`                    |
| Reorder/show/hide in view   | PATCH  | `/api/v2/meta/views/{viewId}/columns/{viewColumnId}` |
| Update form column props    | PATCH  | `/api/v2/meta/form-columns/{formColumnId}`           |
| Update form view settings   | PATCH  | `/api/v2/meta/forms/{viewId}`                        |

### Delete (confirm with user first!)

| Purpose         | Method | Endpoint                          |
| --------------- | ------ | --------------------------------- |
| Delete column   | DELETE | `/api/v2/meta/columns/{columnId}` |
| Delete view     | DELETE | `/api/v2/meta/views/{viewId}`     |

Delete column uses `/columns/{columnId}` directly — NOT nested under `/tables/`.
Delete view returns `true` on success.

---

## Data API — Record Management

| Purpose      | Method | Endpoint                              | Body                          |
| ------------ | ------ | ------------------------------------- | ----------------------------- |
| List rows    | GET    | `/api/v2/tables/{tableId}/records`    | —                             |
| Create row   | POST   | `/api/v2/tables/{tableId}/records`    | `{"Title":"value",...}`       |
| Update row   | PATCH  | `/api/v2/tables/{tableId}/records`    | `{"Id":1,"Title":"new"}`     |
| Delete rows  | DELETE | `/api/v2/tables/{tableId}/records`    | `[{"Id":1},{"Id":2}]`        |

Note: Data API uses `/api/v2/tables/` (no `meta/` segment). Field names in bodies must match column titles exactly.

---

## Key Behaviors & Gotchas

### Renaming Tables
- Requires BOTH `title` and `table_name` in the body: `{"title":"New Name","table_name":"New Name"}`
- Omitting `table_name` returns error: "Missing table name"

### Column Reordering
- `{"order": N}` — 1-based position. API returns `1` on success.
- Primary display column (Title) is always pinned at order 1. Setting `"order": 1` on another column places it at position 2.

### Form Views
- Get form config + columns: `GET /api/v2/meta/forms/{viewId}`
- Form column IDs have `fvc...` prefix
- **CRITICAL**: To set `required`, `label`, `help`, `description` you MUST use `PATCH /api/v2/meta/form-columns/{formColumnId}`. The generic `/views/{id}/columns/{id}` endpoint silently accepts these but does NOT persist them. The generic endpoint only works for `order` and `show` on form views.

```json
{ "required": true, "label": "Custom Label", "help": "Placeholder text", "description": "Help text below field" }
```

### Form View Settings
- Update heading, subheading, success message, redirect URL via `PATCH /api/v2/meta/forms/{viewId}`:

```json
{ "heading": "Form Title", "subheading": "Description", "success_msg": "Thank you!", "redirect_url": "https://example.com", "submit_another_form": true }
```

### SingleSelect / MultiSelect Options
- Options CANNOT be set during column creation (dtxp is ignored)
- Create column first, then set options via separate PATCH:

```
PATCH /api/v2/meta/columns/{columnId}
{"colOptions":{"options":[{"title":"Yes","color":"#4CAF50"},{"title":"No","color":"#F44336"}]}}
```

Colors: `#4CAF50` (green), `#F44336` (red), `#2196F3` (blue), `#FF9800` (orange), `#9C27B0` (purple), `#607D8B` (grey)

### View column IDs vs table column IDs
- View columns (`nc...` / `fvc...`) are different from table columns (`c...`)
- Reordering uses VIEW column IDs
- Cross-reference via `fk_column_id` field on view columns

---

## Endpoints That Do NOT Work

| Documented Endpoint                                 | Status |
| --------------------------------------------------- | ------ |
| `POST /api/v2/meta/tables/{tableId}/fields`         | 404    |
| `POST /api/v2/meta/tables/{tableId}/views`           | 404    |
| `DELETE /api/v2/meta/tables/{tableId}/fields/{id}`  | 404    |
| `DELETE /api/v2/meta/tables/{tableId}/columns/{id}` | 404    |

---

## Common Field Types (`uidt`)

`SingleLineText` `LongText` `Number` `Decimal` `Checkbox` `SingleSelect` `MultiSelect` `Date` `DateTime` `Email` `URL` `Attachment` `Currency` `Percent` `Rating` `Formula` `Lookup` `Rollup` `LinkToAnotherRecord`

---

## Environment Notes

- Use `node -e` for JSON formatting (no python on Windows)
- Bash escapes `!` — avoid `!==` in inline node scripts. Use `===false` or write to temp file: `curl ... > "$TEMP/noco.json" && node -e "const t=JSON.parse(require('fs').readFileSync(process.env.TEMP+'/noco.json','utf8')); ..."`
- Rate limit: 5 req/s. HTTP 429 → wait 30 seconds.
- New columns auto-appear in all views at the end (highest order).

## User input
$ARGUMENTS
