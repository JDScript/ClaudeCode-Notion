# Portable Notion Skills — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Remove all hardcoded Notion UUIDs from skills and docs, add a config file system with an `init` skill for workspace discovery, and add per-project template overrides.

**Architecture:** A gitignored `.claude/notion-config.json` file stores workspace and project IDs. A new `init` skill discovers the workspace and populates this config. All skills that previously used hardcoded IDs now read from config. Templates fall back from Notion project-level overrides to local `docs/templates/` defaults.

**Tech Stack:** Claude Code skills (Markdown), Notion MCP tools, JSON config

---

### Task 1: Add config file to gitignore

**Files:**
- Modify: `.gitignore`

**Step 1: Add the gitignore entry**

Add the following line to `.gitignore` under the existing "Claude Code local settings" section:

```
.claude/notion-config.json
```

The full section should read:

```
# Claude Code local settings
.claude/settings.local.json
.claude/notion-config.json
```

**Step 2: Commit**

```bash
git add .gitignore
git commit -m "chore: gitignore notion config file"
```

---

### Task 2: Create the `init` skill

**Files:**
- Create: `.claude/skills/init/SKILL.md`

**Step 1: Write the init skill**

Create `.claude/skills/init/SKILL.md` with the following content. This is the complete skill definition — write it exactly as shown.

````markdown
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
````

**Step 2: Commit**

```bash
git add .claude/skills/init/SKILL.md
git commit -m "feat: add init skill for workspace discovery"
```

---

### Task 3: Update `scaffold-project` skill — remove hardcoded IDs

**Files:**
- Modify: `.claude/skills/scaffold-project/SKILL.md`

**Context:** This skill has `2306f2eb-947e-805e-9064-ed452a446342` hardcoded in 3 places (Steps 3, 5, and the Reference table at the end). All references need to be replaced with config-based lookup.

**Step 1: Add config loading step**

Insert a new step between the existing Step 0 (Gather User Input) and Step 1 (Read the Page Template). This becomes the new "Step 1" and all subsequent steps shift by one.

Add the following after the `---` that ends Step 0:

```markdown
## Step 0.5: Load Config

Read `.claude/notion-config.json` from the repository root.

- If the file does not exist, stop and tell the user:
  > Config file not found. Please run `/init` first to set up your workspace.
- Extract `workspace_root` from the config. This is the parent page ID used to create the project page and temporary sub-pages.
```

**Step 2: Replace hardcoded IDs in Step 3 (Create Meeting Notes)**

In the current Step 3 (line ~84-97), replace the JSON example:

Replace:
```json
{
  "parent_type": "page_id",
  "parent_id": "2306f2eb-947e-805e-9064-ed452a446342",
  "title": "Meeting Notes",
  "icon": "📝",
  "properties": {}
}
```

With:
```json
{
  "parent_type": "page_id",
  "parent_id": "<workspace_root from config>",
  "title": "Meeting Notes",
  "icon": "📝",
  "properties": {}
}
```

**Step 3: Replace hardcoded IDs in Step 5 (Create Project Page)**

In the current Step 5 (line ~117-133), replace:

```json
{
  "parent_type": "page_id",
  "parent_id": "2306f2eb-947e-805e-9064-ed452a446342",
  "title": "<project_name>",
  "icon": "<icon_emoji>",
  "content": "<substituted template body from Step 4>"
}
```

With:
```json
{
  "parent_type": "page_id",
  "parent_id": "<workspace_root from config>",
  "title": "<project_name>",
  "icon": "<icon_emoji>",
  "content": "<substituted template body from Step 4>"
}
```

Also replace the line:
```
- `parent_id` is always `2306f2eb-947e-805e-9064-ed452a446342` (the Research Space page).
```
With:
```
- `parent_id` is the `workspace_root` value from `.claude/notion-config.json`.
```

**Step 4: Remove the Reference: Fixed IDs section**

Delete the entire "Reference: Fixed IDs" section at the bottom of the file (lines ~285-289):

```markdown
## Reference: Fixed IDs

| Name              | Notion Page ID                         |
|-------------------|----------------------------------------|
| Research Space    | `2306f2eb-947e-805e-9064-ed452a446342` |
```

**Step 5: Commit**

