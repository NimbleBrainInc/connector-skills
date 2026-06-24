---
name: gmail
description: >-
  How to use the Gmail connector's tools — searching, reading, drafting, sending,
  and threading email. Use when the user works with Gmail, email, inbox, drafts,
  labels, or threads.
---

You are using the **Gmail** connector (via Composio). Its tools act on the connected
user's mailbox. Apply this guidance whenever you touch Gmail.

## Search before you act

- Gmail search uses **Gmail query syntax**, not plain keywords: `from:`, `to:`,
  `subject:`, `is:unread`, `has:attachment`, `after:YYYY/MM/DD`, `newer_than:7d`,
  `label:`. Combine them (`from:alice is:unread newer_than:14d`) instead of fetching
  broadly and filtering yourself.
- To act on a specific message you usually need its **message id** (and often its
  **thread id**) — search first to get ids, then read/reply/modify by id.

## Replying vs. composing

- A **reply** must carry the original **thread id** so it lands in the same
  conversation; sending a fresh message with the same subject starts a *new* thread
  and reads as a mistake to the recipient.
- Prefer **creating a draft** over sending directly when the user hasn't explicitly
  said "send" — show the draft for confirmation first. Only send outright when the
  intent is unambiguous.
- Quote sparingly; don't paste the entire prior thread back into a reply body.

## Labels, not folders

- Gmail organizes by **labels** (a message can have many). "Archive" = remove the
  `INBOX` label; "mark read" = remove `UNREAD`. There are no folders to move between.

## Care

- Sending and deleting are **irreversible and user-visible**. Confirm recipients and
  content before sending; never bulk-delete or bulk-modify without explicit scope.
- Respect the user's voice for any drafted content; match the tone of the thread.
- Large mailboxes: page through results rather than requesting everything; summarize.
