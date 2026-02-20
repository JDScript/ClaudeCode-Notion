---
name: manage-tasks
description: Create, update, and query tasks in a research project's Notion Tasks database. Supports natural language commands.
---

# Skill: manage-tasks

This skill creates, updates, and lists tasks in a research project's Notion Tasks database. The Tasks DB standard schema is documented in `docs/samples/tasks-db-schema.md`. All operations follow the same lookup pattern: find the project page, locate its Tasks database, then execute the requested operation.

---

## Step 0: Parse User Intent

Determine the operation and extract all provided details from the user's message.

### Operation types

| Operation | Trigger phrases                                               |
|-----------|---------------------------------------------------------------|
| `create`  | "add a task", "create a task", "new task"                     |
| `update`  | "mark as done", "update task", "set status", "change due date"|
| `list`    | "show tasks", "list tasks", "what tasks", "query tasks"       |

### Fields to extract

| Field          | Applies to          | Notes                                                                                      |
|----------------|---------------------|--------------------------------------------------------------------------------------------|
| `project_name` | all operations      | The research project whose Tasks DB to operate on                                         |
| `task_name`    | create, update      | The title of the task                                                                      |
| `status`       | create, update      | One of: `Not started`, `In progress`, `Done`; default for create is `Not started`         |
| `due_date`     | create, update      | A date string parseable to ISO-8601 (e.g. `2026-03-15`); optional                        |
| `due_end`      | create, update      | End date for a date range; omit if not a range                                             |
| `week`         | create, update, list| One or more of: `Week 2` … `Week 9`; multi-select                                        |
| `assigned_to`  | create, update      | Notion workspace user ID(s); omit if not specified                                        |
| `filter_status`| list                | Filter listed tasks by status                                                              |
| `filter_week`  | list                | Filter listed tasks by week                                                                |

If `project_name` is not provided, ask the user before proceeding.

---

## Step 1: Find the Project Page

Use `notion-search` to locate the project page by name.

```json
{
  "query": "<project_name>"
}
```

From the results, identify the page whose title best matches the user's project name. Record its URL (e.g. `https://www.notion.so/Project-Name-<id>`). If multiple results are returned, prefer the one whose title is an exact or close match and whose parent is the Research Space.

If no matching page is found, inform the user and stop.

---

## Step 2: Find the Tasks Database

Use `notion-fetch` on the project page URL obtained in Step 1.

```json
{
  "url": "<project_page_url>"
}
```

Inspect the fetched page content for a linked or inline database named **Tasks**. It will appear as a `data-source-url` attribute on a database block, with a URL in the form:

```
https://www.notion.so/<workspace>/<collection_id>?v=<view_id>
```

Record the `data-source-url` value — this is the Tasks DB URL. The `collection_id` embedded in the URL path (the UUID segment before `?v=`) is the **data_source_id** needed for creating new task entries.

If no Tasks database is found on the project page, inform the user that the project may not have a Tasks DB and suggest running the `scaffold-project` skill or creating the database manually using the schema in `docs/samples/tasks-db-schema.md`.

---

## Step 3: Execute the Operation

Proceed to the section below that matches the operation determined in Step 0.

---

### Operation A: Create a Task

Use `notion-create-pages` with `parent_type: "data_source_id"` and the `data_source_id` extracted in Step 2. This inserts a new row into the Tasks database.

```json
{
  "parent_type": "data_source_id",
  "parent_id": "<collection_id_from_tasks_db_url>",
  "title": "<task_name>",
  "properties": {
    "Status": "Not started",
    "Week": ["<week_value>"],
    "date:Due:start": "<YYYY-MM-DD>",
    "date:Due:end": "<YYYY-MM-DD or omit>",
    "date:Due:is_datetime": 0
  }
}
```

Rules:

- **`parent_type`** must be `"data_source_id"`, not `"database_id"` or `"page_id"`. This is required for inserting rows into an inline Notion database.
- **`Status`**: Default to `"Not started"` unless the user specified a different value. Valid values: `"Not started"`, `"In progress"`, `"Done"`.
- **`Week`**: Pass as a JSON array of strings. Omit the property if the user did not specify a week.
- **Date fields** — use the expanded format:
  - `"date:Due:start"`: ISO-8601 date string, e.g. `"2026-03-15"`. Omit if no due date was given.
  - `"date:Due:end"`: ISO-8601 date string for the end of a range. Omit entirely when not a date range.
  - `"date:Due:is_datetime"`: `0` for date-only (most common), `1` if the user specified a time component. Omit entirely when no due date is set.
- **`Assigned To`**: Omit unless the user provides Notion user IDs.
- Omit any property key that has no value to set. Do not send `null` or empty strings.

**Minimal create example (no due date, no week):**

```json
{
  "parent_type": "data_source_id",
  "parent_id": "2ab6f2eb-947e-800c-af0c-000b3a250c50",
  "title": "Draft literature review",
  "properties": {
    "Status": "Not started"
  }
}
```

**Full create example (with due date range and week):**

```json
{
  "parent_type": "data_source_id",
  "parent_id": "2ab6f2eb-947e-800c-af0c-000b3a250c50",
  "title": "Implement baseline model",
  "properties": {
    "Status": "In progress",
    "Week": ["Week 4", "Week 5"],
    "date:Due:start": "2026-03-01",
    "date:Due:end": "2026-03-07",
    "date:Due:is_datetime": 0
  }
}
```

---

### Operation B: Update a Task

#### B1. Find the task entry

Use `notion-fetch` on the Tasks DB URL from Step 2 to retrieve all rows.

```json
{
  "url": "<tasks_db_url>"
}
```

Scan the returned rows for an entry whose `Task` title matches the user's description. If there are multiple close matches, show the user a short list and ask which one to update. Record the target task's Notion page URL and page ID.

