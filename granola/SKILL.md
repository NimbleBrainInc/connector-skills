---
name: granola
description: >-
  How to use the Granola connector's tools — finding the user's meetings, reading
  AI summaries and private notes, pulling verbatim transcripts, and answering
  natural-language questions about what was said or decided. Use when the user
  asks about Granola, meeting notes, transcripts, action items, decisions,
  follow-ups, or "what did we discuss / agree to" in a past call.
---

You are using the **Granola** connector (remote MCP). It reads the connected
user's Granola meeting notes, AI summaries, and transcripts. It is **read-only** —
there is no tool to create, edit, or delete anything. Apply this whenever you
touch Granola.

## Pick the right entry point

There are two ways in, and choosing wrong wastes calls:

- **Open-ended or natural-language question** ("what were my action items this
  week?", "what did we decide about pricing?") → use the **natural-language query
  tool**. It searches across notes and returns a synthesized answer. Prefer this
  for anything fuzzy or content-based.
- **Enumerate or browse** ("list my meetings", "what did I have last week",
  "meetings in the Sales folder") → use the **list tool** with a time range, then
  drill in by id. List the **folders** first if the user references one by name —
  you need its folder id to scope a listing.

## Drilling into a meeting

- The list tool returns **titles and metadata**, not content. To get the AI
  summary, private notes, and attendees, fetch detail **by meeting id** (UUIDs).
  The detail tool takes a small **batch of ids** (a handful at a time), so resolve
  the relevant meetings first rather than fetching everything.
- For **exact quotes or literal wording**, pull the **verbatim transcript** by
  meeting id. Use this only when the user needs what was *literally said* —
  transcripts are long; the AI summary is better for "what happened".

## Time ranges

- The list tool takes a **time range** — typically a named window
  (`this_week`, `last_week`, `last_30_days`) or a **custom** range with explicit
  start/end ISO dates. For "recent" questions, prefer a named window over a wide
  custom range; widen only if nothing turns up.

## Preserve citations

- The natural-language query tool returns answers with **inline numbered citation
  links** (e.g. `[[0]](url)`) that point back to the source notes. **Keep these
  links intact** in your reply to the user — they are how the user verifies a
  claim against the original meeting. Don't strip, renumber, or summarize them away.

## Care

- Meeting notes are **sensitive and often private** (client calls, internal
  strategy, personnel). Don't surface a meeting the user didn't ask about, and
  don't paste full transcripts when a summary answers the question.
- **Access depends on the user's plan.** Free plans typically see only the user's
  own notes from the **last ~30 days**; paid plans add notes shared with them and
  notes in private folders. If a meeting isn't found, it may be outside that window
  or not shared with this account — say so rather than inventing content.
- Confirm **which account** is connected (the account-info tool) if the user
  references a specific workspace or seems to have linked the wrong account.
- Don't fabricate decisions, attendees, or action items. If the notes don't say
  it, report that the notes don't cover it.
