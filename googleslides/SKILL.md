---
name: googleslides
description: >-
  How to use the Google Slides connector's tools — creating presentations,
  generating slides from Markdown, copying from a template, reading slide
  structure, batch-editing content, and rendering page thumbnails. Use when the
  user works with Google Slides, slide decks, presentations, slideshows, a pitch
  deck, or wants to build or edit slides.
---

You are using the **Google Slides** connector (via Composio). Its tools act on the
connected user's Google Slides, addressed by **presentation id**. Apply this
guidance whenever you touch Slides.

## Pick the right creation path

- **Whole deck from text** → `GOOGLESLIDES_CREATE_SLIDES_MARKDOWN` (needs `title` +
  `markdown_text`). Best for "make me a deck about X" — `#`/`##` headings become
  slides, bullets become body. Fastest path; skip the raw API.
- **Blank deck** → `GOOGLESLIDES_CREATE_PRESENTATION` (`title`), then edit.
- **Branded/styled deck** → `GOOGLESLIDES_PRESENTATIONS_COPY_FROM_TEMPLATE`
  (`template_presentation_id`) to clone an existing styled deck, then fill it in.
  Prefer this over building styling by hand — it preserves theme, fonts, masters.

## Read before you edit

- To change an existing deck you must address slides and shapes by **object id**.
  These ids are *not* guessable — call `GOOGLESLIDES_PRESENTATIONS_GET`
  (`presentationId`, or `presentationName` to resolve by name first) to get the
  slide list and each page element's `objectId`. Use
  `GOOGLESLIDES_PRESENTATIONS_PAGES_GET` for one page's detail.
- Map the human reference ("the title slide", "slide 3") to a concrete
  `pageObjectId` / element id from that read. Don't invent ids.

## Editing: Markdown vs. raw requests

- `GOOGLESLIDES_PRESENTATIONS_BATCH_UPDATE` (`presentationId`) takes **either**
  `markdown_text` (simple content append) **or** a `requests` array of raw Google
  Slides API operations (`createSlide`, `createShape`, `insertText`,
  `deleteText`, `replaceAllText`, `updateTextStyle`, `deleteObject`, …).
- For targeted edits ("change the heading on slide 2"), use the `requests` array
  with the object ids you read. For bulk find/replace across the deck,
  `replaceAllText` is one request and avoids per-element ids.
- All subrequests in one call apply **atomically** — they all succeed or all fail.
  Batch related edits into a single call rather than many round-trips; it's faster
  and avoids half-applied state.

## Thumbnails

- `GOOGLESLIDES_GET_PAGE_THUMBNAIL2` (`presentationId`, `pageObjectId`) renders a
  page to an image — use it to *show* the user a slide or to verify an edit landed.
  Prefer the v2 tool over the deprecated thumbnail action.

## Care

- Batch updates (especially `deleteObject`, `deleteText`, `replaceAllText`) are
  **irreversible** — there's no undo via the API. Read the deck first, confirm the
  target, and scope `replaceAllText` tightly so it doesn't rewrite unintended text.
- Edits are immediately visible to anyone the presentation is shared with. Confirm
  you're editing the right deck (resolve the id) before writing.
- Google enforces per-minute and daily quotas; if a large edit errors on rate
  limits, batch more per call and back off rather than hammering.
- A presentation that isn't found may simply not be shared with the connector —
  say so instead of guessing an id.
