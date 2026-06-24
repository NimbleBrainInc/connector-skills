---
name: googledocs
description: >-
  How to use the Google Docs connector's tools — finding documents, reading
  their text, creating docs from Markdown, inserting and replacing text, copying,
  and exporting to PDF. Use when the user works with Google Docs, documents,
  reports, write-ups, meeting notes, or wants to draft, edit, or export a doc.
---

You are using the **Google Docs** connector (via Composio). Its tools act on the
connected user's Google Docs (backed by Drive). Apply this guidance whenever you
touch Google Docs.

## Find the document first

- To act on an existing doc you need its **document id** (the long token in the
  doc URL: `docs.google.com/document/d/<id>/edit`). Don't guess ids.
- Resolve a human reference ("the Q3 report") with **search** before reading or
  editing. Search accepts a name/content query and filters (date, starred,
  shared); it uses **Drive query semantics** — `name contains '…'` matches the
  title, `fullText contains '…'` matches body content, and `contains` only
  matches whole tokens (search `roadmap`, not `roadm`). Quote a phrase to match
  it as a unit.
- Reading: use the **plaintext** tool to get the doc's text for summarizing or
  Q&A; use **get-by-id** when you need the structured document (and to confirm a
  doc exists — it errors if not found).

## Creating

- Prefer the **Markdown create** tool over the plain create when you have
  formatted content — headings, lists, tables, blockquotes, and inline emphasis
  all map cleanly from Markdown. Plain create is for blank or simple-text docs.
- **Copy** an existing doc (with a new title) when the user wants a doc *like*
  another one — a template instance — rather than building it from scratch.
- New docs land in the user's Drive root by default.

## Editing existing docs

- Google Docs is an **index-based** model: every character (including newlines)
  has a position, and edits target positions, not line numbers. This is fiddly —
  inserting at the wrong index shifts everything after it.
- **Insert text:** prefer appending to the end of the document over computing an
  index by hand. When you must insert mid-doc, derive a valid index from the
  document structure; an index at a structural boundary (e.g. inside a table) is
  rejected.
- **Replace-all text:** the safe way to do targeted edits — find a literal
  string (or pattern) and swap it everywhere. Use a distinctive anchor so you
  don't replace unintended matches; check the case-sensitivity option.
- **Markdown update replaces the ENTIRE document body** with the supplied
  Markdown. Treat it as a full rewrite, not a patch — read the current content
  first, edit the whole thing, then write it back. Don't reach for it to change
  one line.

## Export

- **Export as PDF** renders the doc to a PDF (Drive caps export at ~10MB). Use it
  to hand off a finished doc; it does not change the source document.

## Care

- Edits are immediately visible to everyone with access, and the index/replace
  and full-Markdown-rewrite tools have **no undo via the API** — confirm you have
  the right document id before writing.
- The full-Markdown-rewrite is the most destructive tool here: it discards
  whatever was in the doc. Never run it without first reading what you'd
  overwrite, and prefer copy-then-edit when the original must be preserved.
- Permissions are per-connection: if a doc isn't found, it may simply not be
  shared with the connected account rather than missing — say so instead of
  inventing content.
- Match the user's voice for any drafted prose; show drafts for confirmation
  before creating or overwriting when the intent isn't explicit.
