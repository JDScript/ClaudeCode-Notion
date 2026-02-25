# References Database Schema

This document describes the standard References database schema used across research projects in Notion. All projects now use the same schema.

---

## Standard Schema

Used by all research projects. Collection IDs are stored in `.claude/notion-config.json` under each project's `references_db` field (populated by the `init` skill).

```sql
CREATE TABLE references (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arxiv" TEXT,           -- type: url
    "Year" TEXT,            -- type: select, options: ["2023", "2024", "2025", "2026"]
    "Project Page" TEXT,    -- type: url
    "Github" TEXT,          -- type: url
    "Conference" TEXT,      -- type: multi_select, options: ["Science Robotics", "ICLR", "ICCV", "NeurIPS", "CVPR", "ICRA", "CoRL", "Preprint", "ECCV"]
    "Name" TEXT             -- type: title
)
```

### Property Details

| Property       | Notion Type    | Notes                                                              |
|----------------|----------------|--------------------------------------------------------------------|
| `url`          | (internal)     | Notion page URL, used as unique row identifier                     |
| `createdTime`  | (internal)     | ISO-8601 timestamp of page creation                                |
| `Arxiv`        | url            | Link to the paper on arxiv.org                                     |
| `Year`         | select         | Publication year; options: `2023`, `2024`, `2025`, `2026`          |
| `Project Page` | url            | Link to the official project website                               |
| `Github`       | url            | Link to the source code repository                                 |
| `Conference`   | multi_select   | Publication venue(s); see Conference Options below                 |
| `Name`         | title          | Paper title; serves as the primary display name of the entry       |

### Conference Options

| Option           | Color   | Usage                                      |
|------------------|---------|--------------------------------------------|
| `Science Robotics` | pink  | Science Robotics journal                   |
| `ICLR`           | blue    | International Conference on Learning Representations |
| `ICCV`           | default | International Conference on Computer Vision |
| `NeurIPS`        | purple  | Neural Information Processing Systems      |
| `CVPR`           | green   | Conference on Computer Vision and Pattern Recognition |
| `ICRA`           | orange  | International Conference on Robotics and Automation |
| `CoRL`           | yellow  | Conference on Robot Learning               |
| `Preprint`       | gray    | Arxiv preprint, not yet published at a venue |
| `ECCV`           | default | European Conference on Computer Vision       |

---

## Usage Notes

- **All projects use the standard schema.** When creating a new References database, replicate this schema exactly.
- **Year options** (`2023`–`2026`) should be extended as needed when papers from additional years are added.
- **Conference options** should be extended as new venues become relevant. Use `Preprint` for papers not yet published at a conference/journal. Because `Conference` is a `multi_select`, a single entry can belong to multiple venues (e.g., a preprint later accepted to a conference).
- The Nav Benchmark DB was upgraded from a simplified schema on 2026-02-20: the `Arixv` typo was fixed to `Arxiv`, and `Year`, `Conference`, `Github` fields were added.
