---
name: slack
description: >-
  How to use the Slack connector's tools — sending and scheduling messages,
  replying in threads, reading channel history, searching the workspace, adding
  reactions, and resolving channels and users. Use when the user works with
  Slack, channels, DMs, threads, messages, reactions, or wants to post or look
  something up in a workspace.
---

You are using the **Slack** connector (via Composio). Its tools act on the
connected workspace as the authorized user/bot. Apply this guidance whenever you
touch Slack.

## Resolve the target first — use IDs, not names

- Most tools take a `channel`. Prefer a **channel ID** (`C…` public, `G…`
  private, `D…` DM) over a `#name`. Name resolution is unreliable and fails
  outright on org-wide tokens. Resolve a human reference with the
  find-channels tool (`SLACK_FIND_CHANNELS`, by query) or list-channels before
  posting.
- To DM or @-mention a person, resolve them to a **user ID** first —
  find-user-by-email (`SLACK_FIND_USER_BY_EMAIL_ADDRESS`) or list-users. Mention
  in text as `<@U012ABC>`, not `@name`.

## Posting and replying

- Send (`SLACK_SEND_MESSAGE`) requires `channel` **and** at least one content
  field (`text`, `markdown_text`, `blocks`, or `attachments`) — omitting all
  errors with `no_text`. Keep a message body under ~4000 chars; split or
  thread longer content.
- A **reply** must pass `thread_ts` = the **parent message's `ts`** (the
  original, never a reply's `ts`). Timestamps are strings with fractional
  precision (`"1700000000.123456"`); pass them exactly as returned.
- Slack uses **mrkdwn**, not full Markdown: `*bold*`, `_italic_`, `~strike~`,
  `> quote`, `` `code` ``. Links are `<url|label>`. There are no `#` headings.
- Edit with the update-message tool and delete with the delete tool — both need
  the exact `ts` + `channel`. To react, add-reaction
  (`SLACK_ADD_REACTION_TO_AN_ITEM`) takes the emoji `name` (no colons),
  `channel`, and message `timestamp`.

## Reading and searching

- Channel history (`SLACK_FETCH_CONVERSATION_HISTORY`) returns the **main
  timeline only** — threaded replies are not included. To read a thread, call
  fetch-message-thread with the channel and the **parent `ts`**. Page with
  `cursor`; bound with `oldest`/`latest` rather than pulling everything.
- Search (`SLACK_SEARCH_MESSAGES`) is workspace-wide and uses **Slack search
  operators**, not plain keywords: `in:#channel`, `from:@user`,
  `before:YYYY-MM-DD`, `after:YYYY-MM-DD`, `has:link`, `has::emoji:`. Combine
  them (`deploy in:#eng from:@alice after:2026-01-01`) instead of fetching broad
  and filtering yourself. Search only sees content the connected identity can
  access.

## Care

- Posting is **immediate and visible to everyone in the channel** — there is no
  draft state. Confirm the channel (right `C…` id, public vs. private) and the
  content before sending, especially for @here/@channel or cross-channel posts.
- **Delete is irreversible**, and only the original author/bot can delete its own
  messages. Never bulk-delete or bulk-react without explicit scope.
- Don't @-mention broadly (`@channel`, `@here`) unless asked — it notifies
  everyone. Scheduled messages (`SLACK_SCHEDULE_MESSAGE`, `post_at` = future
  Unix time) still fire later; say so when you set one.
- Honor rate limits (~1 message/sec; HTTP 429 carries `Retry-After`). When in
  doubt about which channel, ask rather than guessing — a wrong-channel post is
  user-visible and hard to undo cleanly.