```bash
git add .claude/skills/scaffold-project/SKILL.md
git commit -m "refactor: remove hardcoded IDs from scaffold-project skill"
```

---

### Task 4: Update `manage-tasks` skill — replace example IDs

**Files:**
- Modify: `.claude/skills/manage-tasks/SKILL.md`

**Context:** This skill discovers IDs dynamically (search → fetch → extract). The only hardcoded IDs are in two JSON examples at lines 122 and 135 that show specific collection IDs.

**Step 1: Replace the minimal create example**

Replace `"parent_id": "2ab6f2eb-947e-800c-af0c-000b3a250c50"` in the minimal example (line ~122) with `"parent_id": "<collection_id_from_step_2>"`.

The full example block should become:

```json
{
  "parent_type": "data_source_id",
  "parent_id": "<collection_id_from_step_2>",
  "title": "Draft literature review",
  "properties": {
    "Status": "Not started"
  }
}
```

**Step 2: Replace the full create example**

Replace `"parent_id": "2ab6f2eb-947e-800c-af0c-000b3a250c50"` in the full example (line ~135) with `"parent_id": "<collection_id_from_step_2>"`.

The full example block should become:

```json
{
  "parent_type": "data_source_id",
  "parent_id": "<collection_id_from_step_2>",
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

**Step 3: Commit**

```bash
git add .claude/skills/manage-tasks/SKILL.md
git commit -m "refactor: remove hardcoded IDs from manage-tasks skill"
```

---

### Task 5: Update `read-dataset` skill — replace hardcoded Datasets DB ID

**Files:**
- Modify: `.claude/skills/read-dataset/SKILL.md`

**Context:** This skill hardcodes `30d6f2eb-947e-80d8-82f8-000b470f76d4` (Nav Benchmark Datasets DB) in Step 4 for both searching and creating entries. It needs to resolve the Datasets DB ID from config or dynamically.

**Step 1: Add config loading to Step 1**

After the existing Step 1 content (Identify the Dataset), add a note about resolving the target project. Insert the following at the end of Step 1, after "Proceed to Step 2; the target page will be resolved in Step 4.":

```markdown
**Resolving the target Datasets DB:**

If the user specifies which project the dataset belongs to (or if it can be inferred), read `.claude/notion-config.json` and look up the project's `datasets_db` collection ID. If the config file does not exist, stop and tell the user to run `/init` first. If the project does not have a `datasets_db` entry in config, ask the user which project's Datasets DB to use.
```

**Step 2: Replace hardcoded ID in Step 4**

In Step 4 (line ~158), replace:

```
Search the Datasets DB (collection ID: `30d6f2eb-947e-80d8-82f8-000b470f76d4`) for an existing entry whose `Arxiv` property matches the arxiv URL.
```

With:

```
Search the project's Datasets DB (collection ID from `.claude/notion-config.json` → `projects.<project_name>.datasets_db`) for an existing entry whose `Arxiv` property matches the arxiv URL.
```

**Step 3: Replace hardcoded ID in the create example**

In Step 4 (line ~166), replace:

```json
{
  "parent": {"data_source_id": "30d6f2eb-947e-80d8-82f8-000b470f76d4"},
  "pages": [{
    "properties": {
      "Name": "<dataset_name>",
      "Arxiv": "<arxiv_url>"
    }
  }]
}
```

With:

```json
{
  "parent": {"data_source_id": "<datasets_db_collection_id from config>"},
  "pages": [{
    "properties": {
      "Name": "<dataset_name>",
      "Arxiv": "<arxiv_url>"
    }
  }]
}
```

**Step 4: Commit**

```bash
git add .claude/skills/read-dataset/SKILL.md
git commit -m "refactor: remove hardcoded IDs from read-dataset skill"
```

---

### Task 6: Add per-project template fallback to template-using skills

**Files:**
- Modify: `.claude/skills/read-paper/SKILL.md`
- Modify: `.claude/skills/read-dataset/SKILL.md`
- Modify: `.claude/skills/meeting-notes/SKILL.md`
- Modify: `.claude/skills/scaffold-project/SKILL.md`
- Modify: `.claude/skills/extract-technical-details/SKILL.md`

**Context:** Each of these skills has a "Read Template" step that reads from `docs/templates/<name>.md`. We need to add a fallback chain: check Notion for a project-specific template first, then fall back to the local file.

**Step 1: Add template fallback to read-paper**

In `.claude/skills/read-paper/SKILL.md`, locate Step 3a (line ~130-146) which says "Read the file at: `docs/templates/paper-summary.md`". Replace the step's opening paragraph with:

```markdown
### 3a. Read the template

