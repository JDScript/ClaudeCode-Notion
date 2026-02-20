# Notion Research Management — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a Skills-First Notion research project management system with 4 Claude Code skills, supporting templates, documentation, and a properly configured CLAUDE.md.

**Architecture:** All functionality in `.claude/skills/` directories (SKILL.md per skill). Templates in `docs/templates/` define Notion page content structure. DB schema snapshots in `docs/samples/` serve as reference. No Python code — pure skills + Notion MCP.

**Tech Stack:** Claude Code skills (SKILL.md format), Notion MCP (notion-search, notion-fetch, notion-create-pages, notion-update-page), Notion-flavored Markdown.

---

## Key Reference IDs (from Notion workspace)

- **Research Space** page: `2306f2eb-947e-805e-9064-ed452a446342`
- **Functional Reorientation** page: `2a36f2eb-947e-8018-a0ee-c70cc2a319ae`
  - Tasks DB data source: `collection://2ab6f2eb-947e-800c-af0c-000b3a250c50`
  - References DB data source: `collection://2ab6f2eb-947e-80f0-8f62-000b76f8f4eb`
- **Navigation Reasoning Benchmark** page: `30d6f2eb-947e-80e9-aa5a-fee50297f0c8`
  - References DB data source: `collection://30d6f2eb-947e-80ed-958b-000ba8100415`

---

### Task 1: Create directory structure

**Files:**
- Create: `.claude/skills/` (directory)
- Create: `docs/architecture/` (directory)
- Create: `docs/templates/` (directory)
- Create: `docs/samples/` (directory)
- Create: `docs/skills/` (directory)

**Step 1: Create all directories**

```bash
mkdir -p .claude/skills/read-paper
mkdir -p .claude/skills/manage-tasks
mkdir -p .claude/skills/meeting-notes
mkdir -p .claude/skills/scaffold-project
mkdir -p docs/architecture
mkdir -p docs/templates
mkdir -p docs/samples
mkdir -p docs/skills
```

**Step 2: Update .gitignore**

Add to `.gitignore`:
```
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info

# Virtual environments
.venv

# Claude Code local settings
.claude/settings.local.json
```

**Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: create project directory structure and update .gitignore"
```

---

### Task 2: Write DB schema snapshots

**Files:**
- Create: `docs/samples/references-db-schema.md`
- Create: `docs/samples/tasks-db-schema.md`

**Step 1: Write References DB schema**

Create `docs/samples/references-db-schema.md` documenting the standard References DB schema (from Functional Reorientation) as well as the simpler Navigation Reasoning Benchmark variant. Include the SQLite table definition, property types, and select/multi-select options.

Standard schema (Functional Reorientation):
```sql
CREATE TABLE references (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arxiv" TEXT,           -- URL
    "Year" TEXT,            -- select: ["2023", "2024", "2025", "2026"]
    "Project Page" TEXT,    -- URL
    "Github" TEXT,          -- URL
    "Conference" TEXT,      -- multi_select: ["Science Robotics", "ICLR", "ICCV"]
    "Name" TEXT             -- title
)
```

Nav Benchmark variant (simpler):
```sql
CREATE TABLE references (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arixv" TEXT,           -- URL (note: typo "Arixv" in actual DB)
    "Project Page" TEXT,    -- URL
    "Name" TEXT             -- title
)
```

**Step 2: Write Tasks DB schema**

Create `docs/samples/tasks-db-schema.md` documenting the standard Tasks DB schema (from Functional Reorientation). Include the SQLite table definition, status options, and board views.

Standard schema:
```sql
CREATE TABLE tasks (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Assigned To" TEXT,         -- person (JSON array of user IDs)
    "date:Due:start" TEXT,      -- ISO-8601 date
    "date:Due:end" TEXT,        -- ISO-8601 date (optional, for ranges)
    "date:Due:is_datetime" INT, -- 0=date, 1=datetime
    "Status" TEXT,              -- status: ["Not started", "In progress", "Done"]
    "Week" TEXT,                -- multi_select: ["Week 2".."Week 9"]
    "Task" TEXT                 -- title
)
```

Views: Board (grouped by Status), Table (grouped by Week).

**Step 3: Commit**

```bash
git add docs/samples/
git commit -m "docs: add Notion DB schema snapshots for References and Tasks"
```

---

### Task 3: Write Notion content templates

**Files:**
- Create: `docs/templates/paper-summary.md`
- Create: `docs/templates/project-page.md`
- Create: `docs/templates/meeting-notes.md`

**Step 1: Write paper summary template**

Create `docs/templates/paper-summary.md`. This defines the Notion page content structure for a paper in the References DB. Key sections:

```markdown
# Paper Summary Template

