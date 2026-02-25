# Datasets Database Schema

This document describes the Datasets database schema used in the Navigation Reasoning Benchmark project in Notion.

---

## Schema

Collection IDs are stored in `.claude/notion-config.json` under each project's `datasets_db` field (populated by the `init` skill).

```sql
CREATE TABLE datasets (
    url TEXT UNIQUE,
    createdTime TEXT,
    "Arxiv" TEXT,                   -- type: url
    "Project Page" TEXT,            -- type: url
    "Year" TEXT,                    -- type: select, options: ["2023", "2024", "2025", "2026"]
    "License" TEXT,                 -- type: select, options: ["CC BY 4.0", "CC BY-SA 4.0", "CC BY-NC-SA 4.0", "CC0 1.0", "Apache 2.0", "MIT"]
    "Scene Type" TEXT,              -- type: multi_select, options: ["Indoor", "Outdoor", "Urban", "Campus", "Off-road", "Mix"]
    "Embodiment" TEXT,              -- type: rich_text
    "Camera" TEXT,                  -- type: rich_text
    "LiDAR" TEXT,                   -- type: rich_text
    "IMU" INTEGER,                  -- type: checkbox
    "GPS" INTEGER,                  -- type: checkbox
    "Episodes" REAL,                -- type: number
    "Video Length (h)" REAL,        -- type: number
    "Trajectory Length (km)" REAL,  -- type: number
    "Scene Size (km²)" REAL,       -- type: number
    "Name" TEXT                     -- type: title
)
```

### Property Details

| Property               | Notion Type    | Notes                                                        |
|------------------------|----------------|--------------------------------------------------------------|
| `url`                  | (internal)     | Notion page URL, used as unique row identifier               |
| `createdTime`          | (internal)     | ISO-8601 timestamp of page creation                          |
| `Arxiv`                | url            | Link to the dataset paper on arxiv.org                       |
| `Project Page`         | url            | Link to the official dataset website                         |
| `Year`                 | select         | Publication year; options: `2023`, `2024`, `2025`, `2026`    |
| `License`              | select         | Data license; see License Options below                      |
| `Scene Type`           | multi_select   | Environment type(s); see Scene Type Options below            |
| `Embodiment`           | rich_text      | Robot platform or data collection method                     |
| `Camera`               | rich_text      | Camera configuration (count, type, model)                    |
| `LiDAR`                | rich_text      | LiDAR configuration (count, type, model)                     |
| `IMU`                  | checkbox       | Whether IMU data is included                                 |
| `GPS`                  | checkbox       | Whether GPS data is included                                 |
| `Episodes`             | number         | Number of episodes or trajectories                           |
| `Video Length (h)`     | number         | Total recording duration in hours                            |
| `Trajectory Length (km)` | number       | Total trajectory distance in kilometers                      |
| `Scene Size (km²)`    | number         | Total scene coverage area in square kilometers               |
| `Name`                 | title          | Dataset name; serves as the primary display name             |

### License Options

| Option             | Usage                                    |
|--------------------|------------------------------------------|
| `CC BY 4.0`        | Creative Commons Attribution 4.0        |
| `CC BY-SA 4.0`     | Creative Commons ShareAlike 4.0         |
| `CC BY-NC-SA 4.0`  | Creative Commons NonCommercial ShareAlike|
| `CC0 1.0`          | Creative Commons Zero (public domain)   |
| `Apache 2.0`       | Apache License 2.0                      |
| `MIT`              | MIT License                             |

### Scene Type Options

| Option     | Usage                                         |
|------------|-----------------------------------------------|
| `Indoor`   | Indoor environments (buildings, rooms)        |
| `Outdoor`  | General outdoor environments                  |
| `Urban`    | Urban streets and city environments           |
| `Campus`   | University or institutional campus settings   |
| `Off-road` | Unstructured terrain, trails                  |
| `Mix`      | Mixed indoor/outdoor or varied environments   |

---

## Usage Notes

- **Year options** (`2023`–`2026`) should be extended as needed.
- **License options** should be extended when new license types are encountered.
- **Scene Type options** should be extended as new environment categories become relevant. Because `Scene Type` is `multi_select`, a single entry can belong to multiple types.
- **Numeric fields** (Episodes, Video Length, Trajectory Length, Scene Size) may be left empty when data is unavailable.
- **Embodiment** uses free-form text to accommodate diverse platforms (e.g., `Husky`, `Human-ego`, `Spot`, `Wheeled`, `Head-mount`).
- **Camera and LiDAR** use concise configuration strings (e.g., `2 RGB (Stereo)`, `1 (Ouster OS1-128)`).
