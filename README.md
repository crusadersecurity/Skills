# Crusader Skills

[Agent Skills](https://getskillmd.com) for [**Crusader**](https://crusaderproxy.com/) - the local-first web security proxy built for the AI-agent era. Point your AI coding agent at this repo and it learns to *hunt* with Crusader, not just click buttons.

**No client to install.** Crusader is already agent-native: it ships an MCP server (`crusader mcp serve`) with a self-describing `agent.guide` contract, plus a full `crusader` CLI. This skill is pure instructions, no bundled code: one line to connect, no Node runtime, no access token. It also does what other proxy skills cannot: turn a confirmed bug into a self-proving `crusader://poc/` link the triager reproduces in one click.

## Official Links

- Website: [crusaderproxy.com](https://crusaderproxy.com/)
- Documentation: [Crusader docs](https://crusaderproxy.com/docs/)
- MCP guide: [Drive Crusader from Claude Code](https://crusaderproxy.com/docs/mcp-claude-code.html)
- Extension catalog: [Crusader Extensions](https://crusaderproxy.com/extensions.html)

## Install

With the [`skills`](https://getskillmd.com) CLI:

```bash
pnpm dlx skills add crusadersecurity/skills --skill='*'
```

Install globally across your system:

```bash
pnpm dlx skills add crusadersecurity/skills --skill='*' -g
```

Or install a single skill:

```bash
pnpm dlx skills add crusadersecurity/skills --skill='crusader-mode'
```

Browse it on the web after publish: **https://getskillmd.com/github/crusadersecurity/skills/crusader-mode**

## Skills

| Skill | What it does |
| --- | --- |
| [**crusader-mode**](skills/crusader-mode/SKILL.md) | Drives Crusader over MCP to hunt web/API vulnerabilities (IDOR/BOLA, auth, injection, SSRF/OOB, races) and package confirmed bugs into shareable, self-proving `crusader://poc/` links. |

## What is an Agent Skill?

A `SKILL.md` is a portable instruction file that teaches an AI coding agent (Claude Code and others) how to operate a tool: its workflow, its commands, and the expert methodology to apply. It is Markdown with a small YAML front matter (`name`, `description`); the agent loads it on demand when the `description` matches the task at hand.

`crusader-mode` is deliberately thin over Crusader's built-in **`agent.guide`** MCP tool, Crusader's own self-describing contract, so it stays accurate as Crusader evolves. The skill supplies the *hunting methodology*; `agent.guide` supplies the live *mechanics*.

## Requirements

- [Crusader](https://crusaderproxy.com/) installed, with its MCP server available as `crusader mcp serve`.
- An MCP-capable agent. For Claude Code: `claude mcp add crusader -s user -- crusader mcp serve`.

## License

MIT - see [LICENSE](LICENSE).
