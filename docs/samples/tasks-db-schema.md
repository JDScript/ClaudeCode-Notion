# Tasks Database Schema

This document describes the Tasks database schema used to track work items within research projects in Notion. The schema shown here is sourced from the Functional Reorientation project and serves as the standard template for new projects.

---

## Schema

Source project: Functional Reorientation
Collection ID: `2ab6f2eb-947e-800c-af0c-000b3a250c50`

```sql
CREATE TABLE "collection://2ab6f2eb-947e-800c-af0c-000b3a250c50" (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Assigned To" TEXT,         -- type: person (JSON array of user IDs)
    "date:Due:start" TEXT,      -- ISO-8601 date
    "date:Due:end" TEXT,        -- ISO-8601 date (optional, for ranges)
    "date:Due:is_datetime" INT, -- 0=date, 1=datetime
    "Status" TEXT,              -- type: status, options: ["Not started", "In progress", "Done"]
    "Week" TEXT,                -- type: multi_select, options: ["Week 2".."Week 9"]
    "Task" TEXT                 -- type: title
)
```

---

## Property Details

| Property               | Notion Type  | Notes                                                                                      |
|------------------------|--------------|--------------------------------------------------------------------------------------------|
| `url`                  | (internal)   | Notion page URL, used as unique row identifier                                             |
| `createdTime`          | (internal)   | ISO-8601 timestamp of page creation                                                        |
| `Assigned To`          | person       | Stored as a JSON array of Notion user ID strings; may be empty or contain multiple users   |
| `date:Due:start`       | date         | Start date (or sole date) of the due date field; ISO-8601 format (e.g., `2026-03-01`)     |
| `date:Due:end`         | date         | End date for a due-date range; omit or set to `null` when not a range                     |
| `date:Due:is_datetime` | integer      | `0` = date only, `1` = includes a specific time component                                 |
| `Status`               | status       | Workflow status of the task; see Status Groups below                                       |
| `Week`                 | multi_select | Sprint/calendar week(s) the task belongs to; options: `Week 2` through `Week 9`           |
| `Task`                 | title        | Short description of the task; serves as the primary display name of the entry             |

---

## Status Groups

Notion's `status` type organizes options into named groups that control board-view column behavior.

| Group         | Options in this group |
|---------------|-----------------------|
| `to_do`       | `Not started`         |
| `in_progress` | `In progress`         |
| `complete`    | `Done`                |

The default status for newly created tasks is `Not started`.

---

## Views

The Tasks database is configured with two standard views:

| View  | Type  | Grouping  | Description                                                                 |
|-------|-------|-----------|-----------------------------------------------------------------------------|
| Board | Board | `Status`  | Kanban-style columns for `Not started`, `In progress`, and `Done`           |
| Table | Table | `Week`    | Flat table with rows grouped by the `Week` multi-select value               |

---

## Usage Notes

- **Default status:** When inserting a new task programmatically or via the UI, set `Status` to `"Not started"` unless the task is already underway.
- **Week options** (`Week 2` through `Week 9`) correspond to the weeks of the semester sprint schedule. Extend this list as needed when a project spans more weeks; because `Week` is a `multi_select`, a task can be tagged with multiple weeks if it spans a boundary.
- **Due date ranges:** To represent a multi-day deadline span, populate both `date:Due:start` and `date:Due:end`. For a single deadline, populate only `date:Due:start` and leave `date:Due:end` empty.
- **Assigned To encoding:** The `person` type is stored as a JSON array of Notion workspace user ID strings, for example `["abc123", "def456"]`. An unassigned task should store an empty array `[]` rather than `null`.
- **Status groups are fixed** at the Notion schema level. Adding new status options requires assigning each option to one of the three group keys (`to_do`, `in_progress`, `complete`); the group key itself cannot be renamed without recreating the property.
- When creating a new Tasks database for a different project, use this schema as the baseline and adjust the `Week` options to match the new project's timeline.
