---
name: zoom
description: >-
  How to use the Zoom connector's tools — listing scheduled meetings, looking up
  past meeting instances and participants, fetching cloud recordings and AI
  meeting summaries, and finding company contacts. Use when the user works with
  Zoom, meetings, calls, recordings, transcripts, meeting summaries, attendees,
  or who was on a call.
---

You are using the **Zoom** connector (via Composio). Its tools act on the connected
Zoom account, scoped to the authenticated user. Apply this guidance whenever you
touch Zoom. Tool names are prefixed `ZOOM_` (e.g. `ZOOM_LIST_MEETINGS`).

## "me" and the user scope

- Most read tools take a **user id**; pass the literal string **`me`** to mean the
  connected user. `ZOOM_GET_USER` resolves a profile by id, email, or `me`.
- This is a *user-level* connection: you see the connected user's meetings and
  recordings, not the whole account, unless the account explicitly grants more.

## Find the meeting first — and mind id vs. UUID

- Zoom has **two different keys** for a meeting, and the tools are picky about which:
  - **Meeting id** — the numeric id (the "meeting number"). Use it for *scheduled*
    meetings: `ZOOM_GET_A_MEETING`, `ZOOM_GET_MEETING_RECORDINGS`.
  - **Meeting UUID** — a per-occurrence opaque token. Use it for *past* meeting data:
    `ZOOM_GET_PAST_MEETING_PARTICIPANTS`, `ZOOM_GET_A_MEETING_SUMMARY`.
- A recurring meeting has **one id but many UUIDs** (one per occurrence). To get
  data for a specific past run, list its instances with
  `ZOOM_LIST_PAST_MEETING_INSTANCES` first, then act on the right UUID.
- **UUID gotcha:** if a UUID starts with `/` or contains `//`, it must be
  **double URL-encoded** in the request. Pass the raw UUID exactly as returned and
  let the tool handle encoding; never hand-trim or partially escape it.

## Common workflows

- *"What's on my Zoom calendar?"* → `ZOOM_LIST_MEETINGS` for `me`. Note it lists
  **unexpired scheduled** meetings and **excludes instant** meetings.
- *"Who was on the X call?"* → resolve the occurrence's **UUID**
  (`ZOOM_LIST_PAST_MEETING_INSTANCES`), then `ZOOM_GET_PAST_MEETING_PARTICIPANTS`.
- *"Summarize the X meeting"* → `ZOOM_GET_A_MEETING_SUMMARY` by UUID; this returns
  Zoom's **AI Companion** summary, not a transcript you generate.
- *"Get the recording"* → `ZOOM_GET_MEETING_RECORDINGS` for one meeting, or
  `ZOOM_LIST_ALL_RECORDINGS` to sweep a user's cloud recordings over a date range.
  Recordings come back with a **`download_url`**; surface it rather than trying to
  stream the media through the tool.
- *"Find <person>'s contact"* → `ZOOM_SEARCH_COMPANY_CONTACTS` matches contacts
  within the connected org's directory, not arbitrary external people.

## Paging and ranges

- List tools paginate with **`next_page_token`**: an empty token means the last
  page; otherwise re-call with the *same* filters plus the token. The token
  **expires in ~15 minutes**, so don't stash it across a long pause.
- Respect **`page_size`** caps (commonly 30, up to 300 for light endpoints) instead
  of asking for everything at once; loop pages and summarize.
- `ZOOM_LIST_ALL_RECORDINGS` only spans roughly a **one-month window** per call
  (`from`/`to`). For a wider range, walk it month by month rather than one giant ask.

## Care

- These are mostly **read** tools — favor them for reporting on meetings, attendance,
  and recordings. There's no need to confirm before a read.
- **Plan/feature gating is real:** recordings, AI summaries, and past-participant
  data require a paid Zoom plan (and AI Companion for summaries); a clean-but-empty
  result often means "not enabled" or "no participants," not "doesn't exist" — say
  that instead of inventing data.
- **Recordings and summaries are sensitive.** Treat `download_url`s and transcripts
  as confidential; don't re-share them beyond the user's request, and never paste a
  full recording link into an unrelated context.
- If a meeting isn't found, it may have **expired, never occurred, or belong to
  another user** rather than being deleted — distinguish those instead of guessing.
