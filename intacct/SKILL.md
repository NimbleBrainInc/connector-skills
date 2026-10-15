---
name: intacct
description: >-
  How to use the Sage Intacct connector's tools — querying GL, AP, AR, vendors,
  customers, bills, invoices, and payments on the connected Intacct company,
  including totals, top-N rankings, period P&L, and budget-to-actual. Use when the
  user asks about Sage Intacct, accounting data, vendors, spend, bills, AP, AR,
  invoices, payments, budgets, or the general ledger.
---

You are using the **Sage Intacct** connector (remote MCP) to read the connected
Intacct company's accounting data.

## Tools and the order to use them

- **`listAvailableModels`**: find the object to query. Object names take the form
  `<component>/<object>`, e.g. `accounts-payable/bill`, `accounts-payable/vendor`.
  Custom objects are `platform-apps/nsp::<object-name>`. It takes `filter` (a regex
  on `component/object`) and `includeDescriptions`. Pass `includeDescriptions: false`
  unless you need them: the full list with descriptions is long enough to be cut off.
  It lists only the components this connection can read. A dimension's name is still
  on the line that carries it (`dimensions.location.name`) when the dimension's own
  object isn't listed.
- **`getModelHints`** takes **`objectName`** (not `model` or `object`). Call it once
  per object before the first query on it. Sage records object-specific rules there
  (which state means "unpaid", which date field to default to, which amount field
  to sum), and those rules override what the field names suggest.
- **`getModelDefinition`** lists an object's fields, relationships, and types. When
  the hints come back as "No specific hints available", call it before the first
  query on that object. Field names don't carry across objects: not every object
  has `name` (`general-ledger/account-group` uses `title`), and the date field
  differs (`invoiceDate` on an AR invoice, `bill.postingDate` on a bill line,
  `journalEntry.postingDate` on a journal line).
- **`executeQuery`** runs the query. Its object argument is **`object`**; `fields`,
  `filters`, and `orderBy` are arrays.

When a call is rejected for an unknown argument, the error lists the valid ones.
Use those names; don't guess another.

When you run queries in parallel, read hints (and, where there are none, the
definition) for every object new to the conversation in an earlier step, not in
the same batch as its first query.

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
  use `filterExpression` only when you need OR or grouping. Its numbers are the
  filters' positions in `filters`, from 1: `"1 and (2 or 3)"` needs three filters.
  Date macros such as `currentMonth` go with `$eq`; they are not `$between` bounds.
- `size` is at most 100, and `start` begins at 1. A response with `hasMore: true`
  gives the next `start` in `next`. For a top-N question with N up to 100, sort and
  set `size`; past 100, page. An aggregate with `hasMore: true` is missing groups:
  page to the end before you add its rows up.
- If you're unsure a query is valid, run it with `dryRun: true`. That validates the
  fields, filters, orderBy, and paging without running the query.

## Totaling journal lines

Sage's hints for `general-ledger/journal-entry-line` cover listing lines, not
totaling them. Before you report a period total, a P&L, or budget-to-actual:

- **Break the period down by source first.** Group by `journalEntry.moduleName`,
  `journalEntry.glJournal.id`, and `txnType` with `sum:baseAmount`. A consolidation
  or rollup journal (seen as module `45.ATLAS`) restates postings that are already
  on their source journals' lines, so summing every line counts them twice. Exclude
  it with `$ne` on `journalEntry.moduleName`, and say what you excluded.
- **Amounts are unsigned; `txnType` carries the sign.** `sum:baseAmount` without
  `txnType` as a group key adds debits and credits together. Net each account as
  debits minus credits. Expense accounts can carry credits (allocations,
  reversals), so net every account, not only credit-normal ones.
- **Tie one figure to its subledger before trusting the rest.** For example, an
  expense account in the GL against `accounts-payable/bill-line` filtered on the
  same `glAccount.id` and `bill.postingDate` range. A GL total that is a clean
  multiple of the subledger's means a journal is counted twice.
- **Account groups are many-to-many.** `general-ledger/account-group-map` lists an
  account once per group it belongs to, so joining amounts through it multiplies
  them. Use one designated group per account, or the account's `category`.

## Budgets

Sage has no hints for `general-ledger/budget-detail`.

- A row is one budget's amount for an account, a dimension set, and a reporting
  period. Filter on `budget.id` and on `reportingPeriod.startDate`;
  `reportingPeriod.id` is the period's label ("Month Ended …").
- `amount` is unsigned. Classify each account first (`accountType` and
  `normalBalance` on `general-ledger/account`), then net revenue against expense.
- A company keeps several budgets (`general-ledger/budget`, where `isDefault` marks
  the default), and two can hold identical figures. Name the budget every figure
  came from.

## Care

- This is a company's books. State what a figure covers: the object, the date
  field, the period, and any state filter. For example, "all-time bill totals by
  vendor, all bill states". When the user didn't name a period, say so and offer
  to narrow it.
- A failed query returns Intacct's message and an error ID (`REST-…`). The message
  usually names the fix: apply it. Never resend an unchanged query. The exception
  is "Internal error processing query execution. Please try again.": retry it once
  with a smaller `size`.
- `GW-0011` is an authorization problem, not a query problem: the Intacct client
  isn't active yet. Tell the user to reconnect Sage Intacct later; don't rewrite
  the query.
