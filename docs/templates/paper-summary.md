# Template: paper-summary.md

## Purpose

This template defines the Notion page body for a paper entry in the References database.
It is used by the **read-paper** skill when creating or updating a paper summary page.

## Writing Style Guidelines

- **Write for a busy researcher**, not a reviewer. Lead with what matters most.
- **Use plain language.** Avoid jargon-heavy summaries. A PhD student outside the subfield
  should be able to follow the gist.
- **Mix prose and bullets.** Use prose for narrative sections (Problem, Method) and bullets
  only for enumerating distinct results or connections. Not everything should be a bullet.
- **Show importance.** Use **bold** for the single most important takeaway in each section.
  Use a callout for the one-liner. Let the reader skim effectively.
- **Include key figures.** Embed 1-2 of the paper's most important figures (teaser, method
  diagram, or key result). Use the project page or arxiv HTML as image source. Caption each
  figure clearly.

## Variable Placeholders

Variables in `{braces}` are filled by Claude at generation time:

| Variable                  | Description                                                |
|---------------------------|------------------------------------------------------------|
| `{one_liner}`             | A single sentence summarizing the paper's contribution     |
| `{teaser_image_url}`      | URL to the paper's teaser or overview figure               |
| `{teaser_caption}`        | Caption for the teaser figure                              |
| `{problem_motivation}`    | What problem the paper addresses and why it matters (prose)|
| `{method_image_url}`      | URL to the method/pipeline diagram (optional)              |
| `{method_caption}`        | Caption for the method figure (optional)                   |
| `{method_approach}`       | The core technique or methodology proposed (prose)         |
| `{key_results}`           | Main results — mix prose for context, bullets for numbers  |
| `{relevance_connections}` | How this paper connects to the project (prose + bullets)   |

## Image Sourcing

Try to find figures from (in order of preference):
1. The paper's official project page (e.g., `https://project.github.io/figs/teaser.png`)
2. The arxiv HTML version (e.g., `https://arxiv.org/html/XXXX.XXXXXv1/extracted/.../figure.png`)
3. If no figure URLs are reachable, omit the image blocks.

## Regeneration Boundary Rules

- The **AI Summary** zone starts at the line `## AI Summary` and ends just before the line `## My Notes`.
- When Claude regenerates a summary, it replaces **everything between these two markers** and nothing else.
- The **My Notes** zone begins at `## My Notes` and extends to the end of the page.
- The My Notes section is **SACRED** — it is never touched, overwritten, or modified during regeneration.

---

<!-- TEMPLATE BODY START — content below is written to the Notion page -->

## AI Summary

::: callout {icon="💡" color="blue_bg"}
**TL;DR:** {one_liner}
:::

![{teaser_caption}]({teaser_image_url})

### Problem & Motivation
{problem_motivation}

### Method / Approach
![{method_caption}]({method_image_url})

{method_approach}

### Key Results
{key_results}

### Relevance to This Project
{relevance_connections}

---

## My Notes

*Your personal notes here. This section is never overwritten by AI.*
