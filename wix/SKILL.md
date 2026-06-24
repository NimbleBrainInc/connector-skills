---
name: wix
description: >-
  How to use the Wix connector's tools — listing and managing Wix sites,
  calling Wix business APIs (Stores, Bookings, Blog, Events, Members, CMS data
  collections, contacts), and searching the Wix REST/SDK/Headless docs. Use when
  the user works with Wix, their website, online store, products, bookings,
  blog posts, events, members, contacts, or wants to create or publish a site.
---

You are using the **Wix** connector (remote MCP). Its tools act on the connected
Wix account and its sites, and can search Wix's developer documentation. Apply
this guidance whenever you touch Wix.

## Find the site first

- A Wix account holds **many sites**. Resolve the user's reference ("my store",
  "Party Costume Store") to a concrete site by calling **ListWixSites** before any
  action — don't guess a site id. Most action tools need the selected site.
- If several sites match, confirm which one before acting; if none match, say so
  rather than acting on the wrong site.

## Acting on a site: docs-then-API

- The connector does **not** expose one tool per Wix feature. Instead you pick a
  **Wix business solution** (Stores, Bookings, Blog, Events, Members, CMS/Data,
  Contacts/Inbox) and an API **method**, then invoke it via the call-site-API tool.
- Many business solutions share method names (Stores, Restaurants, and Events each
  have an Orders API). **Name the solution explicitly** in the call so the right
  API is used.
- When you're unsure of the exact method, request/response shape, or field names,
  search the docs first (**SearchWixRESTDocumentation** / **SearchWixSDKDocumentation**
  / **SearchWixHeadlessDocumentation**), then fetch the method schema
  (**ReadFullDocsMethodSchema**) or the full article (**ReadFullDocsArticle**) before
  building the call. This prevents wrong-field no-ops.

## Common workflows

- **Stores discount**: there's no "set discount" — model it as a **coupon** (e.g.
  create a coupon for X% off, time-boxed). Look up the Coupons API schema first.
- **CMS data**: create/query **data collections** (rows are typed columns); honor
  the existing column schema and mark PII fields when asked.
- **Blog / Events / Bookings**: create then **publish/confirm** as a separate step —
  a created blog post or site is a draft until explicitly published.
- **Contacts / Inbox email**: search for the contact first; create it only if absent.

## Site management

- **ManageWixSite** does account-level actions — **create**, **install an app on**,
  or **publish** a site. **ListWixSites** enumerates them.

## Documentation tools

- The `Search*Documentation` tools and **ReadFullDocs\*** are for **writing Wix code**
  and learning API shapes (REST, SDK, Headless, Design System, Build Apps). Use them
  to inform code or API calls — they don't change anything on the site.

## Care

- API calls and site management hit the **live site and real customers**:
  publishing, applying a coupon, sending an inbox email, or creating/publishing blog
  posts are immediately visible and some are hard to undo. Confirm the **site**,
  scope, and content before a write — never bulk-create or bulk-email without
  explicit scope.
- Treat **create-site / install-app / publish** as high-impact; state what you're
  about to do and on which site, and prefer a draft over publishing when intent is
  ambiguous.
- **SupportAndFeedback** sends a message to Wix — only use it when the user asks to
  send feedback.
