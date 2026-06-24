---
name: neon
description: >-
  How to use the Neon connector's tools — running SQL against serverless
  Postgres, inspecting schema, and making schema changes safely through Neon's
  branch-based migration workflow. Use when the user works with Neon, Postgres,
  a database, tables, queries, schema, migrations, or branches.
---

You are using the **Neon** connector (remote MCP). Its tools act on the connected
user's Neon account — serverless Postgres organized as projects → branches →
databases. Apply this guidance whenever you touch Neon.

## Resolve the target first

- Neon is a hierarchy: a **project** contains **branches** (Git-like copies of the
  database), each branch has one or more **databases**. Most write/query tools need
  a **projectId**, a **branchId**, and a **databaseName** — resolve these before
  acting, don't guess ids.
- **List/describe to discover**: list projects, describe a project to see its
  branches and databases, then list tables and describe a table's schema before you
  query or change anything. Inspect the schema first — a wrong column or table name
  errors or returns nothing.

## Querying

- `run_sql` executes a **single** statement; `run_sql_transaction` runs several in
  one transaction (all-or-nothing). Use the transaction form when statements must
  succeed or fail together.
- These run **real Postgres** — standard SQL, not a query DSL. `SELECT` for reads;
  prefer parameterized/well-scoped statements over string-building. Add `LIMIT` to
  exploratory queries instead of pulling whole tables, and summarize large results.
- To hand a connection to the user (e.g. for their app or psql), use the
  connection-string tool rather than printing credentials you assembled yourself.

## Schema changes go through the migration workflow

Do **not** run `ALTER`/`CREATE`/`DROP` directly on a main branch with `run_sql`.
Neon provides a safe two-phase flow — use it:

1. **Prepare** the migration with the DDL. Neon spins up a **temporary branch**,
   applies the change there, and returns a **migrationId** plus the temp branch.
2. **Test** on that temporary branch — query the changed tables, confirm the schema
   is what you intended. Tell the user the migration is staged on a test branch and
   show what changed before committing.
3. **Complete** the migration with the migrationId only after the user confirms —
   this applies the change to the real branch and cleans up the temp branch.

Never call complete without testing the prepared branch first. The same prepare /
test / complete pattern exists for query tuning.

## Care

- **Irreversible**: deleting a project (destroys all its branches and data),
  deleting a branch, and resetting a branch from its parent (discards the branch's
  changes). Also any `DROP`/`DELETE`/`TRUNCATE` you send through `run_sql`. Confirm
  the exact target (and that it's the branch you think) before any of these — never
  bulk-delete or reset without explicit scope.
- Distinguish **branch resets** from migrations: a reset throws away work; the
  migration flow preserves it. Don't reach for reset to "undo" — prefer a new branch.
- Branches are cheap and isolated — when in doubt about a destructive or risky
  change, do it on a fresh branch first rather than on main.
- If a project/branch isn't found, it may simply not be on the connected account
  rather than missing — say so instead of inventing ids.
