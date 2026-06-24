---
name: sentry
description: >-
  How to use the Sentry connector's tools — finding orgs/projects, searching
  issues and events, reading issue details, stack traces, traces and spans,
  running Seer root-cause analysis, and reaching long-tail operations
  (releases, alerts, monitors, dashboards, docs) via the tool catalog. Use when
  the user works with Sentry, errors, exceptions, crashes, stack traces,
  performance, spans/traces, releases, alerts, monitors, or asks to debug a
  production issue or investigate an error.
---

You are using the **Sentry** connector (remote MCP). Its tools act on the
connected Sentry account's organizations and projects, and are **read-only** —
they observe and analyze, they do not mutate Sentry state. Apply this guidance
whenever you touch Sentry.

## Resolve org and project first

- Almost every tool takes an **`organizationSlug`** (and often a project). These
  are **slugs**, not display names — resolve them before doing real work.
- Start with **`find_organizations`** to list orgs the user can access, then
  **`find_projects`** for a project slug. Don't guess a slug.
- If the user writes `my-org/my-project`, that's `organizationSlug/projectSlug` —
  parse it directly, no lookup needed. Project params accept a slug or numeric id.
- `whoami` confirms who the token is acting as when access is ambiguous.

## Two tiers of tools: direct and catalog

- A handful of operations are **direct tools**: `find_organizations`,
  `find_projects`, `search_issues`, `search_events`, `analyze_issue_with_seer`,
  and `get_sentry_resource`.
- Everything else is reached through the **catalog**: call
  **`search_sentry_tools`** with keywords to discover the right tool and its
  schema, then **`execute_sentry_tool`** with that name and arguments. This is
  how you get to issue details, stack traces, releases, alerts, monitors,
  dashboards, profiles, tag distributions, issue activity, and docs.
- Don't assume a tool exists by guessing its name. If you don't see a direct
  tool for the task, **search the catalog first** before deciding it's
  unavailable — many useful operations are intentionally not top-level.

## Searching: issues vs. events

- **`search_issues`** returns grouped problems ("the error", deduped). Use it to
  find *what's broken* — `is:unresolved is:unassigned`, `level:error firstSeen:-24h`.
- **`search_events`** returns individual occurrences/spans/logs and supports
  **aggregations** (counts, sums, averages). Use it for *how many / when / where*
  and to drill in by time, environment, release, user, or trace id. Pick the
  `dataset` (errors, logs, spans, metrics, profiles, replays) for the question.
- `query` accepts **natural language, Sentry search syntax, or a mix** — an
  embedded agent translates it but preserves explicit Sentry tokens you pass
  (`is:`, `level:`, `release:`, `environment:`, `firstSeen:-24h`). State the
  filter you want; you don't have to hand-write the whole syntax.
- These AI search tools depend on an LLM provider configured server-side — if
  search returns a config error, fall back to explicit Sentry syntax.

## Investigate one issue

- For a specific issue or a Sentry URL, **`get_sentry_resource`** is the fastest
  path: pass the whole URL and it auto-detects the type (issue, event, trace,
  span, replay, breadcrumbs, monitor, snapshot). For traces it returns a
  condensed overview by default.
- Via the catalog: `get_issue_details` (by `PROJECT-123` id or URL) for title,
  culprit, counts, and latest event; `get_event_stacktrace` for a full thread
  stacktrace; `get_issue_tag_values` for tag distributions (which URLs/browsers/
  releases are affected); `get_issue_activity` for prior comments and who acted.
- **`analyze_issue_with_seer`** runs Sentry's AI root-cause analysis and returns
  a fix recommendation. Only run it when the user asks for analysis or you can't
  determine the cause from the issue details — not as an automatic follow-up. A
  fresh run takes **~2–5 minutes**; set expectations and don't poll tightly.
  Existing analyses return instantly, so reuse one when available.

## Care

- This connector is **read-only** — it cannot resolve, ignore, assign, comment,
  or create projects/teams/DSNs. If the user asks for those, say so and point
  them to the Sentry UI rather than inventing a tool or claiming it was done.
- A "not found" issue/project is often a **permissions or wrong-slug** problem,
  not a missing object — re-resolve the slug or say it isn't accessible rather
  than inventing one.
- Large result sets: narrow with the `query` parameter (orgs/issues/events all
  cap results) and the time range instead of fetching broadly, then summarize.
- Quote stack traces and Seer output as evidence; don't paste an entire event
  payload back verbatim. Stack frames and event data can contain user PII —
  surface only what's needed to explain the issue.
