# Notion Research Management

A Skills-First system for managing research projects in Notion via Claude Code skills and the Notion MCP.

## Prerequisites

- [Claude Code](https://claude.ai/code) with the Notion MCP connected and authorized to the Research Space workspace.

## Quick Start

All functionality is exposed as Claude Code slash commands. Open a Claude Code session in this repository and invoke the skill you need:

```
/scaffold-project
/read-paper
/manage-tasks
/meeting-notes
```

Claude Code will load the corresponding skill and guide you through the interaction. No Python environment or local execution is required.

## Skills

### `/scaffold-project`
Creates a new research project page in the Research Space workspace, following the standard two-column layout with a Tasks DB, References DB, and a Meeting Notes sub-page.

**Example:**
```
/scaffold-project
> Project name: Neural Topology Mapping
> (Claude creates the full project structure in Notion)
```

### `/read-paper`
Fetches an arxiv paper, generates a structured summary, and optionally creates or updates a page in a project's References DB.

**Example:**
```
/read-paper
> Arxiv URL or ID: 2401.12345
> Project: Neural Topology Mapping
> (Claude writes an AI Summary to the paper page; your ## My Notes section is never touched)
```

### `/manage-tasks`
Create, update, list, and close tasks in a project's inline Tasks DB. Supports filtering by status, assignee, and week.

**Example:**
```
/manage-tasks
> Action: add
> Project: Neural Topology Mapping
> Task: Implement baseline model, Due: 2026-03-01, Week: 5
```

### `/meeting-notes`
Creates a dated Meeting Notes entry as a sub-page of a project, pre-populated from the meeting notes template.

**Example:**
```
/meeting-notes
> Project: Neural Topology Mapping
> Date: 2026-02-20
> (Claude creates a new meeting notes sub-page)
```

## Documentation

### Architecture
- `docs/architecture/overview.md` — System architecture and design rationale

### Skills Catalog
- `docs/skills/catalog.md` — Full skill specifications and parameter reference

### Templates
- `docs/templates/project-page.md` — Project page structure template
- `docs/templates/paper-summary.md` — Paper summary page template
- `docs/templates/meeting-notes.md` — Meeting notes page template

### DB Schemas
- `docs/samples/tasks-db-schema.md` — Tasks database schema (fields, types, options)
- `docs/samples/references-db-schema.md` — References database schema

### Design Plans
- `docs/plans/2026-02-20-notion-research-management-design.md` — Initial design document
- `docs/plans/2026-02-20-notion-research-management-impl.md` — Implementation plan
