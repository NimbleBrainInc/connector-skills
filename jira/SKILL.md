---
name: jira
description: >-
  How to use the Jira connector's tools — searching issues with JQL, reading and
  creating issues, editing fields, transitioning status, assigning, and commenting.
  Use when the user works with Jira, issues, tickets, bugs, stories, epics,
  sprints, boards, backlogs, or project tracking.
---

You are using the **Jira** connector (via Composio). Its tools act on the connected
Jira site (projects, issues, boards). Apply this guidance whenever you touch Jira.

## Find the target first

- To act on an issue you reference it by **issue key** (e.g. `PROJ-123`) or numeric
  id. Resolve a human description ("the login bug") into a key by **searching**
  first — don't guess keys.
- Search uses **JQL**, not plain keywords. Build a query: `project = PROJ`,
  `status = "In Progress"`, `assignee = currentUser()`, `text ~ "login"`, with
  `ORDER BY created DESC`. Combine clauses (`project = PROJ AND status != Done
  ORDER BY updated DESC`) instead of fetching broadly and filtering yourself.
- Prefer the **POST JQL search** tool for anything non-trivial — it carries the
  query in the body and handles complex/long JQL more reliably than the simpler
  search variant.
- Reading an issue returns its summary, description, status, assignee, comments,
  and custom fields; pass a `fields` list to keep responses small on large issues.

## Creating and editing

- Creating an issue requires at minimum a **project key**, an **issue type**, and a
  **summary**. Don't assume the type exists — list a project's issue types first if
  unsure (a `Bug`/`Story`/`Task` that isn't configured for that project will error).
- **Edit** an existing issue rather than recreating it. Editing sets fields
  (summary, description, priority, labels) in place.

## Status transitions are not free-text

- You **cannot** set an issue's status by name directly. Status changes go through a
  **workflow transition**: first **get the available transitions** for that specific
  issue, pick the one you want, then **transition** the issue with that
  **transition id**. The valid transitions depend on the issue's current status and
  the project's workflow, so always fetch them per-issue rather than caching ids.

## Users are accountIds, not usernames

- Assigning and user lookups use Atlassian **accountId**, not username or email
  (Jira's privacy changes removed username access). To assign by a person's name,
  **find the user** first to get their accountId, then assign with it. Setting the
  assignee to null/unassigned is the way to clear an assignee.

## Comments and context

- Add comments to record decisions or hand off context; comments are visible to
  everyone with project access. Don't paste large dumps — link or summarize.
- Boards and sprints are agile concepts layered on projects: list boards to find a
  board id, then list its sprints. Issues live in projects; sprint membership is a
  board-level view.

## Care

- **Deleting an issue is permanent and irreversible** — there is no trash. Never
  delete without explicit, specific confirmation; default to closing/transitioning
  to a "Done" or "Won't Do" status instead.
- Transitions, edits, assignments, and comments are immediately visible to the team
  and can fire notifications and automations. Confirm the target issue key before
  writing; prefer commenting/transitioning over destructive edits.
- If an issue or project "isn't found," it may simply be outside the connector's
  permissions rather than missing — say so instead of inventing it.
- Large result sets: page through search results and summarize rather than pulling
  every issue at once.
