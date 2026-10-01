# connector-skills

Curated **skill overlays** for third-party connectors on the [NimbleBrain](https://nimblebrain.ai) platform.

When a connector is installed in a workspace, the runtime looks up the matching overlay here, materializes it into the workspace, and **surfaces it once into the conversation the first time the connector's tools are used** — so the agent gets concise "how to use these tools" guidance exactly when it's relevant, without bloating the system prompt on every turn.

These overlays exist for connectors we **don't control** (Composio toolkits, third-party remote MCP servers) — we can't ship guidance inside a server we don't run, so we curate it here. Connectors we *do* control ship their own skill.

## Layout — `<identity>/SKILL.md`

The runtime resolves an overlay by **connector identity** — a flat connector slug (the Composio toolkit name, or the remote-MCP connector's slug):

| Connector kind | Identity | Path |
|---|---|---|
| Composio toolkit | `<toolkit>` | `gmail/SKILL.md` |
| Other third-party (remote MCP) | the connector slug | `notion/SKILL.md` |

Slugs are unique across both kinds, so the namespace stays flat. A connector with no overlay here is a no-op (no guidance injected) — coverage grows over time.

## Overlay format

Each `SKILL.md` is a **standard [Agent Skill](https://agentskills.io)** — YAML frontmatter (`name`, `description`) + a markdown body. The runtime stamps the rest of the NimbleBrain config (`loading-strategy: dynamic`, `scope`, provenance) when it materializes the overlay, so leave those out and keep the source portable.

```markdown
---
name: gmail
description: >-
  How to use the Gmail connector's tools — sending, searching, drafting, and
  threading email. Use when the user works with Gmail.
---

You are using the **Gmail** connector...
```

### Optional: `metadata.nimblebrain.tool-affinity`

By default an overlay is bound to **every** tool of its connector, so it surfaces on the first call to any of them. To bind it to specific tools instead, declare them under `metadata.nimblebrain.tool-affinity`. This is the only field an overlay may set in that block.

```yaml
---
name: example-mail
description: How to send and draft email with the example connector. Use when sending or drafting.
metadata:
  nimblebrain:
    tool-affinity:
      - send_email
      - create_draft
      - reply_*
---
```

- List the connector's **bare** tool names or globs (`*` matches any run of characters), exactly as the connector names them. Do not add a server prefix: the runtime does not know the name a connector is installed under until install, so it prefixes each pattern with that install's namespace itself.
- A pattern can only match the connector's own tools. `*` means all of them, which is the same as omitting the field.
- Omit the field, or set it to `[]`, and the overlay is bound to all of the connector's tools.
- The value must be a YAML list. A blank `tool-affinity:` or a single string fails validation, and the runtime drops the whole overlay.
- A pattern that matches none of the tools the connector advertises is logged as a warning when the connector connects. Check spelling against the connector's tool list.

Requires the NimbleBrain runtime release that includes [NimbleBrainInc/nimblebrain#1467](https://github.com/NimbleBrainInc/nimblebrain/issues/1467). An earlier runtime rejects a `metadata.nimblebrain` block that has no `loading-strategy`, which drops the overlay. The runtime pins a tag of this repo (see Versioning), so do not cut a tag that contains an overlay declaring `tool-affinity` until every runtime that will pin that tag includes that release.

Conventions:
- **`name`** must be lowercase letters/digits with single hyphens (e.g. `microsoft-teams`, even though the path/slug is `microsoft_teams/`).
- **`description`** says what the connector does *and when to use it* — it's the activation signal.
- **Body** is tool-use guidance only: tool conventions, common workflows, gotchas, when-to-use. No secrets. Keep it tight (a per-skill body cap applies at load) — push depth into the description, not a wall of prose.

## Versioning

The runtime pins a release **tag** of this repo. Cut a new tag to roll updated overlays to the fleet.
