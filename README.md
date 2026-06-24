# connector-skills

Curated **skill overlays** for third-party connectors on the [NimbleBrain](https://nimblebrain.ai) platform.

When a connector is installed in a workspace, the runtime looks up the matching overlay here, materializes it into the workspace, and **surfaces it once into the conversation the first time the connector's tools are used** — so the agent gets concise "how to use these tools" guidance exactly when it's relevant, without bloating the system prompt on every turn.

These overlays exist for connectors we **don't control** (Composio toolkits, third-party remote MCP servers) — we can't ship guidance inside a server we don't run, so we curate it here. Connectors we *do* control ship their own skill.

## Layout — `<identity>/SKILL.md`

The runtime resolves an overlay by **connector identity**:

| Connector kind | Identity | Path |
|---|---|---|
| Composio toolkit | `composio/<toolkit>` | `composio/gmail/SKILL.md` |
| Other third-party (remote MCP) | the connector slug | `notion/SKILL.md` |

A connector with no overlay here is a no-op (no guidance injected) — coverage grows over time.

## Overlay format

Each `SKILL.md` is a **standard [Agent Skill](https://agentskills.io)** — YAML frontmatter (`name`, `description`) + a markdown body. **Do not** add a `metadata.nimblebrain` block: the runtime stamps the NimbleBrain config (`loading-strategy: dynamic`, `tool-affinity`, `scope`, provenance) when it materializes the overlay. Keep the source pristine and portable.

```markdown
---
name: gmail
description: >-
  How to use the Gmail connector's tools — sending, searching, drafting, and
  threading email. Use when the user works with Gmail.
---

You are using the **Gmail** connector...
```

Conventions:
- **`name`** must be lowercase letters/digits with single hyphens (e.g. `microsoft-teams`, even though the path is `composio/microsoft_teams/`).
- **`description`** says what the connector does *and when to use it* — it's the activation signal.
- **Body** is tool-use guidance only: tool conventions, common workflows, gotchas, when-to-use. No secrets. Keep it tight (a per-skill body cap applies at load) — push depth into the description, not a wall of prose.

## Versioning

The runtime pins a release **tag** of this repo. Cut a new tag to roll updated overlays to the fleet.
