# Skills Catalog

This catalog describes all four Claude Code skills in the Notion Research Management system. Each skill lives in `.claude/skills/<skill-name>/SKILL.md` and is invoked via a slash command or natural language.

---

## 1. scaffold-project

**Description:** Creates a new research project page in the Notion Research Space with the standard structure: a two-column header (Project Overview callout + Quick Links callout), a Meeting Notes sub-page, and inline Tasks and References databases.

### Example Invocations

```
/scaffold-project
```
```
Create a new project called "Temporal Grounding" with icon 🎯
```
```
scaffold-project project_name="Visual SLAM" icon=🗺️ one_liner="Real-time visual localization in unstructured environments"
```

### Required Inputs

| Field | Required | Default |
|---|---|---|
| `project_name` | Yes | — |
| `icon_emoji` | Yes | — |
| `one_liner` | No | "Placeholder" |
| `application` | No | "Placeholder" |
| `proposed_solutions` | No | "Placeholder" |

### Outputs

- A new project page under the Research Space (page ID: `2306f2eb-947e-805e-9064-ed452a446342`)
- A Meeting Notes sub-page moved under the project page
- Inline Tasks database (properties: Task, Status, Due, Week, Assigned To)
- Inline References database (properties: Name, Arxiv, Year, Conference, Github, Project Page)
- A summary report with clickable URLs for all created pages

### Templates and Samples Used

- `docs/templates/project-page.md` — defines the two-column callout layout written to the project page body
- `docs/samples/tasks-db-schema.md` — referenced as fallback instructions when `notion-create-database` is unavailable
- `docs/samples/references-db-schema.md` — referenced as fallback instructions when `notion-create-database` is unavailable

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `notion-create-pages` | Create the Meeting Notes sub-page; create the project page |
| `notion-move-pages` | Move Meeting Notes under the project page after both are created |
| `notion-create-database` | Create inline Tasks and References databases (if available) |
| `notion-fetch` | Fetch the enhanced Markdown spec resource (`notion://docs/enhanced-markdown-spec`) |

---

## 2. read-paper

**Description:** Reads an arxiv paper (via URL or an existing Notion References DB entry), generates a structured AI summary using the standard template, fills in metadata properties (Year, Conference, Github, Project Page), and writes the result to the Notion References DB entry. On regeneration, replaces only the AI Summary zone and never touches the user's My Notes section.

### Example Invocations

```
/read-paper https://arxiv.org/abs/2312.04567
```
```
Read this paper and add it to the Functional Reorientation project: https://arxiv.org/abs/2406.12345
```
```
Regenerate the summary for https://www.notion.so/Some-Paper-Title-abc123
```

### Required Inputs

| Field | Required | Notes |
|---|---|---|
| arxiv URL or Notion page URL | Yes | Arxiv URL: canonical `https://arxiv.org/abs/XXXX.XXXXX` form; Notion URL: existing References DB entry |
| project name | Conditional | Required only when no existing DB entry is found and the skill must ask where to create one |

### Outputs

- Notion paper page with AI Summary section filled (Problem & Motivation, Method / Approach, Key Results, Relevance & Connections)
- DB properties updated: Year, Conference, Github, Project Page
- Summary report listing what was written, what properties were set, and related papers found

### Templates and Samples Used

- `docs/templates/paper-summary.md` — defines the `## AI Summary` / `## My Notes` page structure and all `{placeholder}` variables

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `WebFetch` | Fetch arxiv abstract page and optionally the HTML full text |
| `notion-fetch` | Load an existing Notion page to extract the arxiv URL or check for a `## My Notes` section |
| `notion-search` | Search the References DB for an existing entry or find related papers |
| `notion-create-pages` | Create a new References DB entry when none exists |
| `notion-update-page` | Write page content (`replace_content` or `replace_content_range`) and update DB properties |

---

## 3. manage-tasks

**Description:** Creates, updates, and lists tasks in a research project's Notion Tasks database. Accepts natural language commands. Supports filtering by status or week when listing, and uses an expanded date format (`date:Due:start` / `date:Due:end` / `date:Due:is_datetime`) required by the Tasks DB schema.

### Example Invocations

```
Add a task "Draft literature review" to Functional Reorientation, due 2026-03-10, Week 4
```
```
Mark the "Implement baseline model" task as Done in the Visual SLAM project
```
```
Show all tasks in Functional Reorientation that are In progress
```

### Required Inputs

| Field | Required | Notes |
|---|---|---|
| `project_name` | Yes | Must match an existing Notion project page |
| `task_name` | For create/update | Title of the task |
| `operation` | Inferred | `create`, `update`, or `list` — inferred from phrasing |
| `status` | No | `Not started` / `In progress` / `Done`; defaults to `Not started` on create |
| `due_date` | No | ISO-8601 date string |
| `week` | No | One or more of `Week 2` through `Week 9` |

### Outputs

- **Create:** New task row in the Tasks DB, with a summary and clickable Notion URL
- **Update:** Updated task row, reporting which properties changed
- **List:** Markdown table of tasks (Task, Status, Due, Week), optionally filtered

### Templates and Samples Used

- `docs/samples/tasks-db-schema.md` — defines the Tasks DB property names, types, status group values, and expanded date format used in all API calls

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `notion-search` | Find the project page by name |
| `notion-fetch` | Fetch the project page to extract the Tasks DB URL; fetch the Tasks DB to retrieve rows for update/list |
| `notion-create-pages` | Insert a new task row (using `parent_type: "data_source_id"`) |
| `notion-update-page` | Update task properties with `command: "update_properties"` |

---

## 4. meeting-notes

**Description:** Creates a structured meeting notes page under a research project's Meeting Notes sub-page in Notion. Uses the standard meeting notes template with sections for Attendees, Agenda, Discussion, Action Items, and Decisions. Creates the Meeting Notes sub-page automatically if it does not already exist.

### Example Invocations

```
/meeting-notes project="Functional Reorientation"
```
```
Create meeting notes for the Functional Reorientation project, topic "Week 4 Check-in", attendees: Alice, Bob
```
```
Add meeting notes to Visual SLAM: topic="Baseline Review", agenda: review results, plan next steps
```

### Required Inputs

| Field | Required | Default |
|---|---|---|
| `project_name` | Yes | — |
| `topic` | No | "Meeting" |
| `attendees` | No | Placeholder text |
| `agenda_items` | No | Placeholder text |

### Outputs

- A new meeting notes page titled `"<topic> - <Mon DD, YYYY>"` (e.g., `"Week 4 Check-in - Feb 20, 2026"`) created under the project's Meeting Notes sub-page
- Discussion, Action Items, and Decisions sections left as placeholders for post-meeting fill-in
- A summary report with clickable URL to the newly created page

### Templates and Samples Used

- `docs/templates/meeting-notes.md` — defines the five-section page structure (Attendees, Agenda, Discussion, Action Items, Decisions) and placeholder variables

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `notion-search` | Find the project page by name |
| `notion-fetch` | Fetch the project page to locate the Meeting Notes sub-page |
| `notion-create-pages` | Create the Meeting Notes sub-page if missing; create the new meeting notes page |
