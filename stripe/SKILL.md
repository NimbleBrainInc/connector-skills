---
name: stripe
description: >-
  How to use the Stripe connector's tools — customers, payments, charges,
  refunds, invoices, subscriptions, products, prices, payment links, coupons,
  and disputes — plus searching Stripe's docs and API. Use when the user works
  with Stripe, payments, billing, subscriptions, invoices, refunds, customers,
  payouts, or revenue.
---

You are using the **Stripe** connector (remote MCP at mcp.stripe.com). Its tools act
on the connected Stripe account, in whichever mode (test/sandbox or live) the
session's credentials select — there is no mode parameter on the tools. Apply this
guidance whenever you touch Stripe.

## Know which mode you're in

- **Test/sandbox vs live is fixed by the credential, not chosen per call.** Live calls
  move real money. If you can't confirm it's a test session, treat every write as
  real and confirm with the user first.
- `get_stripe_account_info` resolves which account/connected account you're acting on.
  Run it once when the account is ambiguous.

## How the tools are shaped

- This remote server is mostly a **generic API surface**, not one tool per object.
  Most work goes through a pair: `stripe_api_read` (GET) and `stripe_api_write`
  (POST/PATCH/PUT/DELETE). There is **no** `create_customer` / `list_invoices` /
  `cancel_subscription` style tool here — those are API methods you invoke *through*
  the generic pair.
- `stripe_api_search` finds the right API method by keyword; `stripe_api_details` gives
  its exact parameters. Run that flow before a write you're unsure about rather than
  guessing field names.
- A few dedicated tools do exist and are worth preferring when they fit:
  `get_stripe_account_info`, `create_refund`, `search_stripe_resources` /
  `fetch_stripe_resources` (look up objects), and `search_stripe_documentation`.

## Find the object first

- Stripe objects reference each other by **id** with type prefixes: `cus_` (customer),
  `ch_`/`pi_` (charge / payment intent), `in_` (invoice), `sub_` (subscription),
  `prod_` / `price_` (product / price), `plink_` (payment link), `re_` (refund),
  `cpn_` (coupon), `dp_` (dispute). Most writes need the right id — don't guess it.
- Resolve a human reference ("Acme's last invoice") with `search_stripe_resources` /
  `fetch_stripe_resources`, or a `stripe_api_read` list call. Stripe list endpoints are
  **cursor-paginated** (`limit` + `starting_after`); page rather than asking for
  everything.
- For "how do I…" or parameter questions, use `search_stripe_documentation` — it reads
  the live docs/knowledge base, more reliable than recalling field names from memory.

## Money

- Money is in the **smallest currency unit** — `amount: 2000` is $20.00 USD, not $2000.
  Always pass an explicit `currency`. Get this wrong and you charge 100x.

## Common workflows

- **Charge / collect:** for a quick shareable link, create a payment link (it needs a
  `price`, so create the product/price first if it doesn't exist). For programmatic
  charges, create then confirm a payment intent. These run through `stripe_api_write`.
- **Refund:** `create_refund` against a charge or payment intent; omit `amount` for a
  full refund or pass an amount for partial.
- **Billing:** create an invoice, add line items, then **finalize** it — a draft
  invoice isn't billed until finalized. Subscriptions auto-generate invoices on their
  cycle.

## Care

- **Refunds, subscription cancellations, dispute responses, and coupon/object deletes
  are irreversible and move money or affect customers.** Confirm the exact object id,
  amount, and account with the user before any write. Never bulk-refund or
  bulk-cancel without explicit, scoped instruction.
- A failed write may have **partially succeeded** (e.g. the charge went through but the
  response was lost). Don't blindly retry a create — read back the object first, or
  rely on the idempotency the API provides rather than firing a second call.
- Surface amounts to the user in human units (dollars, not cents) so confirmations are
  legible. Echo currency.
