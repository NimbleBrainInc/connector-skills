---
name: mercury
description: >-
  How to use the Mercury connector's tools — reading business banking accounts,
  balances, transactions, statements, cards, treasury, recipients, and the
  organization. Read-only. Use when the user asks about their Mercury account,
  bank balance, cash position, runway, recent transactions, spend, statements,
  debit/credit cards, treasury holdings, or who they pay (recipients).
---

You are using the **Mercury** connector (remote MCP) for Mercury business banking.
Its tools act on the connected Mercury organization. This connector is **read-only**:
it can look up accounts, balances, transactions, statements, cards, treasury, and
recipients — it **cannot move money, create transactions, or change anything**. If a
user asks to send a payment or edit data, say it isn't possible here and point them
to Mercury directly.

## Find the account first

- Start from **`getAccounts`** (list all accounts for the org) to resolve a human
  reference ("checking", "the savings account") into an **account id**. Most other
  tools take an account id — don't guess one.
- `getAccount` returns one account's detail incl. its balance. `getOrganization`
  gives org-level context.
- Tool names follow a `getX` / `getXs` / `listX` convention (camelCase), e.g.
  `getAccounts`, `getAccountCards`, `listTransactions`. If a name doesn't match,
  list the connector's tools rather than inventing one.

## Transactions and statements

- **`listTransactions`** is the cross-account list view — filter by **date range,
  status, and category**, and page with a **cursor**. Use it for "recent spend",
  "last 6 months", "all payments to X". Push filters into the call; don't pull
  everything and filter yourself.
- **`getTransaction`** needs both an **account id and a transaction id** — get the id
  from a list call first.
- Results are **cursor-paginated**: follow the next cursor until you have the range
  the user asked for, then stop. For long histories, summarize rather than dumping
  every row.
- **`getAccountStatements`** returns monthly statements for an account and supports
  date-range filtering; statements may be downloadable PDFs.

## Cards, treasury, recipients

- **`getAccountCards`** lists the debit/credit cards on an account; **`listCredit`**
  lists the org's credit accounts.
- **`getTreasury`** lists treasury accounts; **`getTreasuryTransactions`** pages
  treasury activity for a specific treasury account id.
- **`getRecipients`** / **`getRecipient`** show who the org pays. Recipient and card
  data is sensitive — surface only what's asked.
- **`listCategories`** returns the org's custom categories (distinct from Mercury's
  built-ins) — use it to interpret transaction categories correctly.

## Care

- This is **financial data** and Mercury MCP is **beta**. Treat figures as
  point-in-time and tell the user to verify in Mercury before any consequential
  decision (paying bills, reporting runway, filing). Don't present a balance as a
  guarantee.
- Don't expose full card numbers, account/routing numbers, or recipient banking
  details unless explicitly asked; redact by default.
- For balance/runway questions, state the account(s) and the as-of moment you read,
  since balances move.
