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

### 2b. Extract figures from arxiv source

**This step is critical — always attempt figure extraction.** The teaser figure (Figure 1)
and method/pipeline figure are key visual elements for the summary.

Use a three-tier fallback strategy:

#### Tier 1: Arxiv HTML (preferred — gives publicly accessible URLs)

Fetch the HTML version:

```
https://arxiv.org/html/XXXX.XXXXXvN
```

Try `v1` first. Also use a prompt to extract introduction, method, and results text.

Extract all `<img>` tags. Arxiv HTML uses a naming convention:
- `S0.F1` or id containing `F1` → Figure 1 (usually the teaser)
- Figures in the method section (S3.F* or similar) → method/pipeline figure
- Image filenames are like `x1.png`, `x2.png`, etc.

Construct full URLs as:

```
https://arxiv.org/html/XXXX.XXXXXv1/x1.png   (teaser)
https://arxiv.org/html/XXXX.XXXXXv1/x3.png   (method, typically Figure 3)
```

Verify the URL returns HTTP 200 before using it.

#### Tier 2: Arxiv source tarball (when HTML version is unavailable)

Download and extract the arxiv source:

```bash
curl -sL "https://arxiv.org/e-print/XXXX.XXXXX" -o /tmp/paper_src.tar.gz
mkdir -p /tmp/paper_src && tar -xzf /tmp/paper_src.tar.gz -C /tmp/paper_src
```

Then identify figures:

1. **List image files:** `find /tmp/paper_src -name "*.png" -o -name "*.jpg" -o -name "*.pdf"`
2. **Read the .tex files** to find which images are the teaser and method figures:
   - Search for `\label{fig:teaser}` or `\label{fig:1}` — the `\includegraphics` in that
     figure environment is the teaser image filename.
   - Search for `\label{fig:method}` or `\label{fig:pipeline}` or `\label{fig:overview}` —
     that is the method figure.
3. **Read the figure as an image** using the Read tool if it is a PNG/JPG (the Read tool
   can display images). For PDF figures, note the filename for reference.

When using source-extracted figures, the images are local and cannot be directly embedded
in Notion via URL. In this case:
- Check if the **arxiv HTML version** exists and try to find the same figure there for a URL.
- Check the **project page** for hosted versions of the same figures.
- If no URL is available, **omit the image block** from the Notion page and note it in the
  report. Do NOT use a local file path as an image URL.

#### Tier 3: Project page (last resort)

If the paper has a project page, use `WebFetch` to find teaser/method images hosted there.
Project pages often have high-resolution figures with stable URLs.

### 2c. Consolidate extracted metadata

After fetching, record the following fields for later use:

| Field              | Source                                              |
|--------------------|-----------------------------------------------------|
| `title`            | Abstract page title                                 |
| `authors`          | Author list from abstract page                      |
| `year`             | Year from the submission/publication date           |
| `conference`       | Venue string if found, otherwise leave empty        |
| `github_url`       | GitHub link if found in abstract or HTML, else empty|
| `project_page`     | Official project page URL if found, else empty      |
| `abstract_text`    | The full abstract                                   |
| `method_notes`     | Key method details from HTML full text (if fetched) |
| `results_notes`    | Key quantitative results from HTML full text        |
| `teaser_image_url` | URL to Figure 1 / teaser (from Step 2b)            |
| `teaser_caption`   | Caption text for the teaser figure                  |
| `method_image_url` | URL to method/pipeline figure (from Step 2b)        |
| `method_caption`   | Caption text for the method figure                  |

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

The extracted template body includes image placeholders for the teaser and method figures.
Use the URLs extracted in Step 2b. If a figure URL is not available, omit that image line
entirely (do not leave a broken `![]()` in the output).

### 3b. Generate the summary

Fill in all `{placeholder}` variables using the content gathered in Step 2:

| Placeholder               | How to fill it                                                                                       |
|---------------------------|------------------------------------------------------------------------------------------------------|
| `{one_liner}`             | A single sentence (≤25 words) stating the paper's core contribution                                 |
| `{problem_motivation}`    | 2–4 sentences: what problem is being solved, why it is hard, and why it matters                      |
| `{method_approach}`       | 3–6 sentences: the core technique, architecture, training setup, or algorithm proposed               |
| `{key_results}`           | Bullet list of the most important quantitative results, benchmarks beaten, or qualitative findings   |

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

Content written: AI Summary (Problem & Motivation, Method, Key Results)

Properties updated:
  Year:         <value or "not set — could not determine">
  Conference:   <value or "not set — preprint or venue unclear">
  Github:       <url or "not found">
  Project Page: <url or "not found">

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
