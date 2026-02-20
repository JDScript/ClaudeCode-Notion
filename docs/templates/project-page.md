# Template: project-page.md

## Purpose

This template defines the Notion page body for a new research project page.
It is used by the **scaffold-project** skill when creating a project entry under the Projects database.
The structure is modeled after the Functional Reorientation project page in Notion.

## Variable Placeholders

Variables in `{braces}` are filled by Claude or user input at creation time:

| Variable               | Description                                                         |
|------------------------|---------------------------------------------------------------------|
| `{one_liner}`          | A single sentence describing what the project is about              |
| `{application}`        | The target application domain or use case                           |
| `{proposed_solutions}` | Brief description of the approach(es) being explored                |
| `{MEETING_NOTES_URL}`  | Replaced with the URL of the Meeting Notes sub-page after creation  |

## Page Icon

The icon for the project page should be chosen by the user or by Claude based on the project
topic. It is set as the Notion page icon property, not inside the template body.

## Post-Creation Steps

After the page body is written using this template, the scaffold-project skill must:

1. Create a **Meeting Notes** sub-page and obtain its URL, then substitute `{MEETING_NOTES_URL}`
   in the Quick Links callout.
2. Create a **Tasks** database as an inline database block on this page.
3. Create a **References** database as an inline database block on this page.

These databases are not represented in this template body — they are created separately by the
skill and appended below the columns block.

---

<!-- TEMPLATE BODY START — content below is written to the Notion page -->

<columns>
	<column>
		::: callout {icon="💡" color="gray_bg"}
			### Project Overview
			**One Liner:** {one_liner}
			**Application:** {application}
			**Proposed Solutions:** {proposed_solutions}
		:::
	</column>
	<column>
		::: callout {icon="💡" color="green_bg"}
			### Quick Links
			<page url="{MEETING_NOTES_URL}">Meeting Notes</page>
		:::
	</column>
</columns>
