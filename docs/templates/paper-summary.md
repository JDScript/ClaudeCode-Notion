# Template: paper-summary.md

## Purpose

This template defines the Notion page body for a paper entry in the References database.
It is used by the **read-paper** skill when creating or updating a paper summary page.

## Variable Placeholders

Variables in `{braces}` are filled by Claude at generation time:

| Variable                | Description                                              |
|-------------------------|----------------------------------------------------------|
| `{one_liner}`           | A single sentence summarizing the paper's contribution   |
| `{problem_motivation}`  | What problem the paper addresses and why it matters      |
| `{method_approach}`     | The core technique or methodology proposed               |
| `{key_results}`         | Main quantitative or qualitative results                 |
| `{relevance_connections}` | How this paper connects to current research interests  |

## Regeneration Boundary Rules

- The **AI Summary** zone starts at the line `## AI Summary` and ends just before the line `## My Notes`.
- When Claude regenerates a summary, it replaces **everything between these two markers** and nothing else.
- The **My Notes** zone begins at `## My Notes` and extends to the end of the page.
- The My Notes section is **SACRED** — it is never touched, overwritten, or modified during regeneration.

---

<!-- TEMPLATE BODY START — content below is written to the Notion page -->

## AI Summary

::: callout {icon="📄" color="blue_bg"}
**One-liner:** {one_liner}
:::

### Problem & Motivation
{problem_motivation}

### Method / Approach
{method_approach}

### Key Results
{key_results}

### Relevance & Connections
{relevance_connections}

---

## My Notes

*Your personal notes here. This section is never overwritten by AI.*
