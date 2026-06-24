---
name: linear
description: >-
  How to use the Linear connector's tools — finding, reading, creating, and
  updating issues, projects, comments, and cycles, plus searching Linear's
  product docs. Use when the user works with Linear, issues, tickets, bugs,
  sprints, cycles, projects, roadmaps, the backlog, or "my Linear issues".
---

You are using the **Linear** connector (remote MCP). Its tools act on the connected
Linear workspace under the authenticated user's identity. Apply this guidance
whenever you touch Linear.

## Find the target first

- Resolve a human reference into an id before reading or writing. Use the **list**
  tools (issues, projects, teams, users, labels, statuses, cycles) to look up ids;
  don't guess them.
- Linear issues have a **human identifier** like `ENG-123` (team key + number) as
  well as an internal id. Use the `ENG-123` identifier when the user gives one —
  the issue tools accept it.
- Most writes require a **team**. Resolve the team (by key/name) before creating an
  issue if the user hasn't named one and the workspace has more than one.

## Filtering and search

- Filters are **flat parameters**, not raw GraphQL: `assignee`, `team`, `project`,
  `state`/`status`, `label`, `cycle`, plus date and priority filters. Combine them
  instead of listing everything and filtering yourself.
- `assignee` accepts a user **id, name, email, or the literal `me`** (current user);
  use `null` for unassigned. Prefer `me`/`list_my_issues` for "my issues" rather
  than first looking up your own user id.
- **Priority is an integer**: `0`=none, `1`=urgent, `2`=high, `3`=medium, `4`=low.
  Map the user's words to these; don't pass a priority string.
- **Status** ("state") is workflow-specific per team (e.g. Todo / In Progress /
  Done, plus custom states). List the team's statuses to get exact names/ids before
  filtering or setting one — a wrong name won't match.
- Pagination cursors must be the server's returned **opaque/UUID** values; page
  through large result sets rather than requesting everything at once, and summarize.

## Common workflows

- **Create an issue**: needs a `title` and a `team`; optionally description,
  assignee, priority, labels, project, cycle, and a `parent` for a sub-issue. Write
  descriptions in Markdown — Linear renders it.
- **Triage / update**: change status, assignee, priority, labels, or project on an
  existing issue by id. Move work forward with status changes rather than closing and
  recreating.
- **Comment**: add a comment to an issue for discussion or status notes; don't edit
  the issue description to leave a comment.
- **Docs questions**: to answer "how does Linear do X", use the documentation-search
  tool — that's Linear's own product help, separate from workspace issues/projects.

## Care

- Creates and updates are **immediately visible** to the team and can trigger
  notifications. Confirm the team, assignee, and especially the **status** before
  writing — changing state moves work on shared boards and can fire automations.
- Don't bulk-create or bulk-reassign without explicit scope; one stray loop spams a
  team's board and inboxes.
- If an issue, project, or team "isn't found", it may be in a team the connector's
  user can't see rather than nonexistent — say so instead of inventing an id.
- Match the team's conventions for titles and labels; keep comments concise and in
  the user's voice.
