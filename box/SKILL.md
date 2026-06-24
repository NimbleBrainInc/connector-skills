---
name: box
description: >-
  How to use the Box connector's tools — browsing folders, uploading and
  downloading files, copying, sharing via links, commenting, and asking Box AI
  questions about file contents. Use when the user works with Box, files,
  folders, documents, shared links, or cloud storage.
---

You are using the **Box** connector (via Composio). Its tools act on the connected
user's Box account. Apply this guidance whenever you touch Box.

## Everything is an id

- Box addresses files and folders by **numeric id**, not by path or name. The
  **root** folder is always id **`0`** — start there when the user gives no
  location.
- There is no name-based lookup tool here. To find "the Q3 report," **list items
  in a folder** (start at `0`) and walk down by matching names, capturing the id
  of each step. Don't guess ids.
- `LIST_ITEMS_IN_FOLDER` returns files, subfolders, and web links together; each
  entry carries the `id` and `type` you'll pass to other tools. Page through large
  folders (offset/marker) rather than assuming one call returns everything — and
  note the **root folder (`0`) supports offset paging only**, not marker paging.

## Common workflows

- **Read a file's content for the user:** prefer **ASK_QUESTION** (Box AI over the
  file) when they want an answer or summary — pass the file id(s); it can return
  citations. Use **DOWNLOAD_FILE** only when they need the raw bytes/text itself.
- **Upload:** `UPLOAD_FILE` needs the destination **folder id** plus the content;
  uploading to a name that already exists in that folder conflicts — to replace
  content, update the existing file instead of uploading a duplicate.
- **Rename / move:** there's no separate move tool — `UPDATE_FILE` does both
  (change `name` to rename, change the parent folder id to move). Confirm the
  target parent id first.
- **Copy:** `COPY_FILE` duplicates into a destination folder id; the original is
  untouched.
- **Get details:** `GET_FILE_INFORMATION` / `GET_FOLDER_INFORMATION` resolve
  metadata (name, size, parent, modified time) for an id you already hold.

## Sharing

- `ADD_SHARED_LINK_TO_FILE` / `ADD_SHARED_LINK_TO_FOLDER` create a public-ish URL.
  Access level matters — an "anyone with the link" link exposes the content beyond
  the org. **Confirm the intended audience** (company-only vs. open) before
  creating one, and surface the resulting URL to the user.

## Commenting

- `CREATE_COMMENT` attaches a comment to a file id; it's visible to collaborators.
  Don't post on the user's behalf without their wording.

## Care

- **DELETE_FILE** sends the file to **trash** (recoverable for a window, then
  purged) — still user-visible and disruptive. Confirm the exact file id before
  deleting; never bulk-delete without explicit scope.
- Uploads, updates, copies, shared links, and comments are all immediately visible
  to collaborators. Confirm the destination folder id before writing.
- A "not found" id may mean the item isn't shared with the connected account, not
  that it's gone — say so rather than inventing an id.
- Use `GET_CURRENT_USER` if you need to confirm whose account you're acting on.
