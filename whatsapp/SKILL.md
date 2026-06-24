---
name: whatsapp
description: >-
  How to use the WhatsApp connector's tools — sending text, media, location,
  contacts, and interactive button/list messages over the WhatsApp Business
  Cloud API, plus managing approved message templates. Use when the user wants to
  message a customer on WhatsApp, send a WhatsApp notification or template,
  reply to a chat, send an image/PDF/location, or check template status.
---

You are using the **WhatsApp** connector (via Composio). Its tools send messages
from a connected **WhatsApp Business** account (not a personal WhatsApp) over the
Business Cloud API. Apply this guidance whenever you touch WhatsApp.

## The 24-hour window decides which tool to use

This is the most important rule and the cause of most failures:

- A business may send **free-form** messages (text, media, location, contacts,
  interactive) only inside the **24-hour customer-service window** — the 24 hours
  after the recipient's *last inbound message* to the business. Outside it,
  free-form sends are **rejected by Meta**.
- To message someone who hasn't written in the last 24 hours (or ever),
  you **must** use `WHATSAPP_SEND_TEMPLATE_MESSAGE` with a **pre-approved**
  template — that's the only thing allowed to open a conversation.
- When unsure whether the window is open, prefer a template, or confirm with the
  user that the recipient messaged recently. Don't blast a free-form text and let
  it silently fail.

## Recipients and phone numbers

- Recipient numbers are **international format with no `+` and no spaces/dashes**
  (e.g. `14155551234`). Strip formatting before sending.
- Sends also need the **sender** `phone_number_id` (the business number's id, not
  the digits). Resolve it once with `WHATSAPP_GET_PHONE_NUMBERS` and reuse it;
  don't guess it.

## Sending free-form (window open)

- `WHATSAPP_SEND_MESSAGE` — plain text (`text`, up to 4096 chars).
- `WHATSAPP_SEND_MEDIA` — image/video/audio/document/sticker by **public HTTPS
  `link`** + `media_type`; `caption` is allowed except on audio/stickers. For a
  file the user uploaded, first `WHATSAPP_UPLOAD_MEDIA`, then send by the returned
  media id (the by-id send variant) rather than re-hosting it.
- `WHATSAPP_SEND_INTERACTIVE_BUTTONS` (max **3** reply buttons),
  `WHATSAPP_SEND_INTERACTIVE_LIST`, `WHATSAPP_SEND_LOCATION`,
  `WHATSAPP_SEND_CONTACTS` for richer replies — all still gated by the 24h window.

## Templates (the only way to initiate or reach outside the window)

- A template send needs `template_name`, `language_code` (e.g. `en_US`), and a
  `components`/parameters payload supplying the template's variables **in order**
  — a mismatch between the variables you pass and the template's defined ones is a
  hard error. Inspect the real template first with `WHATSAPP_GET_MESSAGE_TEMPLATES`
  / `WHATSAPP_GET_TEMPLATE_STATUS`; don't invent a name or guess its variables.
- Templates only work once Meta has **APPROVED** them. `WHATSAPP_CREATE_MESSAGE_TEMPLATE`
  submits for review (status starts `PENDING`); it is **not** usable until approved,
  which can take time. Don't create-then-immediately-send.

## Care

- Sent messages are **delivered to a real person and cannot be unsent** — confirm
  the recipient number, the body, and (for templates) the variable values before
  sending. Never bulk-send without explicit, scoped confirmation.
- Free-form sends outside the 24h window fail silently from the user's view —
  surface the window rule rather than retrying blindly.
- Deleting a template is **irreversible**, and a deleted template name is **blocked
  for reuse for ~30 days** — never delete one without explicit instruction.
- This is the business's voice to its customers: match tone, avoid spammy framing,
  and respect opt-in/marketing rules (templates are categorized; marketing sends
  to non-opted-in users are a policy violation).
