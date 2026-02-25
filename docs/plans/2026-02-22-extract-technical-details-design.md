# Design: extract-technical-details Skill

**Date:** 2026-02-22
**Status:** Approved

## Problem

The `read-paper` skill generates high-level AI summaries (problem, method, results,
relevance). For papers central to a project, we also need a **reproduction-oriented
technical review** — step-by-step pipeline details with hyperparameters, architectures,
datasets, and training configurations.

## Decision

Create a new skill `extract-technical-details` that:

1. Downloads and reads the full arxiv PDF
2. Generates a structured technical review organized by pipeline stages
3. Creates a sub-page under the paper's existing Notion page
4. Adds a link inside the TL;DR callout on the parent page

## Design

### Skill Invocation

```
/extract-technical-details <arxiv_url_or_notion_page_url>
```

### Output Structure

- **Sub-page title:** "Technical Review"
- **Parent:** The paper's existing Notion References DB entry
- **Link placement:** Inside the TL;DR callout, appended as last line:
  `🔧 [Technical Review →](sub-page-link)`

### Template Sections

The technical review template is organized by pipeline stages:

1. **Pipeline Overview** — High-level flow diagram description
2. **Stage N: <Stage Name>** — Repeated per stage, each containing:
   - Goal/purpose of this stage
   - Input/output specifications
   - Architecture/algorithm details
   - Hyperparameters (in tables)
   - Training setup (environments, hardware, duration)
   - Dataset details (sources, sizes, splits)
3. **Key Hyperparameters Summary** — Single reference table
4. **Hardware & Evaluation** — Physical setup, test objects, metrics

### Content Zones

Same pattern as other skills:
- `## Technical Review` — AI-generated, replaceable on regeneration
- No `## My Notes` on the sub-page (notes live on the parent page)

### Relationship to read-paper

- `read-paper` creates the parent page with AI Summary + My Notes
- `extract-technical-details` creates a child page with deep technical details
- The parent page gets a link added to its TL;DR callout
- Both skills can be run independently; `extract-technical-details` requires
  the parent page to already exist

### Parent Page Modification

When adding the link to the TL;DR callout, use `replace_content_range` to
update only the callout block. The selection targets the TL;DR callout and
appends the link line before the closing `:::`.

## Files Changed

| File | Action |
|------|--------|
| `.claude/skills/extract-technical-details/SKILL.md` | Create |
| `docs/templates/technical-review-template.md` | Create |
