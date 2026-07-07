# Crusader Skills

An Agent Skill that teaches an AI coding agent (Claude Code and others) to hunt web/API vulnerabilities with [Crusader](https://crusaderproxy.com/) and package confirmed bugs into shareable, self-proving POCs.

## Core Capability

Crusader already exposes a first-class **MCP agent interface** (`crusader mcp serve`) with a self-describing `agent.guide` contract, plus a full `crusader` CLI. Unlike skills that must bundle a client shim to talk to their tool, this skill ships **no code**: just `SKILL.md`, pointing the agent at Crusader's native surface and layering the hunting methodology on top.

The standout play: **IDOR/BOLA in one call.** From a captured read-by-id request plus a second account, `poc.prove` discovers a record that account owns live, reads it with the first account, and returns `verdict:"confirmed"`: a real cross-account read plus a `crusader://poc/` link. The owned id is discovered per run, so the proof generalizes across accounts and bakes in no credentials.

## Setup

Requires [Crusader](https://crusaderproxy.com/) with its MCP server available as `crusader mcp serve`. For Claude Code:

```bash
claude mcp add crusader -s user -- crusader mcp serve
```

That is the whole install: no `pnpm install`, no access token, no bundled client. See [SKILL.md](SKILL.md) for the full methodology.

Helpful Crusader docs:

- [Drive Crusader from Claude Code](https://crusaderproxy.com/docs/mcp-claude-code.html)
- [The CLI and CI pipeline](https://crusaderproxy.com/docs/cli.html)
- [Write your first plugin](https://crusaderproxy.com/docs/write-a-plugin.html)

## What It Covers

- Access control: IDOR/BOLA (`poc.analyze` -> `poc.prove`), function-level authz (`identity.replay` + `oracle.actor_diff`)
- Blind / out-of-band: `beacon.mint` -> inject -> `beacon.poll`
- Race conditions: `repeater.race` single-packet proof
- XSS: `browser.confirm_xss`
- OSS-aware prioritization: `context.get` / `context.leads`
- Confirm-before-claim discipline, then `findings.create` / `findings.submit_verdict`
- Packaging: `poc.build` / `poc.link` -> a `crusader://poc/` link the triager reproduces in one click
