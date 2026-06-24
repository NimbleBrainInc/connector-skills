---
name: microsoft-teams
description: >-
  How to use the Microsoft Teams connector's tools — reading and posting channel
  and chat messages, replying in threads, listing teams, channels, chats, members,
  and users, and creating chats, channels, and meetings. Use when the user works
  with Microsoft Teams, Teams channels, group chats, DMs, posting or replying to a
  message, @mentioning someone, or scheduling a Teams meeting.
---

You are using the **Microsoft Teams** connector (via Composio, backed by Microsoft
Graph). Its tools act as the connected user. Apply this guidance whenever you touch
Teams.

## Resolve ids before you act

Teams has no human-readable handles — every write needs opaque ids, and there are
two distinct surfaces:

- **Channel messages** live in a **team** and a **channel**. To post or read one you
  need a **team id** *and* a **channel id**. Start by listing the user's teams, then
  list that team's channels to get the channel id (the default channel is "General").
- **Chat messages** (1:1, group, meeting chats) live in a **chat** and need only a
  **chat id**. List the user's chats to resolve a person/group reference to a chat id.
- Don't guess ids. Resolve a name → id with a list call first, then act by id. To
  @mention or DM a person you'll also need their **user id** — list users to find it.

## Posting and replying

- **Post a channel message** and **send a chat message** create *new top-level*
  messages. Posting a channel message does **not** reply to an existing thread.
- To continue a thread, use the **reply** tool with the parent **message id** — a
  fresh top-level post with the same text reads as a mistake and won't thread.
- Read a channel thread by listing the channel's messages, then listing the
  **replies** for the message id you care about; a chat is a flat message list.

## Message body: text vs HTML

- A message body is **text** (default) or **html**. Set the content type to `html`
  when you need formatting (links, bold, lists).
- **@mentions require html**: put `<at id="0">Name</at>` in the content *and* a
  matching entry in the message's `mentions` array (same id, plus the target's user
  id). Plain-text "@name" does not notify anyone.
- Don't paste a whole prior thread back into a reply; keep messages scoped.

## Creating things

- **Create chat** with the member user ids (1:1 or group). Creating a 1:1 chat that
  already exists returns the existing chat rather than duplicating — safe to call.
- **Create channel** under a team; a private/shared channel needs members specified.
- **Create meeting** schedules a Teams online meeting and returns its join link —
  surface that link to the user.

## Gotchas

- **Pagination**: list calls (messages, chats, members, users) are paged. Page
  through and summarize rather than assuming the first page is the whole story.
- Listing channel messages needs only team id + channel id (no chat id); mixing up
  the channel vs chat id spaces is the most common error — keep them straight.
- Permissions are per-connection: if a team/channel/chat isn't found, the connected
  user may simply not be a member, not that it's missing. Say so rather than inventing.
- A connection limited to read scopes will fail writes; don't retry a 403 as if it
  were transient.

## Care

- Posts and replies are **immediately visible** to everyone in the channel or chat,
  and notify members. Confirm the target team/channel/chat (and recipients for a new
  chat) before sending; never bulk-post.
- Prefer **showing the user the drafted message** for confirmation unless the intent
  to send is unambiguous. Match the tone of the conversation.
- Deleting a channel message is a **soft-delete** (recoverable), but still removes it
  from everyone's view — treat it as destructive and confirm first. Don't delete or
  bulk-modify without explicit scope.
