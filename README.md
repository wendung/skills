# Wendung skills

Agent skills for [Wendung](https://wendung.app) funnel and web analytics. Skills teach AI coding agents (Claude Code, Cursor, Codex, and others) how to work with Wendung without guessing at APIs.

## Install

```bash
npx skills add wendung/skills
```

This installs every skill in this repo for your configured agents. To install a single skill:

```bash
npx skills add wendung/skills --skill wendung-sdk
```

## Plugin

The `wendung` plugin installs the `wendung-sdk` skill and connects two Wendung MCP servers:

| Server | URL | What it does | Sign-in |
|---|---|---|---|
| `wendung` | https://api.wendung.app/mcp | Reads your workspaces, projects, funnels and live analytics. Read access by default. Write tools need a client that requests write access and your approval at sign-in | OAuth with your Wendung account |
| `wendung-docs` | https://docs.wendung.app/mcp | Searches the Wendung docs | None |

The skill may also fetch pages from https://docs.wendung.app. The plugin runs no local code.

### Claude Code

```
/plugin install wendung --marketplace wendung/skills
```

On Claude Code versions before 2.1.275, add the marketplace first with `/plugin marketplace add wendung/skills`, then run `/plugin install wendung@wendung`. After installing, run `/mcp` and sign in to the `wendung` server.

### Cursor

The same repository is a Cursor plugin with the same skill and servers. After installing it, sign in to the `wendung` server from Cursor's MCP settings.

## Skills

| Skill | Description |
|---|---|
| [`wendung-sdk`](skills/wendung-sdk/SKILL.md) | Install, configure, and use `@wendung/sdk` for Wendung funnel and web analytics, in the browser and on the server. Covers init options, pageview tracking, server-side events, batching and retry behavior, identity and session rules, and framework patterns. |

## Contributing

Each skill lives in `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) followed by the skill body. The `wendung-sdk` skill is also published at https://docs.wendung.app/skills/wendung-sdk.yaml; keep the two copies in sync when editing.

The repository root is also the `wendung` plugin. `.claude-plugin/` holds the Claude Code manifest and marketplace, `.cursor-plugin/plugin.json` the Cursor manifest, and `.mcp.json` the two MCP servers that both share. The plugin name is permanent: installs are recorded under it. Raise `version` in both `.claude-plugin/plugin.json` and `.cursor-plugin/plugin.json` whenever `skills/` or `.mcp.json` changes, or plugin users keep the old copy.

## License

[MIT](LICENSE)
