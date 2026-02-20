# Notion Research Project Management — Design Document

**Date**: 2026-02-20
**Status**: Approved

## Overview

A Claude Code project that manages research projects in Notion via MCP + `.claude` skills. The system is scoped to the **Research Space** workspace, covering task management, paper reading, meeting notes, and project scaffolding.

The architecture is **Skills-First**: all functionality lives in `.claude/skills/` with supporting templates in `docs/templates/`. No Python code needed — pure Claude Code skills + Notion MCP.

## Notion Workspace Structure (Current)

```
Research Space (page: 2306f2eb-947e-805e-9064-ed452a446342)
├── CRAG (page)
│   ├── Experiments DB (database)
│   └── Update pages
├── Functional Reorientation (page) ← reference template
│   ├── Project Overview callout
│   ├── Quick Links callout (GitHub, Zoom, Meeting Notes)
│   ├── Tasks DB (Status, Due, Assigned To, Week)
│   ├── References DB (Name, Arxiv, Conference, Year, Github, Project Page)
│   └── Meeting Notes (sub-page with child pages)
├── Navigation Reasoning Benchmark (page)
│   ├── Project Overview callout (placeholder)
│   ├── Quick Links callout (Meeting Notes)
│   └── References DB (Name, Arixv, Project Page) ← simpler schema
└── Guide @ AI4CE Planck (page)
```

**Reference project**: Functional Reorientation is the most complete project and serves as the template for new projects.

## Project Structure

```
notion/
├── .claude/
│   └── skills/
│       ├── read-paper.md          # Arxiv paper reading & summary
│       ├── manage-tasks.md        # Task CRUD & queries
│       ├── meeting-notes.md       # Meeting notes creation
│       └── scaffold-project.md    # New project scaffolding
├── docs/
│   ├── architecture/
│   │   └── overview.md            # System architecture
│   ├── templates/
│   │   ├── paper-summary.md       # Paper page content template
│   │   ├── project-page.md        # Project page structure template
│   │   └── meeting-notes.md       # Meeting notes template
│   ├── skills/
│   │   └── catalog.md             # Skills index & documentation
│   ├── plans/                     # Design documents
│   └── samples/
│       ├── references-db-schema.md
│       └── tasks-db-schema.md
├── CLAUDE.md
├── README.md
├── pyproject.toml
└── .gitignore
```

## Skills Design

### Skill 1: `read-paper`

**Purpose**: Auto-read an arxiv paper and generate a structured summary in Notion.

**Trigger**: User provides an arxiv link, or references a paper entry in a References DB.

**Flow**:
1. Fetch paper content from arxiv URL (abstract, sections)
2. Read `docs/templates/paper-summary.md` for the content template
3. Generate structured summary and write to the Notion page content
4. Auto-extract & fill DB properties: Conference, Year, Author (add to schema if missing)
5. Identify connections to other papers in the same References DB

**Content zone convention**:
- `## AI Summary` section — Claude-generated, safe to regenerate
- `## My Notes` section — User-written, NEVER overwritten on regenerate

**Regenerate logic**: Use Notion MCP `replace_content_range` targeting only the `## AI Summary` section. Everything below `## My Notes` is preserved.

### Skill 2: `manage-tasks`

**Purpose**: Create, update, and query tasks in a project's Tasks DB.

**Trigger**: Natural language, e.g., "add a task to Nav Benchmark: implement baseline"

**Capabilities**:
- Create task (default Status = "Not started")
- Update task status, due date, assignee
- Query tasks by project, status, or week
- Locate the correct Tasks DB by searching within the specified project

### Skill 3: `meeting-notes`

**Purpose**: Create a structured meeting notes page under a project.

**Trigger**: e.g., "create meeting notes for Nav Benchmark"

**Flow**:
1. Read `docs/templates/meeting-notes.md`
2. Find or create the Meeting Notes sub-page under the target project
3. Create a new page with title `Meeting - {YYYY-MM-DD}` (or custom title)
4. Pre-fill template structure (Attendees, Agenda, Discussion, Action Items)

### Skill 4: `scaffold-project`

**Purpose**: Create a new research project with the standard structure.

**Trigger**: e.g., "create a new project: Robot Grasping"

**Flow** (modeled after Functional Reorientation):
1. Create project page under Research Space with icon
2. Add two-column layout:
   - Left: Project Overview callout (placeholder)
   - Right: Quick Links callout with Meeting Notes link
3. Create inline Tasks DB with standard schema (Task, Status, Due, Assigned To, Week)
4. Create inline References DB with standard schema (Name, Arxiv, Conference, Year, Github, Project Page)
5. Create Meeting Notes sub-page

## Key Conventions

1. **Template-driven generation**: All Notion content follows templates in `docs/templates/`. Modify templates to change output format.
2. **User notes protection**: AI-generated and user-written content are separated by heading markers. Regeneration only touches AI sections.
3. **Schema snapshots**: `docs/samples/` records real DB schemas for skill reference.
4. **Functional Reorientation as reference**: New projects mirror its structure. The References DB schema from Functional Reorientation (with Arxiv, Conference, Year, Github, Project Page) is the standard.

## Future Considerations

- Python automation layer for batch operations if needed
- Notion webhook integration for automatic paper processing on DB entry
- Cross-project paper deduplication
- Export/backup utilities
