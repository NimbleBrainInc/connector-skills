---
name: webflow
description: >-
  How to use the Webflow connector's tools — listing sites, reading CMS
  collection schemas, creating and updating CMS items, editing page SEO
  settings, and publishing. Use when the user works with Webflow, their
  website, CMS collections, blog posts, page metadata, or wants to publish
  changes to a Webflow site.
---

You are using the **Webflow** connector (remote MCP). Its tools act on the
connected Webflow workspace's sites and CMS. Apply this guidance whenever you
touch Webflow.

## Resolve ids first

- Almost every tool needs a **site_id**, **collection_id**, or **item_id** — not
  a human name. Start by listing sites, then drill down: site -> collections ->
  items. Don't guess ids; list to get them.
- Before writing CMS content, **read the collection schema** (the get-schema
  tool). It returns each field's exact `slug`/name and type (text, rich text,
  option, reference, multi-reference, image). Writing a wrong field name silently
  no-ops or errors.

## Tool families: Data API vs Designer API

- **Data API** tools (sites, collections, items, pages, assets, custom code) work
  immediately over OAuth — this is the common path for content and CMS work.
- **Designer API** tools (creating elements/styles/variables on the canvas)
  require the **Webflow Designer open in a browser with the MCP Bridge App
  running**. If the bridge isn't open these tools fail — tell the user to open the
  Designer and launch the app rather than retrying.

## Staged vs live — the key distinction

- Most write tools have a **staged** form (creates/edits a draft) and a separate
  **`*_live`** form (e.g. create-item-live) that **publishes immediately to the
  live site**. Default to the **staged** form unless the user explicitly says
  publish or "make it live."
- Items also carry an **`isDraft`** flag; a staged item with `isDraft: true` won't
  appear publicly even after a site publish.
- A separate **publish** step pushes staged changes live (per item, or a whole-site
  publish to chosen domains). Confirm the target domains before a site publish.

## Common workflows

- **Add a blog post / CMS row:** list sites -> list collections -> get schema ->
  create item (staged) with fields matching the schema -> review -> publish.
- **Bulk content:** use the bulk create/update tool for many items instead of one
  call each; it's faster and easier on rate limits.
- **SEO / metadata:** page tools update title, meta description, OG data, and slug.
  Note static page-content updates apply to **secondary locales only**, not the
  default locale.
- For a quick question about Webflow itself, the **ask-Webflow-AI** tool answers
  from Webflow's docs — use it for how-to, not for acting on the user's site.

## Gotchas

- **Rich-text links:** `href` attributes inside rich-text fields can be stripped on
  read-write round trips. After editing rich text, verify links survived.
- **Images:** the connector can attach assets that already exist in the asset
  library but **cannot generate or resize images**. Upload first, then reference.
- **Rate limits:** roughly 60 API requests/min, and **site publish is ~1 per
  minute**. On a 429, back off (honor `Retry-After`) rather than hammering; batch
  and pace writes.

## Care

- Publishing and deleting are **immediately user-visible on the live site and
  hard to undo**. Confirm scope before any `*_live` write, publish, or delete —
  never bulk-publish or bulk-delete without explicit user sign-off.
- Editing CMS items affects real site content. Prefer staged edits, show the user
  what will change, and publish only on explicit instruction.