If no matching task is found, inform the user.

#### B2. Apply the update

Use `notion-update-page` with `command: "update_properties"` on the target task page.

```json
{
  "page_id": "<task_page_id>",
  "command": "update_properties",
  "properties": {
    "Status": "<new_status>",
    "Week": ["<week_value>"],
    "date:Due:start": "<YYYY-MM-DD>",
    "date:Due:end": "<YYYY-MM-DD or omit>",
    "date:Due:is_datetime": 0
  }
}
```

Rules:

- Include **only** the properties the user wants to change. Do not overwrite fields the user did not mention.
- **`Status`**: Set to `"Not started"`, `"In progress"`, or `"Done"` as requested.
- **`Week`**: Pass as a JSON array. Replaces the existing week values entirely.
- **Date fields** — always use the expanded three-key format when updating Due:
  - `"date:Due:start"`: required whenever changing the due date.
  - `"date:Due:end"`: include only when setting a date range; omit to clear the end date or when not changing it to a range.
  - `"date:Due:is_datetime"`: `0` for date-only, `1` if time is included. Must be sent together with `date:Due:start`.
- To **clear** the due date, send `"date:Due:start": null` and `"date:Due:is_datetime": 0`.

**Common update examples:**

Mark a task done:

```json
{
  "page_id": "<task_page_id>",
  "command": "update_properties",
  "properties": {
    "Status": "Done"
  }
}
```

Set a due date:

```json
{
  "page_id": "<task_page_id>",
  "command": "update_properties",
  "properties": {
    "date:Due:start": "2026-03-20",
    "date:Due:is_datetime": 0
  }
}
```

Change status and reassign week:

```json
{
  "page_id": "<task_page_id>",
  "command": "update_properties",
  "properties": {
    "Status": "In progress",
    "Week": ["Week 6"]
  }
}
```

---

### Operation C: List / Query Tasks

Use `notion-fetch` on the Tasks DB URL from Step 2 to retrieve all task rows.

```json
{
  "url": "<tasks_db_url>"
}
```

From the returned rows, collect each task's properties: `Task` (title), `Status`, `date:Due:start`, `date:Due:end`, `Week`.

Apply any filters the user specified:

- **Filter by status**: Retain only rows where `Status` matches `filter_status`.
- **Filter by week**: Retain only rows where the `Week` multi-select contains `filter_week`.
- Both filters may be applied simultaneously.

Present results as a Markdown table:

```
| Task                        | Status      | Due        | Week          |
|-----------------------------|-------------|------------|---------------|
| Draft literature review     | Not started | —          | Week 3        |
| Implement baseline model    | In progress | 2026-03-01 | Week 4, Week 5|
| Write evaluation section    | Done        | 2026-03-20 | Week 6        |
```

If no tasks match the filter, report: "No tasks found matching the given criteria."

If the Tasks DB is empty, report: "The Tasks database has no entries yet."

---

## Step 4: Report Results

### After Create

```
Task created in <project_name> Tasks DB:

  Title:   <task_name>
  Status:  <status>
  Due:     <due_date range, or "—" if not set>
  Week:    <week values, or "—" if not set>

Notion page: <task_page_url>
```

### After Update

```
Task updated:

  Title:   <task_name>
  Changed: <list the properties that were updated and their new values>

Notion page: <task_page_url>
```

### After List

Display the table produced in Step 3C, preceded by:

```
Tasks in <project_name> (<N> total, filtered by: <filter description or "none">):
```

---

## Expanded Date Property Format

The Tasks DB uses Notion's expanded date column format. Whenever reading or writing a `Due` date, use these three keys — never a single `"Due"` key:

| Key                    | Type    | Description                                     |
|------------------------|---------|-------------------------------------------------|
| `date:Due:start`       | string  | ISO-8601 date, e.g. `"2026-03-15"` (required)  |
| `date:Due:end`         | string  | ISO-8601 date for range end (optional)          |
| `date:Due:is_datetime` | integer | `0` = date only, `1` = date + time              |

**Single date:**

```json
"date:Due:start": "2026-03-15",
"date:Due:is_datetime": 0
```

**Date range:**

```json
"date:Due:start": "2026-03-01",
"date:Due:end": "2026-03-07",
"date:Due:is_datetime": 0
```

**Clear the date:**

```json
"date:Due:start": null,
"date:Due:is_datetime": 0
```

Never send a bare `"Due"` property key — it will be ignored or cause an error.

---

## Reference: MCP Tool Names

| Tool                  | Usage in this skill                                                  |
|-----------------------|----------------------------------------------------------------------|
| `notion-search`       | Step 1: Find the project page by name                                |
| `notion-fetch`        | Step 2: Fetch project page to locate Tasks DB; Step 3B/C: fetch rows|
| `notion-create-pages` | Step 3A: Insert a new task row into the Tasks database               |
| `notion-update-page`  | Step 3B: Update task properties (`update_properties` command)        |

---

## Reference: Tasks DB Schema

Full schema documentation (property types, status groups, week options, usage notes):

```
docs/samples/tasks-db-schema.md
```

Standard properties:

| Property               | Notion Type  | Values / Notes                                    |
|------------------------|--------------|---------------------------------------------------|
| `Task`                 | title        | Display name of the task                          |
| `Status`               | status       | `Not started` / `In progress` / `Done`            |
| `date:Due:start`       | date         | ISO-8601 date string                              |
| `date:Due:end`         | date         | ISO-8601 date string; omit when not a range       |
| `date:Due:is_datetime` | integer      | `0` = date only, `1` = datetime                   |
| `Week`                 | multi_select | `Week 2` … `Week 9`; array of strings             |
| `Assigned To`          | person       | JSON array of Notion user ID strings              |
