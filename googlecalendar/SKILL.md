---
name: googlecalendar
description: >-
  How to use the Google Calendar connector's tools — finding events, creating
  and editing them, inviting attendees, and checking availability. Use when the
  user works with their calendar, schedule, meetings, events, invites, agenda,
  availability, free/busy, or asks to book or reschedule a time.
---

You are using the **Google Calendar** connector (via Composio). Its tools act on the
connected user's calendars. Apply this guidance whenever you touch the calendar.

## Anchor to "now" first

- The model has no reliable clock. Before resolving any relative time ("tomorrow",
  "next Tuesday", "this afternoon"), call the **get-current-date-time** tool to fix
  the present moment and the user's timezone. Don't guess today's date.
- Times are **RFC3339** with an explicit offset (`2026-06-24T14:00:00-10:00`) or `Z`
  for UTC. Always pair a `start`/`end` with a `timezone` (IANA, e.g.
  `Pacific/Honolulu`) — a naked datetime is ambiguous and lands in the wrong hour.

## Find / resolve before you act

- To edit, move, or delete an event you need its **event id**. Search first to get
  the id — use the find-event tool (free-text `query` over summary, description,
  location, attendees, with optional `time_min`/`time_max`) or the events-list tool
  (a calendar's events in a time window). Don't invent ids.
- `calendar_id` defaults to **`primary`** (the user's main calendar). For any other
  calendar, resolve the real id with the list-calendars tool — arbitrary email
  addresses are not valid calendar ids.
- Set `single_events: true` when you need individual occurrences of a recurring
  series rather than the master event.

## Creating and editing events

- Creating an event needs at minimum a **summary**, a **start**, and a **timezone**;
  give duration via an end time (or duration fields). Pass **attendees** as email
  strings or `{email, optional}` objects — plain names won't work.
- The natural-language **quick-add** tool ("lunch with Sam Friday 1pm") is fine for a
  quick personal hold, but it can't set attendees or recurrence — use the full
  create tool when those matter.
- **Update vs. patch:** the update (PUT) tool is a *full replacement* — fields you
  omit get cleared, so you must resend everything you want to keep. Prefer the
  **patch** tool for partial edits (change just the time, add one attendee). Note
  that array fields like attendees/recurrence are still replaced wholesale when
  provided — read the current list, modify, then send the full set.
- To remove one guest, prefer the remove-attendee tool over patching the whole list.

## Invites and notifications

- Creating, updating, deleting, or changing attendees **emails the guests**. This is
  real, user-visible mail — not a dry run. Editing tools take `send_updates`
  (`all` / `externalOnly` / `none`); when a change is cosmetic or you're still
  iterating, set `none` so you don't spam attendees with churn.
- Create may attach a Google Meet link by default — only request one when a video
  call is intended.

## Availability / scheduling

- To propose a meeting time, use the find-free-slots tool over the relevant
  calendars and time range (it returns busy intervals; you compute gaps that fit the
  desired duration). Prefer it over fetching every event and reasoning by hand.
- Honor the user's working hours and timezone when proposing slots; don't suggest
  3am. Offer a few concrete options rather than auto-booking.

## Care

- **Deleting an event is permanent** and notifies attendees — confirm the exact event
  (title + time) before deleting, and never bulk-delete without explicit scope. To
  cancel softly, consider patching `status: cancelled` instead.
- Confirm before booking, moving, or sending invites on the user's behalf when intent
  is at all ambiguous — surface the details and let them approve.
- Mind shared and secondary calendars: a write may be visible to colleagues. Check
  which `calendar_id` you're acting on before writing.
