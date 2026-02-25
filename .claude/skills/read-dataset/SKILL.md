---
name: read-dataset
description: Read a dataset paper from arxiv, generate a structured summary in Notion, and fill in dataset metadata properties. Preserves user notes on regeneration.
---

# Skill: read-dataset

This skill reads a dataset paper from arxiv, generates a structured AI summary, fills in metadata properties (Year, License, Scene Type, Embodiment, Camera, LiDAR, IMU, GPS, Episodes, Video Length, Trajectory Length, Scene Size), and writes everything to the Datasets DB entry in Notion. On regeneration it replaces only the AI Summary section and never touches the user's notes.

---

## Step 1: Identify the Dataset

Accept one of two forms of input from the user:

- **An arxiv URL** — e.g. `https://arxiv.org/abs/2309.13549`
- **A Notion page URL** of an existing Datasets DB entry — e.g. `https://www.notion.so/Some-Dataset-<id>`

**If the user provides a Notion page URL:**

Use `notion-fetch` on that URL to load the page. Extract the arxiv URL from the `Arxiv` property. Record both the arxiv URL and the Notion page ID/URL as the target.

**If the user provides an arxiv URL directly:**

Normalize it to the canonical abstract form: `https://arxiv.org/abs/XXXX.XXXXX`. Proceed to Step 2; the target page will be resolved in Step 4.

---

## Step 2: Fetch Dataset Paper Content

### 2a. Fetch the abstract page

Use `WebFetch` on the canonical arxiv abstract URL with a prompt asking to extract:

- Full paper title (dataset name)
- Author list
- Abstract text
- Submission year
- Any GitHub repository link
- **Any official project/dataset page link** (CRITICAL — always extract this)
- License information (e.g. CC BY 4.0, Apache 2.0)
- Conference/venue if mentioned (e.g. ICRA 2025, CoRL 2024)

### 2b. Optionally fetch the HTML full text

If the abstract alone is insufficient to extract dataset details (sensor specs, scale metrics), use `WebFetch` on the HTML version:

```
https://arxiv.org/html/XXXX.XXXXXvN
```

Try version `v1` first. Use a prompt asking to extract:
- Scene types covered (Indoor, Outdoor, Urban, Campus, Off-road, Mix, etc.)
- Robot/embodiment platform
- Camera configuration (number, type: RGB, RGB-D, Stereo, 360)
- LiDAR configuration (number, type: 2D/3D, model)
- IMU availability (yes/no)
- GPS availability (yes/no)
- Number of episodes/trajectories
- Total video length (hours)
- Total trajectory length (km)
- Scene coverage area (km²)
- License

### 2c. Consolidate extracted metadata

After fetching, record:

| Field               | Source                                           |
|---------------------|--------------------------------------------------|
| `name`              | Dataset name from paper title                    |
| `year`              | Year from submission/publication date            |
| `arxiv_url`         | Canonical arxiv URL                              |
| `github_url`        | GitHub link if found                             |
| `project_page`      | Official dataset page URL if found               |
| `license`           | License string                                   |
| `scene_type`        | Scene type(s) — multi_select values              |
| `embodiment`        | Robot platform or data collection method         |
| `camera`            | Camera configuration string                      |
| `lidar`             | LiDAR configuration string                       |
| `imu`               | Boolean — whether IMU data is included           |
| `gps`               | Boolean — whether GPS data is included           |
| `episodes`          | Number of episodes/trajectories                  |
| `video_length_h`    | Total video/recording length in hours            |
| `traj_length_km`    | Total trajectory length in km                    |
| `scene_size_km2`    | Scene coverage area in km²                       |
| `abstract_text`     | The full abstract                                |

---

## Step 3: Read Template and Generate Summary

### 3a. Template structure

The summary follows this template:

```
## AI Summary

::: callout {icon="💡" color="blue_bg"}
**TL;DR:** {one_liner}
:::

### Overview
{dataset_overview}

### Data Collection
{data_collection}

### Sensor Configuration
{sensor_config}

### Scale & Coverage
{scale_coverage}

### Relevance to This Project
{relevance_connections}

### Discrepancies from Original Table
{discrepancies}

---

## My Notes

*Your personal notes here. This section is never overwritten by AI.*
```

### 3b. Search for related datasets

Use `notion-search` to find other entries in the same project's Datasets DB. Look for datasets with overlapping scene types, sensor configurations, or embodiment platforms. Note 2–4 connections.

### 3c. Generate the summary

Fill in all placeholders:

| Placeholder             | How to fill it                                                                     |
|-------------------------|------------------------------------------------------------------------------------|
| `{one_liner}`           | Single sentence (≤25 words) describing the dataset's core contribution             |
| `{dataset_overview}`    | 2–4 sentences: what the dataset covers, its purpose, and what makes it unique      |
| `{data_collection}`     | 2–4 sentences: how data was collected, platform used, environments covered         |
| `{sensor_config}`       | Bullet list of all sensors with specifications                                     |
| `{scale_coverage}`      | Bullet list of scale metrics (episodes, hours, km, area) with context              |
| `{relevance_connections}` | 2–4 bullets connecting this dataset to related entries or the broader project    |

