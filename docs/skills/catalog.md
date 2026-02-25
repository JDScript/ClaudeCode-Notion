# Skills Catalog

This catalog describes all seven Claude Code skills in the Notion Research Management system. Each skill lives in `.claude/skills/<skill-name>/SKILL.md` and is invoked via a slash command or natural language.

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

- A new project page under the workspace root (page ID from `.claude/notion-config.json`)
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

---

## 5. read-dataset

**Description:** Reads a dataset paper from arxiv, generates a structured AI summary, fills in sensor and scale metadata properties, and writes the result to a project's Datasets DB entry. On regeneration, replaces only the AI Summary zone.

### Example Invocations

```
/read-dataset https://arxiv.org/abs/2309.13549
```
```
Read this dataset and add it to the Navigation Reasoning Benchmark project: https://arxiv.org/abs/2406.12345
```

### Required Inputs

| Field | Required | Notes |
|---|---|---|
| arxiv URL or Notion page URL | Yes | Arxiv URL or existing Datasets DB entry |
| project name | Conditional | Required when no existing DB entry is found |

### Outputs

- Notion dataset page with AI Summary (Overview, Data Collection, Sensor Config, Scale, Relevance)
- DB properties updated: Year, License, Scene Type, Embodiment, Camera, LiDAR, IMU, GPS, Episodes, Video Length, Trajectory Length, Scene Size

### Templates and Samples Used

- Inline template in skill definition (default) or project-specific "Dataset Summary Template" from Notion
- `docs/samples/datasets-db-schema.md`

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `WebFetch` | Fetch arxiv abstract and HTML full text |
| `notion-fetch` | Load existing page, check for My Notes section |
| `notion-search` | Search Datasets DB for existing or related entries |
| `notion-create-pages` | Create new Datasets DB entry |
| `notion-update-page` | Write content and update properties |

---

## 6. extract-technical-details

**Description:** Reads the full arxiv PDF of a paper, extracts step-by-step implementation details organized by pipeline stages, creates a "Technical Review" sub-page under the paper's Notion References DB entry, and links it from the TL;DR callout.

### Example Invocations

```
/extract-technical-details https://arxiv.org/abs/2510.08556
```

### Required Inputs

| Field | Required | Notes |
|---|---|---|
| arxiv URL or Notion page URL | Yes | Parent References DB entry must already exist |

### Outputs

- Technical Review sub-page with per-stage details (architecture, hyperparameters, training setup)
- Link added to parent page's TL;DR callout

### Templates and Samples Used

- `docs/templates/technical-review-template.md` or project-specific "Technical Review Template" from Notion

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `WebFetch` | Optionally fetch project page for extra details |
| `notion-fetch` | Fetch parent page for arxiv URL and content |
| `notion-search` | Find parent page or check for existing sub-page |
| `notion-create-pages` | Create the Technical Review sub-page |
| `notion-update-page` | Update sub-page content and parent callout |

---

## 7. init

**Description:** Discovers the user's Notion workspace structure (projects, databases, sub-pages) and writes a local config file (`.claude/notion-config.json`) that all other skills read at runtime. Re-runnable to sync new projects.

### Example Invocations

```
/init
```

### Required Inputs

| Field | Required | Notes |
|---|---|---|
| Workspace root URL | Yes | The Notion page containing all research projects |

### Outputs

- `.claude/notion-config.json` populated with workspace root ID and all discovered project IDs, database IDs, and sub-page IDs
- Summary report of discovered projects

### Key MCP Tools Called

| Tool | Purpose |
|---|---|
| `notion-fetch` | Fetch workspace root and each project page |
