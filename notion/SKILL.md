---
name: notion
description: >-
  How to use the Notion connector's tools — searching the workspace, reading and
  creating pages, and appending blocks. Use when the user works with Notion, docs,
  wikis, databases, or pages.
---

You are using the **Notion** connector (remote MCP). Its tools act on the connected
Notion workspace. Apply this guidance whenever you touch Notion.

## Find the target first

- **Search** the workspace to resolve a human reference ("the roadmap doc") into a
  **page or database id** before reading or writing. Don't guess ids.
- Notion has two distinct object types: **pages** (free-form, block-based) and
  **databases** (rows = pages with typed properties). Check which you're dealing
  with — you *query* a database (with filters/sorts) but *read* a page's blocks.

## Reading and writing

- Page content is a tree of **blocks** (paragraphs, headings, to-dos, toggles,
  tables). Reading a page may require fetching its block children, sometimes paged.
- To add content, **append blocks** to a page rather than trying to rewrite it
  wholesale. Creating a page requires a **parent** (a page id or a database id); a
  page created in a database must supply that database's required properties.
- Notion expects its own rich-text/block shapes, not raw Markdown — map headings,
  lists, and checkboxes to the corresponding block types.

## Databases

- When adding a row to a database, match the existing **property schema** (names and
  types — title, select, date, relation). Inspect the schema before writing; a wrong
  property name silently no-ops or errors.

## Care

- Edits are immediately visible to everyone with access. Confirm the target page
  before writing; prefer appending or creating over destructive edits.
- Permissions are per-integration: if a page isn't found, it may simply not be
  shared with the connector rather than missing — say so instead of inventing it.
