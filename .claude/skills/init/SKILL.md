---
name: init
description: Initialize workspace config by discovering Notion projects and their databases. Run once per user, re-runnable to sync.
---

# Skill: init

This skill discovers the user's Notion workspace structure and writes a local config file (`.claude/notion-config.json`) that all other skills read at runtime. It replaces all hardcoded Notion IDs with dynamically discovered ones.

---

## Step 0: Check for Existing Config

Read `.claude/notion-config.json` if it exists. If it does, load it as `existing_config` — new discoveries will be merged into it. If it does not exist, start with an empty config.

---

## Step 1: Get the Workspace Root

Ask the user for the URL of their workspace root page in Notion. This is the top-level page under which all research projects live.

Example:
```
Please provide the URL of your Notion workspace root page
(the page that contains all your research projects):
```

Use `notion-fetch` on the provided URL. Record the page ID from the response. This becomes `workspace_root` in the config.

---

## Step 2: Discover Child Projects

From the fetched workspace root page content, identify all child pages. These are the research project pages. Each will appear as a `<page>` tag or a linked page reference in the page body.

For each child page found, record:
- `name`: the page title
- `page_url`: the full Notion URL
- `page_id`: extracted from the URL

---

## Step 3: Crawl Each Project

For each project page discovered in Step 2, use `notion-fetch` on its URL. Inspect the page content to identify:

### Inline Databases

Look for `data-source-url` attributes on database blocks. Common databases:

| Database | Identifying keywords in title |
|----------|-------------------------------|
| Tasks | "Tasks" |
| References | "References" |
| Datasets | "Datasets" |

For each database found, extract the `collection_id` from the `data-source-url` (the UUID segment in the URL path before `?v=`). Record it under the project entry with the appropriate key (`tasks_db`, `references_db`, `datasets_db`).

### Sub-pages

Look for `<page>` tags in the content:

| Sub-page | Identifying keywords |
|----------|---------------------|
| Meeting Notes | "Meeting Notes" |
| Templates | "Templates" |

For each sub-page found, extract its page ID and record it under the project entry (`meeting_notes`, `templates`).

---

## Step 4: Merge with Existing Config

If `existing_config` was loaded in Step 0, merge the new discoveries:

- **`workspace_root`**: Always overwrite with the newly provided value.
- **`projects`**: For each discovered project:
  - If the project name already exists in config, update its fields with newly discovered values. Do NOT delete fields that exist in config but were not discovered (the user may have added them manually).
  - If the project name is new, add it.
- **Never delete** projects or fields that exist in the old config but were not found during this crawl. The user may have manually added entries for pages that `init` cannot auto-discover.

---

## Step 5: Write Config File

Write the merged config to `.claude/notion-config.json` with the following structure:

```json
{
  "workspace_root": "<page-id>",
  "projects": {
    "<Project Name>": {
      "page_id": "<page-id>",
      "tasks_db": "<collection-id>",
      "references_db": "<collection-id>",
      "datasets_db": "<collection-id>",
      "meeting_notes": "<page-id>",
      "templates": "<page-id>"
    }
  }
}
```

Rules:
- Only include fields that have values. Do not write `null` or empty strings.
- Use the Write tool to create/overwrite `.claude/notion-config.json`.
- Format the JSON with 2-space indentation for readability.

---

## Step 6: Report Results

Display a summary of what was discovered:

```
Workspace initialized!

Root page: <workspace_root_id>

Projects discovered (<N> total):
  <Project Name 1>
    page:          <page_id>
    tasks_db:      <id or "not found">
    references_db: <id or "not found">
    datasets_db:   <id or "not found">
    meeting_notes: <id or "not found">
    templates:     <id or "not found">

  <Project Name 2>
    ...

Config written to: .claude/notion-config.json
```

If this was a re-run, also note:
- How many new projects were added
- How many existing projects were updated
- That no projects were deleted

---

## Reference: MCP Tool Names

| Tool           | Usage in this skill                                    |
|----------------|--------------------------------------------------------|
| `notion-fetch` | Fetch workspace root page and each project page        |
