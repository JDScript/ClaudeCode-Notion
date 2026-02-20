# References Database Schema

This document describes the References database schema used across research projects in Notion. Two variants are in use: the Standard Schema (from the Functional Reorientation project) and the Simplified Schema (from the Navigation Reasoning Benchmark project).

---

## Standard Schema

Source project: Functional Reorientation
Collection ID: `2ab6f2eb-947e-80f0-8f62-000b76f8f4eb`

This is the target schema for all new projects.

```sql
CREATE TABLE "collection://2ab6f2eb-947e-80f0-8f62-000b76f8f4eb" (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arxiv" TEXT,           -- type: url
    "Year" TEXT,            -- type: select, options: ["2023", "2024", "2025", "2026"]
    "Project Page" TEXT,    -- type: url
    "Github" TEXT,          -- type: url
    "Conference" TEXT,      -- type: multi_select, options: ["Science Robotics", "ICLR", "ICCV"]
    "Name" TEXT             -- type: title
)
```

### Property Details

| Property       | Notion Type    | Notes                                                              |
|----------------|----------------|--------------------------------------------------------------------|
| `url`          | (internal)     | Notion page URL, used as unique row identifier                     |
| `createdTime`  | (internal)     | ISO-8601 timestamp of page creation                                |
| `Arxiv`        | url            | Link to the paper on arxiv.org                                     |
| `Year`         | select         | Publication year; options: `2023`, `2024`, `2025`, `2026`          |
| `Project Page` | url            | Link to the official project website                               |
| `Github`       | url            | Link to the source code repository                                 |
| `Conference`   | multi_select   | Publication venue(s); options: `Science Robotics`, `ICLR`, `ICCV` |
| `Name`         | title          | Paper title; serves as the primary display name of the entry       |

---

## Simplified Schema

Source project: Navigation Reasoning Benchmark
Collection ID: `30d6f2eb-947e-80ed-958b-000ba8100415`

This schema is a reduced version used in an earlier project. It omits Year, Github, and Conference fields.

```sql
CREATE TABLE "collection://30d6f2eb-947e-80ed-958b-000ba8100415" (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arixv" TEXT,           -- type: url (note: typo in actual DB)
    "Project Page" TEXT,    -- type: url
    "Name" TEXT             -- type: title
)
```

### Property Details

| Property       | Notion Type | Notes                                                                   |
|----------------|-------------|-------------------------------------------------------------------------|
| `url`          | (internal)  | Notion page URL, used as unique row identifier                          |
| `createdTime`  | (internal)  | ISO-8601 timestamp of page creation                                     |
| `Arixv`        | url         | Link to the paper on arxiv.org; **field name is a typo** (should be `Arxiv`) |
| `Project Page` | url         | Link to the official project website                                    |
| `Name`         | title       | Paper title; serves as the primary display name of the entry            |

---

## Usage Notes

- **The Standard Schema is the recommended template for all new projects.** When creating a new References database, use the Functional Reorientation schema as the baseline.
- **Year options** (`2023`, `2024`, `2025`, `2026`) should be extended as needed when papers from additional years are added.
- **Conference options** (`Science Robotics`, `ICLR`, `ICCV`) should be extended as new venues are relevant to a project. Because `Conference` is a `multi_select`, a single entry can belong to multiple venues (e.g., an extended journal version of a conference paper).
- **Simplified Schema caveat:** The Navigation Reasoning Benchmark database uses `"Arixv"` (with a transposed `x` and `i`) instead of the correct `"Arxiv"`. This is a known typo in the live database and should not be replicated in new projects.
- When migrating entries from the Simplified Schema to the Standard Schema, the `Arixv` field maps directly to `Arxiv`; the missing fields (`Year`, `Github`, `Conference`) should be populated manually if the information is available.
