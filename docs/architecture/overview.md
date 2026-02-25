# Architecture Overview

## System Overview

The Notion Research Management system follows a Skills-First architecture: all executable logic lives in `.claude/skills/` as Claude Code skill definitions, content templates live in `docs/templates/`, and database schema documentation lives in `docs/samples/`. There is no Python code, no build step, and no server process. The system is entirely driven by Claude Code invoking skills in response to user commands, which in turn call Notion MCP tools to read and write the Notion workspace. Modifying behavior means editing a skill definition or a template file — not changing code.

## Components

### `.claude/skills/` — Skills (7 total)

Each skill is a directory containing a `SKILL.md` file that describes a step-by-step procedure Claude Code executes when the skill is invoked.

| Skill | Directory | Purpose |
|---|---|---|
| scaffold-project | `.claude/skills/scaffold-project/` | Create a new research project page with standard structure |
| read-paper | `.claude/skills/read-paper/` | Fetch an arxiv paper, generate a summary, fill metadata |
| manage-tasks | `.claude/skills/manage-tasks/` | CRUD operations on a project's Tasks database |
| meeting-notes | `.claude/skills/meeting-notes/` | Create structured meeting notes pages |
| read-dataset | `.claude/skills/read-dataset/` | Read a dataset paper, generate summary, fill sensor metadata |
| extract-technical-details | `.claude/skills/extract-technical-details/` | Extract reproduction-oriented technical review |
| init | `.claude/skills/init/` | Discover workspace structure, populate config file |

### `docs/templates/` — Content Templates

Templates are Markdown files that define the page body written to Notion. Each template contains a `<!-- TEMPLATE BODY START -->` marker; only content below that marker is written to Notion. Variable placeholders in `{braces}` are substituted at generation time.

| Template | Used by skill |
|---|---|
| `paper-summary.md` | read-paper |
| `project-page.md` | scaffold-project |
| `meeting-notes.md` | meeting-notes |
| `technical-review-template.md` | extract-technical-details |

### `docs/samples/` — Database Schema Snapshots

These files document the Notion database schemas that skills create and query. They serve as both human reference and fallback instructions when MCP tools cannot create databases programmatically.

| File | Describes |
|---|---|
| `tasks-db-schema.md` | Tasks DB (Task/title, Status/status, Due/date, Week/multi_select, Assigned To/person) |
| `references-db-schema.md` | References DB (Name/title, Arxiv/url, Year/select, Conference/multi_select, Github/url, Project Page/url) |
| `datasets-db-schema.md` | Datasets DB (Name/title, Arxiv/url, Year/select, Scene Type/multi_select, and sensor/scale fields) |

### `CLAUDE.md` — Project Instructions

The `CLAUDE.md` file at the repository root provides Claude Code with project context: what the system is for, how skills relate to each other, and standing conventions (e.g., the content zone boundary rule). Claude Code reads this file automatically at the start of every session.

## Data Flow

```
User command
    │
    ▼
Claude Code (reads SKILL.md)
    │
    ├── reads docs/templates/<template>.md
    │
    ├── calls notion-fetch / notion-search  ──► Notion workspace (read)
    │
    └── calls notion-create-pages / notion-update-page ──► Notion workspace (write)
```

The user issues a natural language command or slash command. Claude Code matches it to a skill and executes the steps in that skill's `SKILL.md`. Skills read templates from `docs/templates/` to determine what content to write, then call Notion MCP tools to make changes in the workspace. No data is stored locally; the Notion workspace is the sole persistent store.

## Notion Workspace Structure

The top-level anchor is the workspace root page (page ID stored in `.claude/notion-config.json` as `workspace_root`, populated by the `init` skill). All research project pages are created as direct children of this page. Each project page follows the Functional Reorientation structure: a two-column header with a Project Overview callout and a Quick Links callout, an inline Tasks database, an inline References database, and a Meeting Notes sub-page. The `scaffold-project` skill creates this structure; the other skills navigate it by fetching the project page and extracting database URLs from the rendered content.

## Content Zone Convention

Every paper summary page is divided into two named zones:

- **AI Summary** (lines from `## AI Summary` through the `---` divider): regenerable. The `read-paper` skill may overwrite this zone freely.
- **My Notes** (lines from `## My Notes` to end of page): protected. No skill ever touches this zone.

When regenerating a summary, `read-paper` uses `notion-update-page` with `command: "replace_content_range"` and `selection_with_ellipsis: "## AI Summary...## My Notes"` to surgically replace only the AI zone. The skill always fetches the page first and checks whether `## My Notes` is present before deciding which write command to use. This convention is enforced by the skill logic, not by Notion itself.

## Template-Driven Generation

Output format is fully controlled by the template files in `docs/templates/`. To change the structure of a paper summary, edit `docs/templates/paper-summary.md`. To change the default project page layout, edit `docs/templates/project-page.md`. Skills read the template at runtime (not at skill-load time), so template changes take effect immediately on the next invocation without any reload or rebuild step.

## Per-Project Template Overrides

Skills that generate content from templates support per-project overrides stored in Notion. Each project can optionally have a "Templates" sub-page containing child pages named by template type (e.g., "Paper Summary Template", "Meeting Notes Template"). When present, the skill uses the Notion-hosted template instead of the local default in `docs/templates/`. The `init` skill discovers Templates sub-pages automatically and records their page IDs in the config file.

## Config-Driven ID Resolution

All workspace and project IDs are stored in `.claude/notion-config.json`, a gitignored file generated by the `init` skill. Skills read this file at startup instead of using hardcoded IDs. This makes the system portable across Notion accounts — each user runs `/init` once to populate their local config.
