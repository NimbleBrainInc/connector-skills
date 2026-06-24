---
name: outlook
description: >-
  How to use the Outlook connector's tools — listing, reading, drafting, sending,
  replying, and filing mail, plus reading and creating calendar events. Use when
  the user works with Outlook, Microsoft 365 mail, their inbox, drafts, mail
  folders, meeting invites, or calendar events.
---

You are using the **Outlook** connector (Microsoft 365, via Composio). Its tools act
on the connected user's mailbox and calendar through the Microsoft Graph API. Apply
this guidance whenever you touch Outlook.

## Find the message first

- To act on a specific message you need its **message id** — an opaque Graph string
  like `AAMkAGI2TAAA=`, **not** an email address. List or search first to get ids,
  then read/reply/move/delete by id. Pass the **complete, untruncated** id; an
  ellipsis-shortened id will fail.
- **List messages** to find what you need. You can scope by folder using a
  **well-known folder name** (`inbox`, `sentitems`, `drafts`, `deleteditems`)
  directly as the folder id, or a specific folder's id.
- Listing supports **OData-style filtering, sorting, and pagination** rather than
  plain keyword search — filter on properties like sender, subject, received date,
  and read state, and page through with a limit instead of pulling everything. To
  walk a thread, filter by **conversationId** to gather every message in it.
- `OUTLOOK_GET_MESSAGE` fetches a single message's full details by id — use it when
  a listing gives you only summaries or you only have an id (e.g. from a webhook).

## Replying vs. composing

- **Reply** with `OUTLOOK_REPLY_EMAIL` (carries the original **message id** plus a
  `comment` body, CC/BCC optional). This keeps the message in the same conversation
  and quotes context for you — don't send a fresh email with the same subject, which
  starts a *new* thread and reads as a mistake. Use **forward** to send a message on
  to a new recipient.
- Prefer **creating a draft** (`OUTLOOK_CREATE_DRAFT`) over sending directly when the
  user hasn't explicitly said "send" — show it for confirmation, then send the draft
  by its id with `OUTLOOK_SEND_DRAFT`. Only send outright (`OUTLOOK_SEND_EMAIL`) when
  the intent is unambiguous.
- Bodies can be plain text or HTML — set the HTML flag when you're sending formatted
  content, and quote sparingly rather than pasting the whole prior thread back.

## Filing and folders

- Outlook organizes mail into real **folders** (unlike Gmail labels). Use
  `OUTLOOK_LIST_MAIL_FOLDERS` to resolve a folder name to its id, then
  `OUTLOOK_MOVE_MESSAGE` to file a message into it by destination id. "Archive" and
  "file" mean *move to a folder*, not toggle a label.
- `OUTLOOK_DELETE_MESSAGE` removes a message (it goes to Deleted Items) — a
  user-visible, destructive action; confirm before deleting and never bulk-delete
  without explicit scope.

## Calendar

- `OUTLOOK_LIST_EVENTS` reads the calendar; filter/page the same way as messages.
- `OUTLOOK_CALENDAR_CREATE_EVENT` needs a subject, **start and end datetimes, and a
  time zone** — supply ISO 8601 datetimes and an explicit time zone so the event
  doesn't land at the wrong hour. Confirm attendees and time before creating, since
  creating an event sends invites.
- `OUTLOOK_ACCEPT_EVENT` responds to a meeting invite by **event id**; this notifies
  the organizer, so only respond on the user's explicit instruction.

## Care

- Sending, deleting, moving, and responding to invites are **user-visible** and hard
  to undo. Confirm recipients, content, and targets before acting.
- Respect the user's voice for any drafted content; match the tone of the thread.
- This is a large, ~280-tool toolkit — only a curated subset is exposed. If a
  capability isn't available as a tool, say so rather than inventing a tool name.
- Large mailboxes: page through results with a limit rather than requesting
  everything, and summarize.
