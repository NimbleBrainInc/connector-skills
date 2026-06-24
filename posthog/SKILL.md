---
name: posthog
description: >-
  How to use the PostHog connector's tools — running HogQL/SQL queries, reading
  events and persons, pulling trends, funnels, and retention, and inspecting
  insights, dashboards, feature flags, and annotations. Use when the user asks
  about product analytics, usage metrics, conversion, churn, active users,
  funnels, retention cohorts, event volume, feature-flag rollout, or "what does
  PostHog say about X".
---

You are using the **PostHog** connector (via Composio). Its tools act on the
connected PostHog account's analytics data. Apply this guidance whenever you
touch PostHog. This is a **read-leaning** working set — most tools query and
inspect; a few can mutate, called out under Care.

## Trust the live tool list, not these names

- This connector exposes many tools, all slugged `POSTHOG_*`. The exact slug
  names vary and change — **look them up in your available tools** and read each
  tool's schema before calling it. Don't invent a slug or a parameter from the
  shape of a name you remember; match against the real list and confirm the
  fields the schema asks for.

## Resolve the project first

- Almost every tool is **scoped to a project** and needs a **project id**. Don't
  guess it — call the list-projects tool once to resolve the human name
  ("Production", "the web app") into an id, then pass that id to every
  subsequent call.

## HogQL is the power tool

- For anything ad-hoc or cross-cutting, prefer the **query** tool with a
  **HogQL** query (`kind: "HogQLQuery"`, `query: "SELECT ..."`). HogQL is
  PostHog's SQL dialect over ClickHouse: `SELECT … FROM events | persons |
  sessions WHERE … GROUP BY … ORDER BY … LIMIT …`.
- Event/person props are nested: `properties.$browser`, `person.properties.email`.
  Auto-joins resolve `events.person.properties.*` for you.
- **Always bound the time range** and keep it tight — `timestamp > now() -
  INTERVAL 7 DAY`, `dateDiff(...)`, etc. Unbounded scans over events are slow and
  expensive. Always add a `LIMIT`.
- Heavy queries may run **async**: the create-query call returns a status with an
  **id**; poll the async-status tool until complete, then read results. Don't
  re-issue the query while one is pending.

## Pre-built analytics vs. raw query

- For standard shapes, prefer the purpose-built **trends**, **funnels**, and
  **retention** tools over hand-rolled HogQL. They return results in PostHog's
  native insight shape and match what the user sees in the UI.
- To discover what events/properties even exist, use the event-definitions /
  property-definitions tools before filtering — event names are case-sensitive
  and project-specific (`$pageview`, `$autocapture`, custom names).

## Inspecting saved work

- **Insights** are saved analyses: list them (with pagination), then fetch one by
  id for its full config/results. Reuse a saved insight instead of re-deriving
  its numbers when the user references it.
- **Dashboards**, **feature flags**, and **annotations** each have a list tool and
  a get-by-id tool — list to resolve a name to an id, then read details.

## Care

- This set is read-leaning, but some endpoints can **mutate** — deleting persons,
  toggling or deleting feature flags, capturing events, editing annotations or
  insights. Default to the read path; only delete/modify when the user explicitly
  asks, and confirm scope first. **Deleting a person is irreversible**, and
  toggling a feature flag is immediately live to real users.
- Numbers are only as good as the filters. State your time window, project, and
  any event/property filters when reporting a figure, so the user can trust it.
- Large result sets: page through (`limit`/`offset` or the cursor the tool
  returns) and summarize rather than dumping every row. Add a `LIMIT` to every
  HogQL query as a backstop.
- Don't fabricate an event, property, flag, or insight name. If a lookup returns
  nothing, say it wasn't found (it may not exist or not be tracked) instead of
  guessing a plausible name.
