---
name: read-paper
description: Auto-read an arxiv paper, generate a structured summary in Notion, and extract metadata (Conference, Year, Authors). Preserves user notes on regeneration.
---

# Skill: read-paper

This skill reads an arxiv paper, generates a structured AI summary using the standard template, fills in metadata properties (Year, Conference, Github, Project Page), and writes everything to the appropriate Notion References DB entry. On regeneration it replaces only the AI Summary section and never touches the user's notes.

---

## Step 1: Identify the Paper

Accept one of two forms of input from the user:

- **An arxiv URL** — e.g. `https://arxiv.org/abs/2312.04567`
- **A Notion page URL** of an existing References DB entry — e.g. `https://www.notion.so/Some-Paper-Title-<id>`

**If the user provides a Notion page URL:**

Use `notion-fetch` on that URL to load the page. Extract the arxiv URL from the page properties (look for a property named `Arxiv` or `Arixv` — note the possible typo in some databases). Record both the arxiv URL and the Notion page ID/URL as the target.

**If the user provides an arxiv URL directly:**

Normalize it to the canonical abstract form: `https://arxiv.org/abs/XXXX.XXXXX`. Proceed to Step 2; the target page will be resolved in Step 4.

---

## Step 2: Fetch Paper Content

### 2a. Fetch the abstract page

Use `WebFetch` on the canonical arxiv abstract URL (e.g. `https://arxiv.org/abs/2312.04567`) with a prompt asking to extract:

- Full paper title
- Author list (all authors, in order)
- Abstract text
- Submission year (from the submission date shown on the page)
- Conference or venue if mentioned (e.g. "Submitted to NeurIPS 2024", "Published at CVPR 2025")
- Any GitHub repository link mentioned in the abstract or comments
- Any official project page link mentioned

### 2b. Optionally fetch the HTML full text

If the abstract alone is insufficient to write a detailed Method or Results section, use `WebFetch` on the HTML version of the paper:

```
https://arxiv.org/html/XXXX.XXXXXvN
```

Try version `v1` first. Use a prompt asking to extract: the introduction (for problem/motivation), the method/approach section, and the main results/experiments section (for key results and numbers).

### 2c. Consolidate extracted metadata

After fetching, record the following fields for later use:

| Field            | Source                                              |
|------------------|-----------------------------------------------------|
| `title`          | Abstract page title                                 |
| `authors`        | Author list from abstract page                      |
| `year`           | Year from the submission/publication date           |
| `conference`     | Venue string if found, otherwise leave empty        |
| `github_url`     | GitHub link if found in abstract or HTML, else empty|
| `project_page`   | Official project page URL if found, else empty      |
| `abstract_text`  | The full abstract                                   |
| `method_notes`   | Key method details from HTML full text (if fetched) |
| `results_notes`  | Key quantitative results from HTML full text        |

---

## Step 3: Read Template and Generate Summary

### 3a. Read the template file

Read the file at:

```
docs/templates/paper-summary.md
```

Extract only the content that appears **after** the line:

```
<!-- TEMPLATE BODY START — content below is written to the Notion page -->
```

The extracted template body is:

```
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
```

### 3b. Search for related papers (for Relevance & Connections)

Use `notion-search` to find other paper entries in the same project's References DB. Look for papers with overlapping topics, methods, or benchmarks. Note 2–4 connections by title and a brief description of how they relate to the current paper. If no clear connections are found, note what broader research area this paper fits into.

### 3c. Generate the summary

Fill in all `{placeholder}` variables using the content gathered in Steps 2 and 3b:

| Placeholder               | How to fill it                                                                                       |
|---------------------------|------------------------------------------------------------------------------------------------------|
| `{one_liner}`             | A single sentence (≤25 words) stating the paper's core contribution                                 |
| `{problem_motivation}`    | 2–4 sentences: what problem is being solved, why it is hard, and why it matters                      |
| `{method_approach}`       | 3–6 sentences: the core technique, architecture, training setup, or algorithm proposed               |
| `{key_results}`           | Bullet list of the most important quantitative results, benchmarks beaten, or qualitative findings   |
| `{relevance_connections}` | 2–4 bullet points connecting this paper to related entries found in Step 3b, or to the broader field |

Write in clear, precise academic prose. Do not copy the abstract verbatim. Synthesize from all sources fetched.

---

## Step 4: Identify the Target Notion Page

### Case A — User provided a Notion page URL (already resolved in Step 1)

The target page is already known. Skip to Step 5.

### Case B — User provided only an arxiv URL

Search the project's References DB for an existing entry whose `Arxiv` (or `Arixv`) property matches the arxiv URL.

**If a matching entry is found:**

Use that page as the target. Record its Notion page URL and ID.

**If no matching entry is found:**

Ask the user which project's References DB to add the paper to. Once confirmed, create a new entry using `notion-create-pages`:

```json
{
  "parent_type": "database_id",
  "parent_id": "<references_db_id>",
  "title": "<title from Step 2>",
  "properties": {
    "Arxiv": "<arxiv_url>"
  }
}
```

Record the URL and ID of the newly created page.

---

## Step 5: Write Content to the Notion Page

### REGENERATION SAFETY — Read this before writing anything

