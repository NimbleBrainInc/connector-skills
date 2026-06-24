---
name: dropbox
description: >-
  How to use the Dropbox connector's tools — browsing folders, searching,
  reading file content, creating and moving files, and managing shared links.
  Use when the user works with Dropbox, cloud files, folders, documents,
  shared links, file requests, or storage.
---

You are using the **Dropbox** connector (remote MCP). Its tools act on the
connected user's Dropbox account. Apply this guidance whenever you touch Dropbox.

## Paths and ids

- Tools address items by **path** or **id**. A path starts with `/` and is
  relative to the account root (`/Reports/q2.pdf`); the root itself is the empty
  string `""`, never `/`. An id looks like `id:abc123xyz` and is **case-sensitive**.
- Prefer **ids** when you have them — an id survives renames and moves; a path
  breaks the moment the file is relocated.
- Don't guess paths. **Search** or **ListFolder** first to resolve a human
  reference ("the budget deck") into a real path or id, then act on it.

## Find before you act

- **Search** finds files/folders by name or content and can filter by folder,
  type, or date — use the filters instead of pulling a broad result and sieving
  it yourself.
- **ListFolder** browses one directory and returns up to ~100 items; large
  folders paginate by **cursor**, so follow the cursor rather than assuming the
  first page is the whole folder.
- **GetFileMetadata** reads properties (size, dates, MIME type, id) for a single
  item; **GetUsageAndQuota** reports storage used vs. available, in bytes.

## Reading content

- **GetFileContent** extracts text from PDFs, Word docs, and other
  text-representable files, capped around **5 MB** and possibly truncated — for a
  large file expect a partial read, not the whole thing.
- To hand a file to the user (or a non-text/binary file), create a **download
  link** rather than trying to read bytes; these links are temporary/single-use.

## Writing, moving, sharing

- **CreateFile** writes a text file from UTF-8 content (~5 MB cap);
  **CreateFolder** makes a directory by path.
- **Move** relocates or renames; **Copy** duplicates. For a large item these may
  run as a **background job** — you get a job handle and must poll
  **CheckJobStatus** until it completes rather than assuming instant success.
  Note: case-only renames aren't supported.
- **CreateSharedLink** generates a link (and can invite a bounded set of viewers,
  ~25, by email); **ListSharedLinks** / **GetSharedLinkMetadata** inspect existing
  links, including audience and expiration. Confirm the intended audience before
  sharing — a link can expose a file beyond the user's expectation.

## Versions and recovery

- **ListFileRevisions** shows a file's history; **RestoreFileRevision** rolls a
  file back to an earlier version, and **RestoreFolder** rewinds a whole folder.
- **Delete** moves items to *Deleted files* (trash), not permanent removal —
  recoverable for a window that depends on the user's plan. **ListRestoreEvents**
  surfaces recent edits/deletions if you need to find what changed.
- **File requests** (**ListFileRequests** / **GetFileRequest** /
  **CreateFileRequest**) let others upload into a folder via a request URL with an
  optional deadline.

## Care

- **Delete, Move, Restore, and shared-link creation are user-visible and affect
  shared collaborators.** Confirm the exact target (path or id) before any of
  these; never bulk-delete or bulk-move without explicit, narrow scope.
- A shared link persists until revoked — treat creating one as exposing the file.
- If an item isn't found, it may be outside the scopes the connector was granted
  rather than missing — say so instead of inventing a path.
