---
name: googledrive
description: >-
  How to use the Google Drive connector's tools — finding files and folders,
  reading metadata, downloading content, creating text files and folders,
  moving and copying, and sharing. Use when the user works with Google Drive,
  Drive, Docs, Sheets, Slides, files, folders, or shared documents.
---

You are using the **Google Drive** connector (via Composio). Its tools act on the
connected user's Drive. Apply this guidance whenever you touch Drive.

## Find the file first

- Resolve a human reference ("the Q3 deck", "the budget folder") into a **file id**
  or **folder id** before reading, downloading, moving, copying, or sharing. Don't
  guess ids — find, then act by id.
- Use the **find/search** tool to locate files and the **find-folder** tool to
  locate folders. Search supports Drive query syntax, not just keywords: filter on
  `name` (`name contains 'budget'`, `name = 'Report'`), `mimeType`
  (`mimeType = 'application/vnd.google-apps.document'`, `mimeType contains 'image/'`),
  `fullText contains 'phrase'` for body content, plus `trashed = false`,
  `sharedWithMe`, `starred`, and modified-date filters. Combine terms with `and`.
  Escape single quotes in names with a backslash.
- Scope to a folder when you can (pass the parent/folder id) instead of searching all
  of Drive and filtering yourself. Multiple files can share a name — confirm the right
  one (by parent, owner, or modified date) before acting.

## Files, folders, and types

- A **folder** is just a file with mimeType `application/vnd.google-apps.folder`.
  Use the create-folder tool for folders; reparent a file by **moving** it.
- Google-native docs (Docs/Sheets/Slides, `application/vnd.google-apps.*`) have no
  raw bytes — downloading them requires an **export** mimeType (e.g. PDF, DOCX,
  CSV). Binary/uploaded files download directly. If a download fails on a native
  doc, you likely need to specify an export format.
- Get-metadata returns `mimeType`, `parents`, `trashed`, owners, and size — check it
  to learn an item's type and where it lives before you act.

## Creating and organizing

- Create text-backed files with the **create-file-from-text** tool (give it a name,
  text content, and optionally a parent folder id). This connector's create surface
  is text/upload — it does not author rich native Docs formatting.
- **Move** changes a file's parent (it relocates, it does not duplicate). **Copy**
  duplicates. Don't move when the user wanted a copy, or vice versa.
- New files land in "My Drive" root unless you pass a parent id — set the parent at
  creation rather than creating then moving.

## Sharing

- Sharing is done by **creating a permission** on a file id: a `type` (user, group,
  domain, anyone) + `role` (reader, commenter, writer, owner) + the target
  (email address or domain). `type: anyone` makes a file **publicly accessible** —
  use it only when the user explicitly asks for a public/shareable link.
- Granting `writer`/`owner` or domain-wide access is a meaningful exposure. Confirm
  the recipient and role before sharing; prefer the least role that satisfies the ask.

## Care

- **Moving and sharing are immediately visible** to others with access, and changing
  ownership or revoking access can lock people out. Confirm the target file and the
  scope before any move or permission change; never bulk-share or bulk-move without
  explicit scope.
- A file "not found" may simply not be shared with the connected account rather than
  deleted — say so instead of inventing it.
- Large Drives: page through search results rather than requesting everything, and
  summarize. Match on parent/owner/date to disambiguate same-named files.
