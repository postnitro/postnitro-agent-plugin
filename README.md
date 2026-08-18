# PostNitro Agent Plugin

Create on-brand social media **carousels, image posts, and short videos** — and schedule them to
LinkedIn, Instagram, TikTok, and Threads — from any AI agent.

Generate from a topic, article, or X thread, or import your own slides. Every operation is
JSON in / JSON out, so an agent can run the whole create-to-schedule workflow unattended.

This repository is a portable plugin conforming to the
[Agent Plugins specification v1.0.0](https://agent-plugins.org).

## What's inside

| Path | Purpose |
|------|---------|
| `plugin.json` | Agent Plugins 1.0.0 manifest (required) |
| `mcp.json` | PostNitro MCP server — 35 tools over Streamable HTTP |
| `skills/postnitro/` | The `postnitro` Agent Skill: CLI-driven create + schedule workflow |
| `.claude-plugin/`, `.mcp.json` | Claude Code client integration |
| `.codex-plugin/`, `.agents/plugins/` | Codex / ChatGPT client integration |
| `skills/postnitro/agents/openai.yaml` | ChatGPT app presentation (icons, brand color) + connector dependency |

```text
postnitro-agent-plugin/
├── plugin.json
├── mcp.json
├── skills/
│   └── postnitro/
│       ├── SKILL.md
│       ├── references/
│       │   └── cli-reference.md
│       └── examples/
│           ├── EXAMPLES.md
│           ├── import-default.json
│           ├── import-infographics.json
│           ├── import-video.json
│           ├── schedule-post.json
│       ├── agents/
│       │   └── openai.yaml
│       └── assets/
│           ├── square-logo.svg
│           └── square-logo.png
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── .mcp.json
├── .codex-plugin/
│   ├── plugin.json
│   └── mcp.json
└── .agents/
    └── plugins/
        └── marketplace.json
```

Only `plugin.json`, `mcp.json`, and `skills/` are the portable spec. The `.claude-plugin/`,
`.codex-plugin/`, `.agents/`, and `.mcp.json` paths are client integrations that let the plugin
install today in clients that have not yet adopted the Agent Plugins layout — they point back at
the same `skills/` directory, so there is only ever one copy of the skill.

The plugin gives an agent two complementary paths to PostNitro:

- **The skill** drives the [`@postnitro/cli`](https://www.npmjs.com/package/@postnitro/cli) —
  best for scripted, chainable workflows and for agents with shell access.
- **The MCP server** exposes 35 native tools — best for agents that prefer structured tool calls
  over shell commands.

Either one is sufficient on its own. Clients that support both will load both.

## Prerequisites

A PostNitro account with a **paid subscription** — the Embed API has no free tier — and an
Embed API key.

To get a key: log in to [postnitro.ai](https://postnitro.ai) → profile icon → **Embed** →
add your domains under **Add Whitelist Domains** → **Generate API Key**. Keys start with `pn-`.

## Installation

### Claude Code

```bash
claude plugin marketplace add postnitro/postnitro-agent-plugin
```

```bash
claude plugin install postnitro@postnitro-agent
```

Then set your key so the bundled MCP server can authenticate:

```bash
export POSTNITRO_API_KEY="pn-your-api-key-here"
```

### Codex CLI

```bash
codex plugin marketplace add postnitro/postnitro-agent-plugin
```

```bash
export POSTNITRO_API_KEY="pn-your-api-key-here"
```

Then install `postnitro` from the marketplace and run `/reload-plugins`. Codex reads its manifest
from `.codex-plugin/plugin.json`, which points at the same `skills/` directory and at
`.codex-plugin/mcp.json` for the MCP server.

### ChatGPT app

Install the plugin from the Plugins Directory once published; MCP servers bundled in a plugin
manifest are picked up automatically. To connect the server by hand instead:
**Settings → MCP servers → Add server → Streamable HTTP**, URL `https://mcp.postnitro.ai/mcp`.

[`skills/postnitro/agents/openai.yaml`](skills/postnitro/agents/openai.yaml) supplies the display
name, description, icons, brand color, and connector dependency used by the ChatGPT desktop app.

### Any Agent Plugins–compatible client

Clone the repository and point your client at the directory:

```bash
git clone https://github.com/postnitro/postnitro-agent-plugin.git
```

Installation, permissions, and UX are each client's responsibility — consult your client's
documentation for how it loads a plugin directory.

## Configuring the MCP server

`mcp.json` declares the hosted PostNitro MCP server:

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/mcp.schema.json",
  "mcpServers": {
    "postnitro": {
      "type": "streamable-http",
      "url": "https://mcp.postnitro.ai/mcp",
      "headers": {
        "Authorization": "Bearer pn-your-api-key-here"
      }
    }
  }
}
```

**Replace `pn-your-api-key-here` with your own key after installing.**

The Agent Plugins spec expands only `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` inside `mcp.json`, so a
portable manifest cannot read your key from the environment — the placeholder must be edited by
hand. Do not commit your real key back to a fork you intend to publish.

Both client-specific MCP files read the key from the `POSTNITRO_API_KEY` environment variable
instead, so no file editing is needed there:

- `.mcp.json` (Claude Code) — `"headers": { "Authorization": "Bearer ${POSTNITRO_API_KEY}" }`
- `.codex-plugin/mcp.json` (Codex) — `"bearer_token_env_var": "POSTNITRO_API_KEY"`

These are kept as two files because the three formats disagree on how a remote server declares its
auth: the portable spec has no env expansion at all, Claude Code uses `type`/`headers`, and Codex
uses `bearer_token_env_var`/`http_headers`.

## Configuring the CLI

The skill installs and drives the CLI:

```bash
npm install -g @postnitro/cli
```

Authenticate, in order of precedence — `--api-key` flag, then env var, then saved config:

```bash
postnitro auth set-key pn-your-api-key-here
```

Optionally save defaults so template, brand, and preset IDs don't have to be repeated on every
call:

```bash
postnitro defaults set --template-id <id> --brand-id <id> --preset-id <id> --response-type PDF
```

> `auth set-key` writes the key in plaintext to `~/.postnitro-cli/config.json`. Avoid it on shared
> or untrusted machines — prefer `POSTNITRO_API_KEY` there — and run `postnitro auth clear` to
> remove it.

## Usage

Ask your agent in plain language:

- "Turn this blog post into a 7-slide LinkedIn carousel and schedule it for Tuesday 9am."
- "Make an image post about our pricing change using the Aurora brand kit."
- "Create a short video from this X thread and save it as a draft."

See [`skills/postnitro/SKILL.md`](skills/postnitro/SKILL.md) for the full workflow,
[`references/cli-reference.md`](skills/postnitro/references/cli-reference.md) for every command and
flag, and [`examples/EXAMPLES.md`](skills/postnitro/examples/EXAMPLES.md) for end-to-end payloads.

**One rule worth knowing up front:** create commands return an `embedPostId` (the async job).
Scheduling needs the `designId`, which comes from `--wait` output or `postnitro carousel output`.
Never pass an `embedPostId` to `schedule`.

## Validating the manifests

```bash
pip install check-jsonschema
```

```bash
check-jsonschema --schemafile https://agent-plugins.org/schemas/1.0.0/plugin.schema.json plugin.json
```

```bash
check-jsonschema --schemafile https://agent-plugins.org/schemas/1.0.0/mcp.schema.json mcp.json
```

Both schemas set `additionalProperties: false`, so unrecognized fields are validation errors.

## Links

- [PostNitro](https://postnitro.ai)
- [Agent Plugins specification](https://agent-plugins.org)
- [`@postnitro/cli` on npm](https://www.npmjs.com/package/@postnitro/cli)
- [Agent Skills specification](https://agentskills.io/specification)
- [Codex plugin documentation](https://developers.openai.com/plugins/build/plugins)

## License

MIT — see [LICENSE](LICENSE).
