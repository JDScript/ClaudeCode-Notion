---
name: extract-technical-details
description: Extract reproduction-oriented technical details from an arxiv paper, create a structured Technical Review sub-page under the paper's Notion entry, and link it from the TL;DR callout.
---

# Skill: extract-technical-details

This skill reads the full arxiv PDF of a paper, extracts step-by-step implementation
details organized by pipeline stages, creates a "Technical Review" sub-page under the
paper's existing Notion References DB entry, and adds a link in the parent page's
TL;DR callout.

---

## Step 1: Identify the Paper

Accept one of:

- **An arxiv URL** — e.g. `https://arxiv.org/abs/2510.08556`
- **A Notion page URL** of an existing References DB entry

**If the user provides a Notion page URL:**

Use `notion-fetch` to load the page. Extract the arxiv URL from the `Arxiv` property.
Record both the arxiv URL and the Notion page ID.

**If the user provides an arxiv URL:**

Normalize to `https://arxiv.org/abs/XXXX.XXXXX`. Then search the workspace for the
matching References DB entry using `notion-search`. The parent page **must already exist**
— if not found, tell the user to run `/read-paper` first.

---

## Step 2: Fetch Full Paper Content

### 2a. Download the PDF

Download the full paper PDF:

```
curl -sL "https://arxiv.org/pdf/XXXX.XXXXX" -o /tmp/paper.pdf
```

### 2b. Read the PDF

Read the PDF in chunks using the Read tool with the `pages` parameter:

- Pages 1–10 (main paper)
- Pages 11–20+ (appendix and supplementary)

Read ALL pages. The appendix often contains the most critical reproduction details
(hyperparameters, architecture specifics, dataset details, training configurations).

### 2c. Optionally fetch project page

If the parent Notion page has a `Project Page` property, fetch it with `WebFetch`
to extract additional implementation details, code snippets, or supplementary tables.

### 2d. Extract structured information

For each pipeline stage identified in the paper, extract:

| Information          | Description                                                    |
|----------------------|----------------------------------------------------------------|
| Stage name & purpose | What this stage does in the overall pipeline                   |
| Inputs / outputs     | What data flows in and out                                     |
| Architecture         | Network architecture, algorithm, or procedure                  |
| Hyperparameters      | All numerical settings (learning rate, batch size, etc.)       |
| Training setup       | Hardware, framework, number of environments, training duration |
| Dataset details      | Object counts, data sources, splits, augmentation              |
| Key design choices   | Why this approach was chosen over alternatives                 |

---

## Step 3: Read Template and Generate Content

### 3a. Read the template file

Read the file at:

```
docs/templates/technical-review-template.md
```

Extract the content after the `<!-- TEMPLATE BODY START -->` marker.

### 3b. Identify pipeline stages

Analyze the paper to identify the major pipeline stages. Common patterns:

- RL policy training stages (oracle, distillation, fine-tuning)
- Data collection procedures (simulation, real-world)
- Model training stages (dynamics models, reward models)
- Deployment pipeline (inference, control)

### 3c. Generate the technical review

Fill in the template. For each stage, write a dedicated `### Stage N: <Name>` section.

**Writing guidelines:**

- **Be concrete.** Always include specific numbers, dimensions, and configurations.
- **Use tables** for hyperparameters, observation spaces, reward components.
- **Quote formulas** where they are essential for reproduction.
- **Note data sources** — exact dataset names, paper references, filtering criteria.
- **Include training scale** — number of environments, GPUs, wall-clock time.
- **Flag ambiguities** — if the paper is unclear on a detail, note it explicitly.

---

## Step 4: Create the Sub-Page in Notion

### 4a. Check for existing Technical Review sub-page

Use `notion-search` with query "Technical Review" scoped to the parent page ID
to check if a sub-page already exists.

**If it exists:** Use `notion-update-page` with `command: "replace_content"` to
update its content. (No My Notes zone on this page — safe to replace entirely.)

**If it does not exist:** Use `notion-create-pages` to create it:

```json
{
  "parent": {"page_id": "<parent_page_id>"},
  "pages": [{
    "properties": {"title": "Technical Review"},
    "content": "<generated technical review content>"
  }]
}
```

Record the new page's URL.

---

## Step 5: Add Link in Parent Page's TL;DR Callout

### 5a. Fetch the parent page

Use `notion-fetch` on the parent page to get current content.

### 5b. Update the TL;DR callout

Locate the TL;DR callout block. It looks like:

```
::: callout {icon="💡" color="blue_bg"}
**TL;DR:** <existing text>
:::
```

Use `notion-update-page` with `command: "replace_content_range"` to add the link
inside the callout, before the closing `:::`:

```json
{
  "page_id": "<parent_page_id>",
  "command": "replace_content_range",
  "selection_with_ellipsis": "::: callout {icon...closing :::",
  "new_str": "::: callout {icon=\"💡\" color=\"blue_bg\"}\n**TL;DR:** <existing text>\n\n🔧 [Technical Review →](<sub_page_url>)\n:::"
}
```

**If the callout already contains a Technical Review link**, update the URL rather
than adding a duplicate.

---

## Step 6: Report Results

After completing all steps, report:

```
Technical Review created: <paper_title>

Sub-page:    <sub_page_url>
Parent page: <parent_page_url>

Stages documented:
  1. <Stage 1 name>
  2. <Stage 2 name>
  ...

Link added to TL;DR callout ✓
```

---

## Reference: MCP Tool Names

| Tool                   | Usage in this skill                                        |
|------------------------|------------------------------------------------------------|
| `notion-fetch`         | Fetch parent page to get arxiv URL and current content     |
| `notion-search`        | Find the parent page or check for existing sub-page        |
| `notion-create-pages`  | Create the Technical Review sub-page                       |
| `notion-update-page`   | Update sub-page content and add link in parent callout     |
| `WebFetch`             | Optionally fetch project page for extra details            |
