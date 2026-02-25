# Template: technical-review-template.md

## Purpose

This template defines the Notion page body for a Technical Review sub-page.
It is used by the **extract-technical-details** skill when creating a reproduction-
oriented technical review under a paper's References DB entry.

## Writing Style Guidelines

- **Write for someone reproducing the work.** Every detail that affects reproduction
  should be included.
- **Use tables heavily.** Hyperparameters, observation spaces, reward components,
  and dataset statistics are best presented as tables.
- **Be precise with numbers.** Always include units. If a value is approximate
  or inferred, note it.
- **Quote key formulas.** Use inline LaTeX-style notation where needed for rewards,
  losses, and architecture definitions.
- **Flag gaps.** If the paper does not specify a detail needed for reproduction,
  explicitly note "Not specified in paper."
- **Keep prose minimal.** Favor structured lists and tables over long paragraphs.

## Variable Placeholders

Variables in `{braces}` are filled by Claude at generation time:

| Variable                 | Description                                             |
|--------------------------|---------------------------------------------------------|
| `{paper_title}`          | Full paper title                                        |
| `{arxiv_url}`            | Canonical arxiv URL                                     |
| `{pipeline_overview}`    | High-level pipeline description with stage listing      |
| `{stage_sections}`       | Repeated per stage — see Stage Section Template below   |
| `{hyperparams_summary}`  | Consolidated hyperparameters table                      |
| `{hardware_evaluation}`  | Hardware setup, test objects, evaluation metrics         |

## Stage Section Template

Each stage section follows this structure:

```markdown
### Stage N: <Stage Name>

**Goal:** <one-sentence purpose>

**Input:** <what this stage receives>
**Output:** <what this stage produces>

#### Architecture / Algorithm
<network architecture, algorithm description, key equations>

#### Training Configuration
| Parameter | Value |
|-----------|-------|
| Framework | ... |
| Environments | ... |
| ...       | ...   |

#### Dataset
<data sources, sizes, splits, filtering criteria>

#### Key Design Choices
<why this approach, alternatives considered, ablation insights>
```

## Regeneration Rules

- The Technical Review sub-page has NO `## My Notes` section.
- On regeneration, the entire page content is replaced via `replace_content`.
- User notes for this paper live on the parent page's `## My Notes` section.

---

<!-- TEMPLATE BODY START — content below is written to the Notion page -->

## Technical Review

::: callout {icon="🔧" color="gray_bg"}
**Reproduction-oriented technical details for:** {paper_title}
**Paper:** {arxiv_url}
:::

### Pipeline Overview
{pipeline_overview}

{stage_sections}

### Key Hyperparameters Summary
{hyperparams_summary}

### Hardware & Evaluation Setup
{hardware_evaluation}
