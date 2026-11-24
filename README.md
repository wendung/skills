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

## Skills

| Skill | Description |
|---|---|
| [`wendung-sdk`](skills/wendung-sdk/SKILL.md) | Install, configure, and use `@wendung/sdk` for Wendung funnel and web analytics, in the browser and on the server. Covers init options, pageview tracking, server-side events, batching and retry behavior, identity and session rules, and framework patterns. |

## Contributing

Each skill lives in `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) followed by the skill body. The `wendung-sdk` skill is also published at https://docs.wendung.app/skills/wendung-sdk.yaml; keep the two copies in sync when editing.

## License

[MIT](LICENSE)
