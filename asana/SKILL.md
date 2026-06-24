---
name: asana
description: >-
  How to use the Asana connector's tools — finding, creating, and updating tasks,
  subtasks, comments, and projects, and navigating workspaces and sections. Use when
  the user works with Asana, tasks, to-dos, projects, assignees, due dates, or team
  workflows.
---

You are using the **Asana** connector (via Composio). Its tools act on the connected
user's Asana account. Apply this guidance whenever you touch Asana.

## Everything is a GID — resolve names first

- Asana addresses every object by an opaque **GID** (global id), not its name. You
  almost always need a workspace, project, section, task, or user GID before you can
  act. Don't guess GIDs.
- Start by establishing context: **get the current user** (returns the user record
  and the workspaces they can access) and **list workspaces**. Most write tools need a
  `workspace` GID, and an org-level project also needs a `team` GID.
- To turn a human reference ("the Launch project", "Priya") into a GID, use
  **typeahead**: it takes a `workspace`, a `resource_type` (`task`, `project`,
  `user`, `tag`, `section`, …), and a `query` string. It's a fast prefix match — not
  exhaustive, not paginated, and capped in size. Use it to resolve a name, then act by
  GID; don't rely on it to enumerate everything.

## Finding tasks

- To list real task sets, prefer the scoped readers: **tasks from a project** (needs
  the project GID), **a user's task list** (needs a user GID — or `me` — plus a
  workspace GID), or **a task's subtasks**. These page; request a page and summarize
  rather than pulling everything.
- **Get a task** returns the full record by GID. Ask for the fields you need; reads
  default to a compact shape, so request `assignee`, `due_on`, `completed`, `notes`,
  etc. explicitly when you need them.
- Read a task's discussion with **get stories for a task** (the activity/comment feed).

## Creating and updating

- **Create a task** requires at least one association: a `workspace`, a `parent`
  (makes it a subtask), or `projects`. Set `assignee` to a user GID or `me`, `due_on`
  for a date, `notes` for the description. Followers, projects, and tags are honored
  **at creation**; afterward they need their own dedicated calls, not a plain update.
- **Create a subtask** under a parent task GID; **create a task comment** appends to a
  task's activity feed.
- **Update a task** changes properties on an existing task by GID (name, notes,
  assignee, due date). Mark a task done by setting `completed: true` — this is the
  normal "finish it" action, and it's reversible. Updating `assignee`/`due_on`/etc.
  via update is fine, but changing followers/projects/tags goes through their own
  endpoints, not the task update.

## Projects and structure

- **Create / get a project**, **list a workspace's projects**, and **get the sections
  in a project** (the columns/headers like To Do / In Progress). Getting a project
  returns metadata, not its task list — fetch tasks-from-a-project for that. Creating
  a project in an organization needs both `workspace` and `team`.

## Care

- Edits are immediately visible to everyone with access to the resource. Confirm the
  target task/project (by GID, not just name) before writing.
- **Prefer completing over deleting.** Marking a task `completed` is the reversible
  default; deletes (where exposed) are permanent and non-recoverable — never delete
  without explicit, scoped confirmation, and never bulk-delete.
- Match the existing structure: assign to real user GIDs, drop tasks into existing
  sections, and reuse the project/workspace already in context rather than inventing
  new ones.
- If a task or project isn't found, it may simply not be shared with the connected
  account rather than missing — say so instead of inventing it. Page large result
  sets and summarize rather than dumping everything.