This template defines the Notion-flavored Markdown content written to a paper's
page in the References database.

## Content Structure

The page content has two clearly separated zones:

### Zone 1: AI-Generated (regenerable)

- **AI Summary** heading — contains all Claude-generated content
  - One-liner (1 sentence summary)
  - Problem & Motivation
  - Method / Approach
  - Key Results
  - Relevance to Project (connections to other papers, relevance to project goals)

### Zone 2: User Notes (protected)

- **My Notes** heading — user-written content, NEVER overwritten

## Notion Markdown Template

(include the exact Notion-flavored Markdown that the read-paper skill should output)
```

The template should contain the exact Notion-flavored Markdown block with placeholder variables like `{title}`, `{one_liner}`, `{problem}`, `{method}`, `{results}`, `{relevance}`.

**Step 2: Write project page template**

Create `docs/templates/project-page.md`. This defines the structure for a new research project page, modeled after Functional Reorientation:

```
Two-column layout:
  Left column:
    - Project Overview callout (gray_bg, icon 💡): heading + placeholder text
  Right column:
    - Quick Links callout (green_bg, icon 💡): heading + Meeting Notes page link

Inline Tasks DB (icon ☑️) with standard schema
Inline References DB (icon 📚) with standard schema
```

Include the exact Notion-flavored Markdown for the page content (using `<columns>`, `<column>`, `::: callout`, `<database>` tags).

**Step 3: Write meeting notes template**

Create `docs/templates/meeting-notes.md`. This defines the structure for a meeting notes page:

```
## Attendees
(placeholder)

## Agenda
1. (placeholder)

## Discussion
(placeholder)

## Action Items
- [ ] (placeholder)

## Decisions
(placeholder)
```

**Step 4: Commit**

```bash
git add docs/templates/
git commit -m "docs: add Notion content templates for papers, projects, and meetings"
```

---

### Task 4: Write scaffold-project skill

**Files:**
- Create: `.claude/skills/scaffold-project/SKILL.md`

**Step 1: Write the skill file**

Create `.claude/skills/scaffold-project/SKILL.md` with frontmatter:

```yaml
---
name: scaffold-project
description: Create a new research project in the Notion Research Space with standard structure (Overview, Quick Links, Tasks DB, References DB, Meeting Notes).
---
```

The skill content should instruct Claude to:

1. Accept project name and optional icon from the user
2. Read the template from `docs/templates/project-page.md`
3. Fetch the enhanced Markdown spec from `notion://docs/enhanced-markdown-spec` MCP resource
4. Create the project page under Research Space (page ID: `2306f2eb-947e-805e-9064-ed452a446342`) using `notion-create-pages`
5. The page content uses the two-column layout with callouts
6. After page creation, create Meeting Notes sub-page under the new project page
7. Note: Inline databases (Tasks, References) cannot be created via the MCP create-pages tool directly — they need to be created separately using `notion-create-database` tool or the user creates them manually
8. Report the created page URL back to the user

Important implementation detail: The Notion MCP `create-pages` tool can include `<database>` tags in content to reference existing databases, but creating NEW inline databases requires checking if `notion-create-database` is available. If not, instruct the user to manually add the Tasks and References databases using the templates as a guide.

**Step 2: Verify skill file is valid**

```bash
cat .claude/skills/scaffold-project/SKILL.md | head -5
# Should show:
# ---
# name: scaffold-project
# description: Create a new research project...
# ---
```

**Step 3: Commit**

```bash
git add .claude/skills/scaffold-project/
git commit -m "feat: add scaffold-project skill for creating new research projects"
```

---

### Task 5: Write read-paper skill

**Files:**
- Create: `.claude/skills/read-paper/SKILL.md`

**Step 1: Write the skill file**

Create `.claude/skills/read-paper/SKILL.md` with frontmatter:

```yaml
---
name: read-paper
description: Auto-read an arxiv paper and generate a structured summary in Notion. Extracts metadata (Conference, Year, Authors) and writes AI Summary while preserving user notes.
---
```

The skill content should instruct Claude to:

