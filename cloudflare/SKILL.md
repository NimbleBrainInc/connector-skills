---
name: cloudflare
description: >-
  How to use the Cloudflare connector's tools — listing and inspecting Workers,
  managing KV namespaces, R2 buckets, D1 databases, and Hyperdrive configs, and
  running SQL against D1. This is the Workers Bindings server, scoped to one
  Cloudflare account. Use when the user works with Cloudflare, Workers, KV, R2,
  D1, Hyperdrive, edge storage, or serverless bindings — NOT DNS records, zones,
  WAF, page rules, or CDN/cache settings (this server does not expose those).
---

You are using the **Cloudflare** connector (remote MCP — the Workers Bindings
server). Its tools read and manage the Workers application primitives in the
connected user's Cloudflare account: Workers, KV, R2, D1, and Hyperdrive. Apply
this guidance whenever you touch Cloudflare.

## Scope check first

- This is the **Workers Bindings** server. It covers compute/storage *bindings*:
  Workers scripts, KV namespaces, R2 buckets, D1 databases, Hyperdrive configs.
- It does **not** manage **DNS records, zones, WAF, page rules, cache, or
  Analytics**. If the user asks for those, say so — don't try to force a binding
  tool to do it.

## Account scoping (the #1 gotcha)

- Credentials may reach **multiple accounts**. Account-scoped tools need an
  **`account_id`**. The server lists available accounts in its instructions —
  read those, confirm which account the user means, then pass `account_id` on
  each call. Don't guess the account.
- If a tool errors with an account/permission message, it usually means the
  `account_id` is missing or wrong, not that the resource doesn't exist.

## Tool naming and find-first

- Tools follow `{resource}_{action}`: list/get/create/delete/update, e.g.
  `kv_namespaces_list`, `r2_bucket_get`, `d1_database_create`,
  `hyperdrive_config_edit`, `workers_list`.
- **List before you act.** Most operations need a resource **id or name**
  (namespace id, bucket name, database id, worker name) — list to resolve a
  human reference into the real identifier, then act by id. Don't invent ids.
- To inspect a Worker, list workers first, then fetch its details/code; large
  scripts come back as text — summarize rather than dumping the whole file.

## D1 queries

- `d1_database_query` runs **SQL against a live D1 database** by id. Treat it
  like any production database connection:
  - Resolve the database id via list first; confirm you're on the right one.
  - For exploration prefer **read-only** SQL (`SELECT`, `PRAGMA table_list`,
    `PRAGMA table_info`) and inspect the schema before writing anything.
  - `INSERT/UPDATE/DELETE/DROP/ALTER` mutate real data and are **not
    transactional across calls** — get explicit confirmation, scope `UPDATE`/
    `DELETE` with a precise `WHERE`, and never run an unbounded mutation.

## Care

- **Creates and deletes are real account changes.** `*_delete` on a KV
  namespace, R2 bucket, or D1 database is **irreversible** and destroys the
  stored data — confirm the exact target id and the user's intent first; never
  bulk-delete.
- Creating resources (namespaces, buckets, databases, Hyperdrive configs) can
  **incur billing** and may collide with names used by deployed Workers.
  Confirm name/region before creating.
- Editing a Worker's bindings or a Hyperdrive config affects **running
  production** behavior. State the blast radius before changing live config.
- Prefer read/list/get operations when the user only wants to understand
  what's there; reach for create/delete only when the intent is explicit.