**Template fallback chain:**

1. Read `.claude/notion-config.json`. If the target project has a `templates` page ID, use `notion-fetch` to load that page and look for a child page titled "Paper Summary Template". If found, use its content as the template.
2. If no project-specific template is found, read the local default at `docs/templates/paper-summary.md`.

Extract only the content that appears **after** the line:
```

**Step 2: Add template fallback to read-dataset**

In `.claude/skills/read-dataset/SKILL.md`, locate Step 3a (line ~93-127) which has the template structure inline. Add before "The summary follows this template:":

```markdown
### 3a. Read the template

**Template fallback chain:**

1. Read `.claude/notion-config.json`. If the target project has a `templates` page ID, use `notion-fetch` to load that page and look for a child page titled "Dataset Summary Template". If found, use its content as the template.
2. If no project-specific template is found, use the default template below.
```

**Step 3: Add template fallback to meeting-notes**

In `.claude/skills/meeting-notes/SKILL.md`, locate Step 3 (line ~83-114). Replace the step's opening text with:

```markdown
## Step 3: Read the Template

**Template fallback chain:**

1. Read `.claude/notion-config.json`. If the target project has a `templates` page ID, use `notion-fetch` to load that page and look for a child page titled "Meeting Notes Template". If found, use its content as the template.
2. If no project-specific template is found, read the local default at `docs/templates/meeting-notes.md`.

Extract only the content that appears **after** the line:
```

**Step 4: Add template fallback to scaffold-project**

In `.claude/skills/scaffold-project/SKILL.md`, locate Step 1 (Read the Page Template). Add before "Read the template file at:":

```markdown
**Template fallback chain:**

1. Read `.claude/notion-config.json`. If a project-specific template for scaffolding exists (unlikely for new projects, but possible if the user pre-configured one), use it.
2. Otherwise, read the local default below.
```

**Step 5: Add template fallback to extract-technical-details**

In `.claude/skills/extract-technical-details/SKILL.md`, locate Step 3a (line ~80-86). Replace with:

```markdown
### 3a. Read the template file

**Template fallback chain:**

1. Read `.claude/notion-config.json`. If the target project has a `templates` page ID, use `notion-fetch` to load that page and look for a child page titled "Technical Review Template". If found, use its content as the template.
2. If no project-specific template is found, read the local default at `docs/templates/technical-review-template.md`.

Extract the content after the `<!-- TEMPLATE BODY START -->` marker.
```

**Step 6: Commit**

```bash
git add .claude/skills/read-paper/SKILL.md .claude/skills/read-dataset/SKILL.md .claude/skills/meeting-notes/SKILL.md .claude/skills/scaffold-project/SKILL.md .claude/skills/extract-technical-details/SKILL.md
git commit -m "feat: add per-project template fallback chain to all skills"
```

---

### Task 7: Update documentation — remove hardcoded IDs

**Files:**
- Modify: `docs/samples/references-db-schema.md`
- Modify: `docs/samples/tasks-db-schema.md`
- Modify: `docs/samples/datasets-db-schema.md`
- Modify: `docs/architecture/overview.md`
- Modify: `docs/skills/catalog.md`

**Step 1: Update references-db-schema.md**

Replace lines 9-13:

```markdown
Used by: Functional Reorientation, Navigation Reasoning Benchmark, and all new projects.

