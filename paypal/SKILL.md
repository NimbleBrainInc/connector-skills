---
name: paypal
description: >-
  How to use the PayPal connector's tools — invoices, orders and payments,
  refunds, subscriptions, products/catalog, disputes, shipment tracking, and
  transaction reporting on a connected PayPal merchant account. Use when the user
  works with PayPal, invoices, billing a customer, capturing or refunding a
  payment, subscriptions/recurring billing, disputes/chargebacks, shipping
  tracking, or merchant transaction reports.
---

You are using the **PayPal** connector (remote MCP). Its tools act on the connected
merchant's PayPal account and **move real money**. Apply this guidance whenever you
touch PayPal.

## Find the target first

- Tools are snake_case and action-first: `create_invoice`, `list_invoices`,
  `get_invoice`, `get_order`, `create_refund`, `show_subscription_details`,
  `list_disputes`, `list_transactions`. The exact set you see depends on the token's
  permissions — restricted tokens expose fewer tools.
- Almost every write/read-by-id needs a PayPal **id**: `invoice_id`, `order_id`,
  `capture_id`, `subscription_id`, `plan_id`, `product_id`, `dispute_id`. **List or
  get first to resolve a human reference ("the invoice I sent Acme") into an id** —
  don't guess ids or amounts.
- Reporting tools (`list_transactions`, `get_merchant_insights`) take **date ranges**;
  pass explicit `start_date`/`end_date` rather than fetching everything.

## Invoices

- Creating an invoice is a **draft** — it does nothing until you `send_invoice`,
  which **emails the customer and requests payment**. Treat send as the commit step:
  confirm recipient, line items, and amounts first, then send.
- Use `send_invoice_reminder` for nudges (safe, repeatable). `cancel_sent_invoice`
  voids an already-sent invoice — irreversible and visible to the customer.

## Orders, payments, and refunds

- `create_order` sets up a payment intent; the order is **captured** (money actually
  taken) by the pay/capture step. Don't capture without explicit user intent.
- A **refund** needs the **`capture_id`** from a captured payment, not the order id —
  fetch the order/transaction to get the capture id first. `create_refund` is
  **irreversible**; partial refunds need an explicit `amount` + `currency`, otherwise
  it refunds in full. Confirm amount and currency before issuing.

## Subscriptions, products, disputes

- Recurring billing is layered: a **product** → a **subscription plan** (billing
  cycles, pricing) → a **subscription** for a customer. Create/inspect in that order.
- `cancel_subscription` stops future billing and is irreversible — confirm before
  cancelling. `update_subscription` revises an existing subscription in place.
- Disputes: `list_disputes`/`get_dispute` are read-only. `accept_dispute_claim`
  **concedes the dispute and refunds the buyer** — a money-losing, irreversible
  resolution; never accept a claim without explicit instruction.

## Care

- This connector spends and moves money. Sending invoices, capturing orders,
  refunding, cancelling subscriptions, and accepting disputes are all **irreversible
  and customer-visible** — state the exact amount, currency, and counterparty, and
  get explicit confirmation before any of them. Never batch these across many records.
- Watch the **environment**: sandbox vs. production are separate worlds. If results
  look like test data (or the account is sandbox), say so rather than assuming live.
- Amounts are currency-specific and minor-unit sensitive — echo back the value and
  currency code you're about to act on.
