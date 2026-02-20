---
name: scaffold-project
description: Create a new research project in the Notion Research Space with standard structure (Overview, Quick Links, Tasks DB, References DB, Meeting Notes).
---

# Skill: scaffold-project

This skill creates a new research project page in the Notion Research Space with the standard layout: a two-column header (Project Overview callout + Quick Links callout), a Meeting Notes sub-page, and inline Tasks and References databases.

---

## Step 0: Gather User Input

Ask the user for the following. Do not proceed until you have at minimum the project name and icon. The other fields have defaults.

| Field                | Required | Default        | Description                                                |
|----------------------|----------|----------------|------------------------------------------------------------|
| `project_name`       | Yes      | —              | The display name for the new project page                  |
| `icon_emoji`         | Yes      | —              | A single emoji to use as the Notion page icon              |
| `one_liner`          | No       | "Placeholder"  | One sentence describing what the project is about          |
| `application`        | No       | "Placeholder"  | Target application domain or use case                      |
| `proposed_solutions` | No       | "Placeholder"  | Brief description of the approach(es) being explored       |

If the user provides all fields inline with the slash command, skip the prompt and proceed directly to Step 1.

---

## Step 1: Read the Page Template

Read the template file at:

```
docs/templates/project-page.md
```

Extract only the content that appears **after** the line:

```
<!-- TEMPLATE BODY START — content below is written to the Notion page -->
```

The extracted template body is:

```
<columns>
	<column>
		::: callout {icon="💡" color="gray_bg"}
			### Project Overview
			**One Liner:** {one_liner}
			**Application:** {application}
			**Proposed Solutions:** {proposed_solutions}
		:::
	</column>
	<column>
		::: callout {icon="💡" color="green_bg"}
			### Quick Links
			<page url="{MEETING_NOTES_URL}">Meeting Notes</page>
		:::
	</column>
</columns>
```

---

## Step 2: Fetch the Enhanced Markdown Spec

Fetch the Notion enhanced Markdown specification from the MCP resource to ensure you use correct syntax when passing content to the Notion MCP tools:

```
notion://docs/enhanced-markdown-spec
```

Use the `notion-fetch` MCP tool or the MCP resource reader. Read and internalize the spec — pay particular attention to:
- How `<columns>` / `<column>` blocks are written
- How callout blocks (`::: callout`) are written including `icon` and `color` attributes
- How the `<page url="...">` tag creates a sub-page reference link

---

## Step 3: Create the Meeting Notes Sub-Page

Create the Meeting Notes sub-page first, using the Research Space page as a temporary parent. You will move it under the new project page in Step 5.

Use the `notion-create-pages` MCP tool with these parameters:

```json
{
  "parent_type": "page_id",
  "parent_id": "2306f2eb-947e-805e-9064-ed452a446342",
  "title": "Meeting Notes",
  "icon": "📝",
  "properties": {}
}
```

Record the URL of the newly created Meeting Notes page. It will be in the tool response as the page URL (e.g., `https://www.notion.so/Meeting-Notes-<id>`). This value becomes `{MEETING_NOTES_URL}`.

---

## Step 4: Build the Final Page Body

Substitute all placeholders in the template body extracted in Step 1:

| Placeholder            | Value                                              |
|------------------------|----------------------------------------------------|
| `{one_liner}`          | User-provided value, or `"Placeholder"`            |
| `{application}`        | User-provided value, or `"Placeholder"`            |
| `{proposed_solutions}` | User-provided value, or `"Placeholder"`            |
| `{MEETING_NOTES_URL}`  | The URL of the Meeting Notes page from Step 3      |

The `<page url="...">` tag in the Quick Links callout must use the exact Meeting Notes page URL so that Notion renders it as an inline page reference (not a plain hyperlink).

---

## Step 5: Create the Project Page Under Research Space

Use the `notion-create-pages` MCP tool with these parameters:

```json
{
  "parent_type": "page_id",
  "parent_id": "2306f2eb-947e-805e-9064-ed452a446342",
  "title": "<project_name>",
  "icon": "<icon_emoji>",
  "content": "<substituted template body from Step 4>"
}
```

- `parent_id` is always `2306f2eb-947e-805e-9064-ed452a446342` (the Research Space page).
- `title` is the `project_name` provided by the user.
- `icon` is the `icon_emoji` provided by the user.
- `content` is the full substituted Markdown body from Step 4, using enhanced Markdown syntax.

Record the URL and page ID of the newly created project page.

---