1. Accept an arxiv URL (or a Notion page URL of a References DB entry that has an arxiv link)
2. If given a Notion page URL, fetch the page to extract the arxiv link
3. Fetch the arxiv paper:
   - Use `WebFetch` on the arxiv abstract page (e.g., `https://arxiv.org/abs/XXXX.XXXXX`) to get title, authors, abstract
   - Use `WebFetch` on the arxiv HTML page (e.g., `https://arxiv.org/html/XXXX.XXXXXv1`) for full paper content if available
4. Read the template from `docs/templates/paper-summary.md`
5. Generate the structured summary following the template
6. Check if the Notion page already has content:
   - If the page has a `## My Notes` section, use `replace_content_range` on `notion-update-page` to only replace the `## AI Summary...## My Notes` range
   - If the page is empty/new, use `replace_content` to write the full template (AI Summary + empty My Notes section)
7. Update DB properties via `notion-update-page` with `update_properties`:
   - Extract and fill: Year (select), Conference (multi_select) if identifiable
   - Note: Author field may need to be added to the DB schema first
8. Search the same References DB for related papers and mention connections in the Relevance section
9. Report completion with a summary of what was extracted

Key convention: The `## AI Summary` section is bounded by the start of `## AI Summary` and the start of `## My Notes`. Everything between these markers is regenerable. Everything at and after `## My Notes` is protected.

**Step 2: Verify skill file**

```bash
cat .claude/skills/read-paper/SKILL.md | head -5
```

**Step 3: Commit**

```bash
git add .claude/skills/read-paper/
git commit -m "feat: add read-paper skill for arxiv paper reading and summarization"
```

---

### Task 6: Write manage-tasks skill

**Files:**
- Create: `.claude/skills/manage-tasks/SKILL.md`

**Step 1: Write the skill file**

Create `.claude/skills/manage-tasks/SKILL.md` with frontmatter:

```yaml
---
name: manage-tasks
description: Create, update, and query tasks in a research project's Notion Tasks database. Supports natural language commands.
---
```

The skill content should instruct Claude to:

1. Parse the user's natural language intent (create / update / query / list)
2. Identify the target project by name using `notion-search`
3. Fetch the project page using `notion-fetch` to find the Tasks DB
4. Reference `docs/samples/tasks-db-schema.md` for the schema
5. For **create**: Use `notion-create-pages` with parent `data_source_id` from the Tasks DB. Default Status = "Not started"
6. For **update**: Search for the task by name in the DB, then use `notion-update-page` with `update_properties`
7. For **query/list**: Use `notion-search` within the project page, or fetch the Tasks DB and query
8. Report results in a readable format

**Step 2: Verify and commit**

```bash
git add .claude/skills/manage-tasks/
git commit -m "feat: add manage-tasks skill for Notion task CRUD operations"
```

---

### Task 7: Write meeting-notes skill

**Files:**
- Create: `.claude/skills/meeting-notes/SKILL.md`

**Step 1: Write the skill file**

Create `.claude/skills/meeting-notes/SKILL.md` with frontmatter:

```yaml
---
name: meeting-notes
description: Create structured meeting notes under a research project's Meeting Notes section in Notion.
---
```

The skill content should instruct Claude to:

1. Accept the project name and optional meeting title/topic
2. Search for the project using `notion-search`
3. Fetch the project page to find the Meeting Notes sub-page
4. Read the template from `docs/templates/meeting-notes.md`
5. Create a new page under Meeting Notes using `notion-create-pages` with:
   - Parent: the Meeting Notes page ID
   - Title: `{Topic} - {Mon DD, YYYY}` or `Meeting - {Mon DD, YYYY}` if no topic given
   - Content: meeting notes template
6. If the Meeting Notes sub-page doesn't exist, create it first under the project page
7. Report the created page URL

**Step 2: Verify and commit**

```bash
git add .claude/skills/meeting-notes/
git commit -m "feat: add meeting-notes skill for creating structured meeting notes"
```

---

### Task 8: Write architecture overview and skills catalog

**Files:**
- Create: `docs/architecture/overview.md`
- Create: `docs/skills/catalog.md`

**Step 1: Write architecture overview**

Create `docs/architecture/overview.md` covering:
- System overview (Skills-First architecture)
- Data flow diagram (User → Claude Code skill → Notion MCP → Notion workspace)
- Component map (skills, templates, samples, CLAUDE.md)
- Notion workspace structure summary with key IDs
- Content zone convention (AI Summary vs My Notes)
- Template-driven generation explanation

**Step 2: Write skills catalog**

