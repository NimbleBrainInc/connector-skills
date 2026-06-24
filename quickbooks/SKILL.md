---
name: quickbooks
description: >-
  How to use the QuickBooks connector's tools — invoices, customers, payments,
  bills, estimates, and financial reports (profit & loss, balance sheet) on the
  connected QuickBooks Online company. Use when the user works with QuickBooks,
  QBO, accounting, bookkeeping, invoicing, billing, customers, payments, vendors,
  or financial reports.
---

You are using the **QuickBooks** connector (via Composio). Its tools act on the
connected QuickBooks Online company file. This is a **financial system of record** —
apply this guidance whenever you touch QuickBooks.

## Find the target first

- Most write/read-by-id tools need the entity's **QuickBooks id** (e.g. an
  invoice id, customer id). Resolve a human reference ("Acme's last invoice")
  into an id **before** acting — list/query first, then read or update by id.
- `QUICKBOOKS_QUERY_ENTITIES` runs a QBO query against any entity. The syntax is
  **SQL-like but proprietary**: `SELECT * FROM Invoice WHERE CustomerRef = '58'
  STARTPOSITION 1 MAXRESULTS 100`. Notable limits — projections aren't supported
  (it always returns all fields), and there's **no `OR`, no `JOIN`, no `GROUP BY`**.
  Use `=, <, >, <=, >=, IN, LIKE` (with `%` wildcard) only. Page with
  `STARTPOSITION`/`MAXRESULTS` (max **1000** rows per page).
- For invoices specifically, `QUICKBOOKS_LIST_INVOICES` is the simple lister;
  reach for `QUICKBOOKS_QUERY_ENTITIES` when you need filtering (by customer,
  date, balance) or to query an entity that has no dedicated list tool.

## Transactions reference other entities by id

- Invoices, payments, bills, and estimates don't embed names — they carry
  **reference ids** (`CustomerRef`, `ItemRef`, `VendorRef`, `AccountRef`). Before
  `QUICKBOOKS_CREATE_INVOICE` or `QUICKBOOKS_CREATE_BILL`, resolve/create the
  customer (`QUICKBOOKS_CREATE_CUSTOMER` / `QUICKBOOKS_READ_CUSTOMER`) and any
  line items so you have their ids.
- Line totals must add up: an invoice's amount derives from its line items, and a
  bill payment / `QUICKBOOKS_CREATE_PAYMENT` should reconcile against the
  document(s) it pays. Don't post a payment for more than the open balance.

## Updating: sparse update needs the current SyncToken

- `QUICKBOOKS_UPDATE_SPARSE_INVOICE` updates only the fields you send, but QBO
  uses **optimistic concurrency**: you must pass the invoice's `Id` **and its
  current `SyncToken`**. Read the invoice first to get the latest token; a stale
  token is rejected. Re-read and retry if it fails.
- Sparse update is the safe default for edits — it leaves untouched fields alone,
  whereas a full update (`QUICKBOOKS_UPDATE_FULL_INVOICE`) replaces the whole
  record and blanks anything you omit.

## Reports

- `QUICKBOOKS_GET_PROFIT_AND_LOSS_REPORT` and
  `QUICKBOOKS_GET_BALANCE_SHEET_REPORT` take a **date range** (and often an
  accounting basis, cash vs. accrual) and return a structured row tree, not flat
  rows. State the period and basis explicitly; don't assume a default. Use
  `QUICKBOOKS_GET_COMPANY_INFO` to confirm the company's fiscal-year start and
  currency before interpreting figures.

## Care

- These are **books of record**. Created invoices, payments, and bills are
  customer/vendor-visible and feed financials and taxes. Confirm amounts,
  customer/vendor, dates, and accounts before creating anything.
- There's **no void tool** — in QBO a void is a special kind of update. If a
  posting must be voided rather than corrected, tell the user to void it in
  QuickBooks directly. For a wrong-but-fixable posting, prefer a corrective
  **sparse update** over deleting and re-creating.
- Deletes exist for some entities, and `QUICKBOOKS_EXECUTE_BATCH_OPERATION` can
  mutate many records at once — both are blunt instruments. Never delete or
  batch-mutate without explicit, narrowly-scoped confirmation.
- Numbers may be in the company's home currency — multi-currency files store an
  exchange rate per transaction. Don't silently mix currencies.
- Amounts and dates are easy to fat-finger and hard to unwind. When the user is
  vague ("invoice them for the usual"), surface the exact figure and line items
  for confirmation rather than guessing.