## Step 6: Move Meeting Notes Under the Project Page

Move the Meeting Notes sub-page (created in Step 3) to be a child of the new project page (created in Step 5).

Use the `notion-move-pages` MCP tool with these parameters:

```json
{
  "page_id": "<meeting_notes_page_id>",
  "new_parent_type": "page_id",
  "new_parent_id": "<project_page_id>"
}
```

- `page_id`: the ID of the Meeting Notes page from Step 3.
- `new_parent_id`: the ID of the project page from Step 5.

After this move, the Meeting Notes page will live inside the project page in the Notion sidebar, and the `<page>` link in the Quick Links callout will correctly point to it.

---

## Step 7: Add Inline Databases (Tasks and References)

The `notion-create-pages` tool cannot create new inline databases with custom schemas. Handle this as follows:

### Option A: Use `notion-create-database` (preferred if available)

If the `notion-create-database` MCP tool is available, create two databases under the project page:

**Tasks Database**

```json
{
  "parent_type": "page_id",
  "parent_id": "<project_page_id>",
  "title": "Tasks",
  "is_inline": true,
  "properties": {
    "Task": { "type": "title" },
    "Status": {
      "type": "status",
      "status": {
        "options": [
          { "name": "Not started", "color": "default" },
          { "name": "In progress", "color": "blue" },
          { "name": "Done", "color": "green" }
        ],
        "groups": [
          { "name": "to_do",       "color": "gray",  "option_ids": [] },
          { "name": "in_progress", "color": "blue",  "option_ids": [] },
          { "name": "complete",    "color": "green", "option_ids": [] }
        ]
      }
    },
    "Due": { "type": "date" },
    "Assigned To": { "type": "people" },
    "Week": {
      "type": "multi_select",
      "multi_select": {
        "options": [
          { "name": "Week 2" }, { "name": "Week 3" }, { "name": "Week 4" },
          { "name": "Week 5" }, { "name": "Week 6" }, { "name": "Week 7" },
          { "name": "Week 8" }, { "name": "Week 9" }
        ]
      }
    }
  }
}
```

**References Database**

```json
{
  "parent_type": "page_id",
  "parent_id": "<project_page_id>",
  "title": "References",
  "is_inline": true,
  "properties": {
    "Name": { "type": "title" },
    "Arxiv": { "type": "url" },
    "Conference": {
      "type": "multi_select",
      "multi_select": {
        "options": [
          { "name": "Science Robotics" },
          { "name": "ICLR" },
          { "name": "ICCV" }
        ]
      }
    },
    "Year": {
      "type": "select",
      "select": {
        "options": [
          { "name": "2023" }, { "name": "2024" },
          { "name": "2025" }, { "name": "2026" }
        ]
      }
    },
    "Github": { "type": "url" },
    "Project Page": { "type": "url" }
  }
}
```

### Option B: Manual creation (fallback)

If `notion-create-database` is not available, inform the user:

> The Notion MCP tools available in this session cannot create inline databases with custom schemas programmatically. Please add the Tasks and References databases manually inside the new project page.
>
> Use the schemas documented in:
> - `docs/samples/tasks-db-schema.md` — Tasks database (properties: Task/title, Status/status, Due/date, Assigned To/person, Week/multi_select)
> - `docs/samples/references-db-schema.md` — References database (properties: Name/title, Arxiv/url, Conference/multi_select, Year/select, Github/url, Project Page/url)

---

## Step 8: Report Results

Summarize what was created:

```
Project page created: <project_page_url>
  Icon: <icon_emoji>  Title: <project_name>

Sub-pages and databases:
  - Meeting Notes:  <meeting_notes_url>  [moved under project page]
  - Tasks DB:       [created inline] or [needs manual creation]
  - References DB:  [created inline] or [needs manual creation]
```

Provide the clickable project page URL so the user can open it directly in Notion.

---

## Reference: MCP Tool Names

| Tool                    | Usage in this skill                                          |
|-------------------------|--------------------------------------------------------------|
| `notion-create-pages`   | Create Meeting Notes sub-page; create the project page       |
| `notion-move-pages`     | Move Meeting Notes under the project page                    |
| `notion-create-database`| (Optional) Create Tasks and References inline databases      |
| `notion-fetch`          | Fetch MCP resource for enhanced Markdown spec (Step 2)       |

---

## Reference: Fixed IDs

| Name              | Notion Page ID                         |
|-------------------|----------------------------------------|
| Research Space    | `2306f2eb-947e-805e-9064-ed452a446342` |
