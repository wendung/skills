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

### Claude Code plugin

The `wendung` plugin installs the skill and connects both Wendung MCP servers in one step:

```
/plugin install wendung --marketplace wendung/skills
```

On Claude Code versions before 2.1.275, add the marketplace first with `/plugin marketplace add wendung/skills`, then run `/plugin install wendung@wendung`. The analytics server asks you to sign in to Wendung the first time your agent uses it.

## Skills

| Skill | Description |
|---|---|
| [`wendung-sdk`](skills/wendung-sdk/SKILL.md) | Install, configure, and use `@wendung/sdk` for Wendung funnel and web analytics, in the browser and on the server. Covers init options, pageview tracking, server-side events, batching and retry behavior, identity and session rules, and framework patterns. |

## Contributing

Each skill lives in `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) followed by the skill body. The `wendung-sdk` skill is also published at https://docs.wendung.app/skills/wendung-sdk.yaml; keep the two copies in sync when editing.

The repository root is also the `wendung` plugin. `.claude-plugin/` holds the Claude Code manifest and marketplace, `.cursor-plugin/plugin.json` the Cursor manifest, and `.mcp.json` the two MCP servers that both share. The plugin name is permanent: installs are recorded under it.

## License

[MIT](LICENSE)
