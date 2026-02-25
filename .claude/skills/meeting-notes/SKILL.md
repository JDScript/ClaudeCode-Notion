---
name: meeting-notes
description: Create structured meeting notes under a research project's Meeting Notes section in Notion.
---

# Skill: meeting-notes

This skill creates a new meeting notes page under a research project's Meeting Notes sub-page in Notion, using the standard meeting notes template. It finds the project, ensures the Meeting Notes sub-page exists, and creates the new page with filled-in template content.

---

## Step 0: Gather User Input

Accept the following from the user. The project name is required. All other fields are optional — if not provided, use the placeholder values shown below.

| Field           | Required | Default / Placeholder              | Description                                              |
|-----------------|----------|------------------------------------|----------------------------------------------------------|
| `project_name`  | Yes      | —                                  | Name of the research project in Notion                   |
| `topic`         | No       | `"Meeting"`                        | The meeting topic or title prefix                        |
| `attendees`     | No       | `"attendee_list"`                  | Names of meeting participants, one per line              |
| `agenda_items`  | No       | `"agenda_items"`                   | Numbered list of agenda items                            |

If the user provides all fields inline with the slash command, skip prompting and proceed directly to Step 1.

Today's date is used for the page title. Format: abbreviated month, zero-padded day, four-digit year (e.g., `Feb 20, 2026`).

---

## Step 1: Find the Project Page

Use `notion-search` to locate the project page by name:

```json
{
  "query": "<project_name>"
}
```

From the search results, identify the Notion page that matches the project. Record its page URL (e.g., `https://www.notion.so/Project-Name-<id>`).

If no matching project page is found, report the error to the user and stop.

---

## Step 2: Fetch the Project Page and Find the Meeting Notes Sub-Page

Use `notion-fetch` on the project page URL found in Step 1:

```json
{
  "url": "<project_page_url>"
}
```

Inspect the fetched content for a `<page>` tag whose display text contains "Meeting Notes". For example:

```
<page url="https://www.notion.so/Meeting-Notes-<id>">Meeting Notes</page>
```

**If a Meeting Notes sub-page is found:**

Record the URL from the `url` attribute and extract its page ID. This is the parent for the new meeting notes page.

**If no Meeting Notes sub-page is found:**

Create one using `notion-create-pages` under the project page:

```json
{
  "parent_type": "page_id",
  "parent_id": "<project_page_id>",
  "title": "Meeting Notes",
  "icon": "📝",
  "properties": {}
}
```

Record the URL and page ID of the newly created Meeting Notes sub-page.

---

## Step 3: Read the Template

**Template fallback chain:**

1. Read `.claude/notion-config.json`. If the target project has a `templates` page ID, use `notion-fetch` to load that page and look for a child page titled "Meeting Notes Template". If found, use its content as the template.
2. If no project-specific template is found, read the local default at `docs/templates/meeting-notes.md`.

Extract only the content that appears **after** the line:

```
<!-- TEMPLATE BODY START — content below is written to the Notion page -->
```

The extracted template body is:

```
## Attendees
- {attendee_list}

## Agenda
1. {agenda_items}

## Discussion
{discussion_content}

## Action Items
- [ ] {action_items}

## Decisions
{decisions}
```

---

## Step 4: Build the Page Title and Content

### Title

Construct the page title using the following rules:

- If a `topic` was provided by the user: `"<topic> - <Mon DD, YYYY>"`
- If no topic was provided: `"Meeting - <Mon DD, YYYY>"`

Use today's date formatted as abbreviated month + zero-padded day + four-digit year. Examples:
- `"Preliminary Meeting 1 - Feb 20, 2026"`
- `"Meeting - Feb 20, 2026"`

### Content

Substitute all `{placeholder}` variables in the extracted template body:

| Placeholder            | Value                                                                 |
|------------------------|-----------------------------------------------------------------------|
| `{attendee_list}`      | User-provided attendees (one name per bullet), or `"attendee_list"`   |
| `{agenda_items}`       | User-provided agenda items (one per numbered line), or `"agenda_items"` |
| `{discussion_content}` | Always leave as `"discussion_content"` (filled in after the meeting)  |
| `{action_items}`       | Always leave as `"action_items"` (filled in after the meeting)        |
| `{decisions}`          | Always leave as `"decisions"` (filled in after the meeting)           |

If the user provided multiple attendees, format them as separate bullet lines:

```
## Attendees
- Alice
- Bob
- Carol
```

If the user provided multiple agenda items, format them as a numbered list:

```
## Agenda
1. Review last week's results
2. Discuss experimental plan
3. Assign action items
```

Action items use Notion to-do checkbox syntax (`- [ ]`). The template placeholder line `- [ ] {action_items}` is left as-is when no action items are provided. If the user supplied action items, replace `{action_items}` with the item text and add additional `- [ ]` lines for each item.

---

## Step 5: Create the Meeting Notes Page

Use `notion-create-pages` with the Meeting Notes sub-page as the parent:

```json
{
  "parent_type": "page_id",
  "parent_id": "<meeting_notes_subpage_id>",
  "title": "<constructed title from Step 4>",
  "content": "<substituted template body from Step 4>"
}
```

- `parent_id`: the page ID of the Meeting Notes sub-page (found or created in Step 2).
- `title`: the full title string constructed in Step 4 (e.g., `"Preliminary Meeting 1 - Feb 20, 2026"`).
- `content`: the complete substituted Markdown body, using the template structure with placeholders replaced.

Record the URL of the newly created page.

---

## Step 6: Report Results

Report the outcome to the user:

```
Meeting notes page created: <created_page_url>
  Title:   <page_title>
  Parent:  <meeting_notes_subpage_url>
  Project: <project_name>
```

Provide the clickable URL so the user can open the new meeting notes page directly in Notion.

---

## Reference: MCP Tool Names

| Tool                  | Usage in this skill                                            |
|-----------------------|----------------------------------------------------------------|
| `notion-search`       | Find the project page by name (Step 1)                         |
| `notion-fetch`        | Fetch the project page to locate the Meeting Notes sub-page (Step 2) |
| `notion-create-pages` | Create the Meeting Notes sub-page if missing (Step 2); create the new meeting notes page (Step 5) |
