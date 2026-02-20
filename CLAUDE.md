# CLAUDE.md

## Project Overview

Notion Research Project Management — a Skills-First system for managing
research projects in Notion via Claude Code skills + Notion MCP.

Scoped to the Research Space workspace. No Python code — all functionality
lives in `.claude/skills/`.

## Notion Workspace

Research Space page ID: `2306f2eb-947e-805e-9064-ed452a446342`

Projects follow a standard structure (modeled after Functional Reorientation):
- Project Overview callout + Quick Links callout (two-column layout)
- Inline Tasks DB (schema: Task, Status, Due, Assigned To, Week)
- Inline References DB (schema: Name, Arxiv, Conference, Year, Github, Project Page)
- Meeting Notes sub-page

## Skills

| Skill | Purpose | Invocation |
|-------|---------|------------|
| `scaffold-project` | Create new research project | `/scaffold-project` |
| `read-paper` | Read arxiv paper, generate summary | `/read-paper` |
| `manage-tasks` | CRUD on project Tasks DB | `/manage-tasks` |
| `meeting-notes` | Create meeting notes | `/meeting-notes` |

## Key Conventions

1. **Content zones**: AI-generated content under `## AI Summary`, user notes under `## My Notes`. Regeneration NEVER touches `## My Notes`.
2. **Template-driven**: All Notion content follows templates in `docs/templates/`.
3. **Schema reference**: DB schemas documented in `docs/samples/`.
4. **Functional Reorientation as reference**: New projects mirror its structure in Notion.

## File Structure

- `.claude/skills/` — Claude Code skill definitions (SKILL.md per skill)
- `docs/templates/` — Notion page content templates
- `docs/samples/` — DB schema snapshots from actual Notion workspace
- `docs/architecture/` — System architecture documentation
- `docs/skills/` — Skills catalog and documentation
- `docs/plans/` — Design and implementation plans
