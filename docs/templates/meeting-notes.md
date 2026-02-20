# Template: meeting-notes.md

## Purpose

This template defines the Notion page body for a meeting notes entry.
It is used by the **meeting-notes** skill when creating a new meeting notes page, either as a
standalone page or as a sub-page within a project.

## Title Format

The page title should follow one of these formats:

- `{Topic} - {Mon DD, YYYY}` — when a specific topic or project name is provided
  (e.g., "Functional Reorientation - Feb 20, 2026")
- `Meeting - {Mon DD, YYYY}` — when no specific topic is given
  (e.g., "Meeting - Feb 20, 2026")

The date uses the abbreviated month name, zero-padded day, and four-digit year.

## Variable Placeholders

Variables in `{braces}` are filled by user input at creation time, or left as descriptive
placeholder text when the user does not supply a value:

| Variable             | Description                                                       |
|----------------------|-------------------------------------------------------------------|
| `{attendee_list}`    | Names of meeting participants, one per bullet                     |
| `{agenda_items}`     | Numbered list of agenda items                                     |
| `{discussion_content}` | Free-form notes from the meeting discussion                   |
| `{action_items}`     | Tasks assigned during the meeting; uses Notion to-do checkbox syntax |
| `{decisions}`        | Key decisions reached during the meeting                         |

## Action Items Syntax

Action items use Notion's to-do checkbox block syntax (`- [ ]`). Each item should be on its own
line prefixed with `- [ ]`. The meeting-notes skill converts these lines into Notion to-do blocks
so that they appear as interactive checkboxes in the Notion UI.

Example:
```
- [ ] Alice: send dataset link by Friday
- [ ] Bob: review related work section
```

---

<!-- TEMPLATE BODY START — content below is written to the Notion page -->

## Attendees
- {attendee_list}

## Agenda
1. {agenda_items}

## Discussion
{discussion_content}

## Action Items
- [ ] {action_items}

## Decisions
{decisions}