Write in clear, precise prose. Synthesize from all sources fetched.

---

## Step 4: Identify the Target Notion Page

### Case A — User provided a Notion page URL (already resolved in Step 1)

The target page is already known. Skip to Step 5.

### Case B — User provided only an arxiv URL

Search the Datasets DB (collection ID: `30d6f2eb-947e-80d8-82f8-000b470f76d4`) for an existing entry whose `Arxiv` property matches the arxiv URL.

**If a matching entry is found:** Use that page as the target.

**If no matching entry is found:** Create a new entry using `notion-create-pages`:

```json
{
  "parent": {"data_source_id": "30d6f2eb-947e-80d8-82f8-000b470f76d4"},
  "pages": [{
    "properties": {
      "Name": "<dataset_name>",
      "Arxiv": "<arxiv_url>"
    }
  }]
}
```

---

## Step 5: Write Content to the Notion Page

### REGENERATION SAFETY — Read this before writing anything

**ALWAYS fetch the target page first** using `notion-fetch` before writing any content. Inspect the existing page body for the presence of a `## My Notes` section.

- **If `## My Notes` is absent** (new or empty page): Use `notion-update-page` with `command: "replace_content"` to write the full template body (AI Summary + empty My Notes section).
- **If `## My Notes` is present** (regenerate case): Use `notion-update-page` with `command: "replace_content_range"` to replace ONLY the AI Summary zone.

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

---

## Step 6: Update DB Properties

Use `notion-update-page` with `command: "update_properties"` to fill in the metadata fields:

```json
{
  "page_id": "<target_page_id>",
  "command": "update_properties",
  "properties": {
    "Year": "<year_string>",
    "License": "<license_string>",
    "Scene Type": "<scene_type_value>",
    "Embodiment": "<embodiment_string>",
    "Camera": "<camera_config_string>",
    "LiDAR": "<lidar_config_string>",
    "IMU": "__YES__or__NO__",
    "GPS": "__YES__or__NO__",
    "Episodes": <number_or_omit>,
    "Video Length (h)": <number_or_omit>,
    "Trajectory Length (km)": <number_or_omit>,
    "Scene Size (km²)": <number_or_omit>,
    "Arxiv": "<arxiv_url>",
    "Project Page": "<project_page_url_or_omit>"
  }
}
```

Rules:

- **Year**: Select value (e.g. `"2024"`). Only set if unambiguously extracted.
- **License**: Select value. Common values: `CC BY 4.0`, `CC BY-SA 4.0`, `CC BY-NC-SA 4.0`, `CC0 1.0`, `Apache 2.0`, `MIT`. If unknown, omit.
- **Scene Type**: Multi_select. Options: `Indoor`, `Outdoor`, `Urban`, `Campus`, `Off-road`, `Mix`. Use the closest match.
- **Embodiment**: Rich text. The robot platform or collection method (e.g. `Husky`, `Human-ego`, `Wheeled`, `Head-mount`).
- **Camera / LiDAR**: Rich text. Concise config string (e.g. `2 RGB (Stereo)`, `1 (Ouster OS1-128)`).
- **IMU / GPS**: Checkbox. Use `"__YES__"` or `"__NO__"`.
- **Numeric fields** (Episodes, Video Length, Trajectory Length, Scene Size): Number type. Omit if unknown.

Only include properties that have values. Do not send null or empty strings.

---

## Step 7: Report Results

After completing all steps, report:

```
Dataset processed: <name>
Notion page:       <page_url>

Content written: AI Summary (Overview, Data Collection, Sensor Config, Scale, Relevance)

Properties updated:
  Year:             <value>
  License:          <value or "not found">
  Scene Type:       <value>
  Embodiment:       <value or "not found">
  Camera:           <value>
  LiDAR:            <value or "none">
  IMU:              <yes/no>
  GPS:              <yes/no>
  Episodes:         <value or "not found">
  Video Length:     <value or "not found">
  Traj Length:      <value or "not found">
  Scene Size:       <value or "not found">

Related datasets found:
  - <name 1>
  - <name 2>
  (or "None found")

Notes:
  <Any caveats>
```

---

## Regeneration Safety

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

---

## Reference: MCP Tool Names

| Tool                   | Usage in this skill                                               |
|------------------------|-------------------------------------------------------------------|
| `notion-fetch`         | Fetch a Notion page to check for existing content                 |
| `notion-search`        | Search the Datasets DB for related entries or existing entry      |
| `notion-create-pages`  | Create a new Datasets DB entry when no existing entry is found    |
| `notion-update-page`   | Write page content and update DB properties                       |
| `WebFetch`             | Fetch arxiv abstract page and optionally the HTML full text       |
