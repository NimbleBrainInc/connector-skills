---
name: zoho
description: >-
  How to use the Zoho CRM connector's tools — finding, reading, creating, and
  updating records across modules (Leads, Contacts, Accounts, Deals), searching
  with criteria, converting leads, and discovering module/field metadata. Use when
  the user works with Zoho CRM, leads, contacts, accounts, deals, pipeline, the
  sales pipeline, or CRM records.
---

You are using the **Zoho CRM** connector (via Composio). Its tools act on the
connected org's CRM. Apply this guidance whenever you touch Zoho.

## Everything is module + API name

- Zoho CRM is organized into **modules**: standard ones use PascalCase API names —
  `Leads`, `Contacts`, `Accounts`, `Deals`, `Tasks`, `Events`, `Calls`, `Products`,
  `Quotes`, `Invoices`, `Campaigns`, `Cases`, `Notes`. Do **not** use `Activities` —
  substitute `Tasks`, `Events`, or `Calls`. Custom modules use their own exact API name.
- Almost every tool takes `module_api_name`. When unsure which module or what a field
  is called, **discover first**: list modules to get valid API names, then fetch a
  module's field metadata to learn the exact field **API names** (not display labels)
  and types before you read, filter, or write.
- Fields are referenced by API name (`Last_Name`, `Email`, `Stage`, `Deal_Name`),
  which often differs from the UI label. A wrong field name errors or silently drops.

## Find the target first

- To act on a specific record you need its **record id**. Resolve a human reference
  ("the Acme deal") to an id by **searching** the right module before updating.
- **Search criteria syntax** is `(Field:operator:value)`, combined with `and`/`or`:
  `((Stage:equals:Negotiation)and(Amount:greater_than:10000))`. Max ~10 criteria.
  - Supported operators: `equals`, `not_equal`, `starts_with`, `in`, `between`,
    `greater_than`, `greater_equal`, `less_than`, `less_equal`.
  - **`contains` is not supported** and errors. For text fields, `equals` already
    behaves like a substring/contains match (`Company:equals:ABC` also matches
    "ABC Inc"), so use `equals` for partial text instead of inventing `contains`.
  - Datetimes need a timezone offset: `2026-01-01T00:00:00+00:00`.
- Use **get records** to list a module's records when you don't have criteria, and
  **get related records** to walk a relationship (a Contact's Deals, an Account's
  Contacts) rather than searching and joining yourself.

## Creating and updating

- Create requires the target module plus its **mandatory fields** — e.g. `Leads`
  and `Contacts` require `Last_Name`; `Deals` require `Deal_Name` and `Stage`. Check
  module fields if a create is rejected for a missing required field.
- Update is **partial**: send the record `id` plus only the fields you're changing;
  unspecified fields are left untouched. Don't refetch-and-resend the whole record.
- New records default to the **authenticated user** as owner unless you set one.

## Converting leads

- **Lead conversion** turns a Lead into a Contact (and Account, optionally a Deal).
  It is **one-directional and effectively irreversible** — the Lead is consumed.
  Confirm intent and the deal details before converting; don't convert speculatively.

## Care

- Writes hit a live, shared CRM that the sales team sees immediately. Confirm the
  target record (by id) before updating, and confirm scope before any bulk write —
  some create/update tools accept many records at once and report **partial success**,
  so check the per-record result, not just an overall OK.
- This connector exposes no delete tool, but updates and lead conversion still change
  real pipeline data — treat them as consequential, not throwaway.
- Large modules: paginate (≤200 per page; use the page token for deep paging) and
  summarize rather than pulling everything. `equals`-as-contains can over-match on
  text — verify you hit the intended record before acting on it.
