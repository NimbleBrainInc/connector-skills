---
name: hubspot
description: >-
  How to use the HubSpot connector's tools — searching and listing contacts,
  creating contacts, deals, and companies, and batch-reading company records in
  the connected CRM. Use when the user works with HubSpot, their CRM, contacts,
  leads, companies, deals, pipeline, or sales records.
---

You are using the **HubSpot** connector (via Composio). Its tools act on the
connected HubSpot CRM. Tool slugs are uppercase and `HUBSPOT_`-prefixed (e.g.
`HUBSPOT_SEARCH_CONTACTS_BY_CRITERIA`, `HUBSPOT_CREATE_DEALS`). Apply this
guidance whenever you touch HubSpot.

## Search before you create

- Always **search for an existing record before creating one** — HubSpot does
  not dedupe for you, and a duplicate contact or company is user-visible and
  annoying to clean up. Search by email (contacts) or domain (companies) first.
- Search takes **filterGroups**, not free text: each filter is
  `{propertyName, operator, value}`. Filters *within* one group are **AND**ed;
  separate groups are **OR**ed. Operators include `EQ`, `NEQ`, `GT`, `LT`,
  `CONTAINS_TOKEN`, `IN`, `HAS_PROPERTY`. Limit: 5 groups, 6 filters each, 18
  total. Example: `[{filters:[{propertyName:"email",operator:"EQ",value:"a@x.com"}]}]`.
- Ask for the **properties** you actually need via a `properties` array;
  otherwise you get a thin default set. Use `HUBSPOT_LIST_CONTACTS_PAGE` for a
  simple paged scan, the search tool when you have criteria.

## Records, properties, and ids

- Every CRM object is a bag of typed **properties** plus an internal id
  (`hs_object_id`). To act on a specific record you usually need that id — get it
  from a search/list result rather than guessing.
- **Create** tools take a `properties` object. Contacts key on `email`;
  companies on `domain` (`name` is display only). Custom properties exist per
  portal — use the exact internal property name, not the UI label, or the value
  silently no-ops.

## Deals and associations

- Creating a deal (`HUBSPOT_CREATE_DEALS`) needs at minimum **`dealname`** and a
  **`dealstage`**; include **`pipeline`** if the portal has more than one (omit
  to use the default). `dealstage` and `pipeline` must be **internal ids**, not
  display labels — a label-as-value fails or lands the deal in the wrong stage.
- A bare deal is orphaned. **Associate** it with the relevant contact and/or
  company (by their internal ids) so it shows up on those records and in
  pipeline views.

## Batch reads

- `HUBSPOT_BATCH_READ_COMPANIES_BY_PROPERTIES` reads many companies in one call
  by id (or by a unique property) — prefer it over a read-per-record loop when
  enriching a list. Pass the `properties` you want returned.

## Care

- **Creates are immediately visible** to the sales team and feed workflows,
  scoring, and notifications. Confirm the data (especially owner, stage, amount)
  before writing; never bulk-create from an unverified list.
- **Search is eventually consistent** — a record you just created may not appear
  in search results for ~5-10s. If a read-after-write turns up empty, wait and
  retry rather than concluding it failed and re-creating it (that's how dupes
  happen).
- **Rate limits are tight** — the CRM search endpoint is ~5 requests/second per
  account, shared with everything else hitting it. Page through results and
  batch reads instead of firing many small calls; back off on 429.
- This is the company's system of record. Prefer creating and associating over
  editing or overwriting existing fields; confirm scope before any bulk change.