Example collection IDs:
- Functional Reorientation: `2ab6f2eb-947e-80f0-8f62-000b76f8f4eb`
- Navigation Reasoning Benchmark: `30d6f2eb-947e-80ed-958b-000ba8100415`
```

With:

```markdown
Used by all research projects. Collection IDs are stored in `.claude/notion-config.json` under each project's `references_db` field (populated by the `init` skill).
```

**Step 2: Update tasks-db-schema.md**

Replace lines 9-10:

```markdown
Source project: Functional Reorientation
Collection ID: `2ab6f2eb-947e-800c-af0c-000b3a250c50`
```

With:

```markdown
Collection IDs are stored in `.claude/notion-config.json` under each project's `tasks_db` field (populated by the `init` skill).
```

Also replace line 13:

```sql
CREATE TABLE "collection://2ab6f2eb-947e-800c-af0c-000b3a250c50" (
```

With:

```sql
CREATE TABLE tasks (
```

**Step 3: Update datasets-db-schema.md**

Replace lines 9-10:

```markdown
Example collection ID:
- Navigation Reasoning Benchmark: `30d6f2eb-947e-80d8-82f8-000b470f76d4`
```

With:

```markdown
Collection IDs are stored in `.claude/notion-config.json` under each project's `datasets_db` field (populated by the `init` skill).
```

**Step 4: Update architecture/overview.md**

In `docs/architecture/overview.md`:

a) Line 9: Change `Skills (4 total)` to `Skills (7 total)` and add `init`, `read-dataset`, `extract-technical-details` to the table.

b) Line 27: Add `technical-review-template.md` to the templates table and `datasets-db-schema.md` to the samples table.

c) Line 62: Replace:
```
The top-level anchor is the Research Space page (page ID: `2306f2eb-947e-805e-9064-ed452a446342`).
```
With:
```
The top-level anchor is the workspace root page (page ID stored in `.claude/notion-config.json` as `workspace_root`, populated by the `init` skill).
```

d) Add a new section after "Template-Driven Generation" about per-project templates:

```markdown
## Per-Project Template Overrides

Skills that generate content from templates support per-project overrides stored in Notion. Each project can optionally have a "Templates" sub-page containing child pages named by template type (e.g., "Paper Summary Template", "Meeting Notes Template"). When present, the skill uses the Notion-hosted template instead of the local default in `docs/templates/`. The `init` skill discovers Templates sub-pages automatically and records their page IDs in the config file.
```

**Step 5: Update skills/catalog.md**

a) Line 1: Change "all four" to "all seven".

b) Line 35: Replace:
```
- A new project page under the Research Space (page ID: `2306f2eb-947e-805e-9064-ed452a446342`)
```
With:
```
- A new project page under the workspace root (page ID from `.claude/notion-config.json`)
```

c) Add sections for the three missing skills (init, read-dataset, extract-technical-details) at the appropriate positions in the catalog.

**Step 6: Commit**

```bash
git add docs/samples/references-db-schema.md docs/samples/tasks-db-schema.md docs/samples/datasets-db-schema.md docs/architecture/overview.md docs/skills/catalog.md
git commit -m "docs: remove hardcoded IDs, update for 7 skills and config system"
```

---

### Task 8: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Update CLAUDE.md**

a) Replace the workspace root ID line:
```
Research Space page ID: `2306f2eb-947e-805e-9064-ed452a446342`
```
With:
```
Workspace root page ID is stored in `.claude/notion-config.json` (run `/init` to populate).
```

b) Add `init` to the Skills table:
```
| `init` | Discover workspace, populate config | `/init` |
```

c) Add to the Key Conventions section:
```
5. **Config-driven**: All workspace and project IDs live in `.claude/notion-config.json` (gitignored, per-user). Run `/init` to populate. Skills read this file instead of hardcoding IDs.
6. **Per-project template overrides**: Projects can have a "Templates" sub-page in Notion. Skills check for project-specific templates before falling back to `docs/templates/` defaults.
```

d) Add `.claude/notion-config.json` mention to File Structure:
```
- `.claude/notion-config.json` — Per-user workspace config (gitignored, generated by `/init`)
```

**Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for config-driven portable skills"
```

---

### Verification Checklist

After all tasks are complete, verify:

1. `grep -r "2306f2eb" --include="*.md" .` returns hits only in `docs/plans/` (historical) and nowhere in skills or active docs
2. `grep -r "2ab6f2eb" --include="*.md" .` returns hits only in `docs/plans/`
3. `grep -r "30d6f2eb" --include="*.md" .` returns hits only in `docs/plans/`
4. `.claude/notion-config.json` is listed in `.gitignore`
5. `.claude/skills/init/SKILL.md` exists and is well-formed
6. All 5 template-using skills mention the template fallback chain
