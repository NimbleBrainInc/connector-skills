---
name: intacct
description: >-
  How to use the Sage Intacct connector's tools — querying GL, AP, AR, vendors,
  customers, bills, invoices, and payments on the connected Intacct company,
  including totals, top-N rankings, and other aggregates. Use when the user asks
  about Sage Intacct, accounting data, vendors, spend, bills, AP, AR, invoices,
  payments, or the general ledger.
---

You are using the **Sage Intacct** connector (remote MCP) to read the connected
Intacct company's accounting data.

## Tools and the order to use them

- **`listAvailableModels`**: find the object to query. Object names take the form
  `<component>/<object>`, e.g. `accounts-payable/bill`, `accounts-payable/vendor`.
  Custom objects are `platform-apps/nsp::<object-name>`.
- **`getModelHints`** takes **`objectName`** (not `model` or `object`). Call it once
  per object before the first query on it. Sage records object-specific rules there
  (which state means "unpaid", which date field to default to, which amount field
  to sum), and those rules override what the field names suggest.
- **`getModelDefinition`** lists an object's fields, relationships, and types. Use
  it to confirm a field name rather than guessing one.
- **`executeQuery`** runs the query. Its object argument is **`object`**; `fields`,
  `filters`, and `orderBy` are arrays.

When a call is rejected for an unknown argument, the error lists the valid ones.
Use those names; don't guess another.

## executeQuery has no `groupBy`

The arguments are `object`, `fields`, `filters`, `filterExpression`,
`filterParameters`, `orderBy`, `start`, `size`, `useStructuredContent`, and `dryRun`.
There is no `groupBy`, `having`, or `limit`.

**Aggregates group implicitly.** Put an aggregate as `function:field` in `fields`
(`sum`, `count`, `avg`, `min`, `max`). Every plain field in the same list becomes a
group key:

```json
{
  "object": "accounts-payable/bill",
  "fields": ["vendor.id", "vendor.name", "sum:totalBaseAmount"],
  "orderBy": [{"sum:totalBaseAmount": "desc"}],
  "size": 10
}
```

That returns one row per vendor, and `totalCount` counts vendors, not bills.

- **Sort an aggregate by its aggregate key.** Write
  `{"sum:totalBaseAmount": "desc"}`, not `{"totalBaseAmount": "desc"}`. The bare
  field name fails with REST-1138.
- `count` needs a field: `count:id`, never `count`.
- Some objects add currency as an extra group key on monetary aggregates, so a
  multi-currency company gets one row per currency even when you asked for a single
  total. The object's hints say when this applies. Don't cap such a total with
  `size: 1`, and never add amounts in different currencies together. A top-N ranking
  lists a vendor once per currency, so N rows can hold fewer than N vendors: rank
  within each currency.

## Filters, paging, and checking

- A filter is one operator object, e.g. `{"$eq": {"createdDate": "currentQuarter"}}`
  or `{"$gt": {"totalBaseAmountDue": "0"}}`. Filters combine with AND by default, so
  use `filterExpression` only when you need OR or grouping. Date macros such as
  `currentMonth` go with `$eq`; they are not `$between` bounds.
- `size` is at most 100, and `start` begins at 1. A response with `hasMore: true`
  gives the next `start` in `next`. For a top-N question with N up to 100, sort and
  set `size`; past 100, page.
- If you're unsure a query is valid, run it with `dryRun: true`. That validates the
  fields, filters, orderBy, and paging without running the query.

## Care

- This is a company's books. State what a figure covers: the object, the date
  field, the period, and any state filter. For example, "all-time bill totals by
  vendor, all bill states". When the user didn't name a period, say so and offer
  to narrow it.
- A failed query returns Intacct's message and an error ID (`REST-…`). The message
  usually names the fix: apply it. Never resend an unchanged query.
- `GW-0011` is an authorization problem, not a query problem: the Intacct client
  isn't active yet. Tell the user to reconnect Sage Intacct later; don't rewrite
  the query.