Create `docs/skills/catalog.md` listing all 4 skills with:
- Skill name and description
- Example invocations
- Required inputs and outputs
- Which templates and samples each skill uses

**Step 3: Commit**

```bash
git add docs/architecture/ docs/skills/
git commit -m "docs: add architecture overview and skills catalog"
```

---

### Task 9: Write CLAUDE.md

**Files:**
- Create: `CLAUDE.md`

**Step 1: Write CLAUDE.md**

Create `CLAUDE.md` at project root with:

```markdown
# CLAUDE.md

## Project Overview

Notion Research Project Management — a Skills-First system for managing
research projects in Notion via Claude Code skills + Notion MCP.

Scoped to the Research Space workspace. No Python code — all functionality
lives in `.claude/skills/`.

## Notion Workspace

Research Space page ID: `2306f2eb-947e-805e-9064-ed452a446342`

Projects follow a standard structure (modeled after Functional Reorientation):
- Project Overview callout + Quick Links callout (two-column)
- Inline Tasks DB (schema: Task, Status, Due, Assigned To, Week)
- Inline References DB (schema: Name, Arxiv, Conference, Year, Github, Project Page)
- Meeting Notes sub-page

## Skills

| Skill | Purpose |
|-------|---------|
| `read-paper` | Read arxiv paper, generate summary, extract metadata |
| `manage-tasks` | CRUD operations on project Tasks DB |
| `meeting-notes` | Create structured meeting notes |
| `scaffold-project` | Create new research project with standard structure |

## Key Conventions

1. **Content zones**: AI-generated content under `## AI Summary`, user notes under `## My Notes`. Regeneration NEVER touches `## My Notes`.
2. **Template-driven**: All Notion content follows templates in `docs/templates/`.
3. **Schema reference**: DB schemas documented in `docs/samples/`.

## File Structure

- `.claude/skills/` — Claude Code skill definitions
- `docs/templates/` — Notion page content templates
- `docs/samples/` — DB schema snapshots
- `docs/architecture/` — System architecture docs
- `docs/skills/` — Skills catalog and documentation
- `docs/plans/` — Design and implementation plans
```

**Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with project instructions for Claude Code"
```

---

### Task 10: Write README.md

**Files:**
- Modify: `README.md`

**Step 1: Write README**

Write `README.md` with:
- Project title and one-line description
- Quick start (how to use the skills)
- Links to all docs:
  - `docs/architecture/overview.md`
  - `docs/skills/catalog.md`
  - `docs/templates/` (with links to each template)
  - `docs/samples/` (with links to each schema)
  - `docs/plans/` (design doc)
- Brief description of each skill with example usage

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with project overview and documentation links"
```

---

### Task 11: Integration verification

**Step 1: Verify all files exist**

```bash
find . -type f -not -path './.git/*' -not -path './.venv/*' | sort
```

Expected output should list all created files matching the design doc structure.

**Step 2: Test read-paper skill against Notion**

Invoke the `read-paper` skill with the paper already in Navigation Reasoning Benchmark's References DB: "Thinking in 360°: Humanoid Visual Search in the Wild" (page ID: `30d6f2eb-947e-80f3-8d10-d6c684c2ed2d`). Verify:
- Paper content is fetched
- AI Summary section is written to the Notion page
- My Notes section is present and empty
- DB properties are updated if possible

**Step 3: Test manage-tasks skill**

Invoke `manage-tasks` to create a test task in an existing project. Verify the task appears in the Notion Tasks DB.

**Step 4: Final commit if any fixes needed**

```bash
git add -A
git commit -m "fix: integration verification fixes"
```

---

## Task Dependency Graph

```
Task 1 (dirs)
  ├── Task 2 (samples) ──┐
  ├── Task 3 (templates) ─┤
  │                        ├── Task 4 (scaffold-project skill)
  │                        ├── Task 5 (read-paper skill)
  │                        ├── Task 6 (manage-tasks skill)
  │                        └── Task 7 (meeting-notes skill)
  │                              │
  │                              ├── Task 8 (architecture + catalog)
  │                              ├── Task 9 (CLAUDE.md)
  │                              └── Task 10 (README.md)
  │                                    │
  └─────────────────────────────────── Task 11 (verification)
```

Tasks 2-3 can run in parallel after Task 1.
Tasks 4-7 can run in parallel after Tasks 2-3.
Tasks 8-10 can run in parallel after Tasks 4-7.
Task 11 runs last.