**ALWAYS fetch the target page first** using `notion-fetch` before writing any content. Inspect the existing page body for the presence of a `## My Notes` section.

- **If `## My Notes` is absent** (new or empty page): Use `notion-update-page` with `command: "replace_content"` to write the full template body (AI Summary + empty My Notes section).
- **If `## My Notes` is present** (regenerate case): Use `notion-update-page` with `command: "replace_content_range"` to replace ONLY the AI Summary zone. See the Regeneration Safety section below for exact parameters.

**NEVER use `replace_content` when `## My Notes` already exists.** Doing so would destroy the user's personal notes.

### 5a. New or empty page

Use `notion-update-page` with:

```json
{
  "page_id": "<target_page_id>",
  "command": "replace_content",
  "content": "<full substituted template body from Step 3c>"
}
```

The content written is the entire template body: the `## AI Summary` section followed by `---` and the `## My Notes` section with its placeholder text.

### 5b. Regenerate case (## My Notes already exists)

Use `notion-update-page` with:

```json
{
  "page_id": "<target_page_id>",
  "command": "replace_content_range",
  "selection_with_ellipsis": "## AI Summary...## My Notes",
  "new_str": "<everything from '## AI Summary' up to but NOT including '---\n\n## My Notes'>"
}
```

The `new_str` value must end with the `---` divider line and a trailing newline, but must not include `## My Notes` or anything after it. The existing `## My Notes` content is preserved exactly as-is.

---

## Step 6: Update DB Properties

Use `notion-update-page` with `command: "update_properties"` to fill in the metadata fields:

```json
{
  "page_id": "<target_page_id>",
  "command": "update_properties",
  "properties": {
    "Year": "<year_string>",
    "Conference": ["<conference_string>"],
    "Github": "<github_url_or_omit>",
    "Project Page": "<project_page_url_or_omit>"
  }
}
```

Rules:

- **Year**: Set as a `select` value (string, e.g. `"2024"`). Only set if the year was unambiguously extracted.
- **Conference**: Set as a `multi_select` value (array of strings). Use the canonical short name if recognizable (e.g. `"CVPR"`, `"NeurIPS"`, `"ICLR"`, `"ICRA"`, `"CoRL"`, `"Science Robotics"`). If the paper is an arXiv preprint with no confirmed venue, set to `["Preprint"]`. Always set this field — never leave it empty.
- **Github**: Set as a `url` value. Omit the property if no GitHub link was found.
- **Project Page**: Set as a `url` value. Omit the property if no project page was found.

Only include properties in the update payload that have values to set. Do not send null or empty strings for missing fields.

---

## Regeneration Safety

This section is the authoritative reference for how to handle the `## My Notes` boundary.

### The two zones

```
## AI Summary          ← REPLACEABLE zone starts here
...
---                    ← REPLACEABLE zone ends here (this divider is part of the AI Summary zone)

## My Notes            ← SACRED zone starts here
...                    ← SACRED zone extends to end of page
```

### Decision tree

```
Fetch target page
        │
        ▼
Does page body contain "## My Notes"?
        │
   No ──┴── Yes
   │              │
   ▼              ▼
replace_content   replace_content_range
(full template)   selection: "## AI Summary...## My Notes"
                  new_str:   AI Summary block + "---\n\n" (no My Notes line)
```

### replace_content_range boundaries

- `selection_with_ellipsis`: `"## AI Summary...## My Notes"`
  - This selects from the start of `## AI Summary` up to (but not including) `## My Notes`.
- `new_str`: The newly generated AI Summary content, ending with `---` followed by two newlines.
  - Must NOT contain `## My Notes` or any text that follows it.
  - The existing `## My Notes` heading and all user content beneath it remain untouched.

### Why this matters

User notes are irreplaceable. A single incorrect `replace_content` call on a page that has notes would permanently destroy them. Always check first.

---

## Step 7: Report Results

After completing all steps, report a summary to the user:

```
Paper processed: <title>
Notion page:     <page_url>

Content written: AI Summary (Problem & Motivation, Method, Key Results, Relevance & Connections)

Properties updated:
  Year:         <value or "not set — could not determine">
  Conference:   <value or "not set — preprint or venue unclear">
  Github:       <url or "not found">
  Project Page: <url or "not found">

Related papers found in References DB:
  - <title 1>
  - <title 2>
  (or "None found")

Notes:
  <Any caveats, e.g. "HTML full text was not available; summary based on abstract only.">
  <e.g. "The 'Arxiv' property in this DB is spelled 'Arixv' — used that name for the update.">
```

Provide the clickable Notion page URL so the user can open it directly.

---

## Reference: MCP Tool Names

| Tool                   | Usage in this skill                                                     |
|------------------------|-------------------------------------------------------------------------|
| `notion-fetch`         | Fetch a Notion page to extract arxiv URL or check for existing content  |
| `notion-search`        | Search the References DB for related papers (Step 3b) or existing entry |
| `notion-create-pages`  | Create a new References DB entry when no existing entry is found        |
| `notion-update-page`   | Write page content and update DB properties                             |
| `WebFetch`             | Fetch arxiv abstract page and optionally the HTML full text             |
