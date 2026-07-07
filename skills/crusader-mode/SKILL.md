---
name: crusader-mode
description: Full Crusader security-proxy integration for Claude Code. Hunt web/API vulnerabilities (IDOR/BOLA, auth, injection, SSRF/OOB, races) over Crusader's native MCP, confirm with oracles, and turn confirmed bugs into shareable self-proving crusader://poc/ links. Use when testing an app, reproducing a reported bug, or whenever the user mentions Crusader, a proxy hunt, or packaging a proof-of-concept.
---

# Crusader Mode

[Crusader](https://crusaderproxy.com/) is a local-first web security proxy with a first-class **MCP agent interface**. This skill teaches you to hunt with it the way experienced bug-bounty hunters do: work from real captured traffic, **confirm with an oracle before you claim anything**, and hand the triager a POC that proves itself.

Useful docs:

- [Drive Crusader from Claude Code](https://crusaderproxy.com/docs/mcp-claude-code.html)
- [The CLI and CI pipeline](https://crusaderproxy.com/docs/cli.html)
- [Crusader documentation](https://crusaderproxy.com/docs/)

## 0. Connect Once

Crusader ships an MCP server. If it is not already configured in your client:

```bash
claude mcp add crusader -s user -- crusader mcp serve
```

For any other MCP client, add a server whose command is `crusader mcp serve`.

## 1. Always Start With `agent.guide`

**Before any other Crusader tool, call `agent.guide`.** It returns the live contract: the current tool list, the DB-backed query model, replay-safety defaults, scope rules, and the continuous-use loop. Follow that loop. This skill is the *methodology* on top of it; `agent.guide` is the ground truth for the *mechanics*. If a tool named here differs from `agent.guide`, trust `agent.guide`.

## 2. Orient Without Sending Traffic

- `project.current`: identify the active project and target.
- `scope.list`: if it reports `open_scope=true` or `needs_scope_setup=true`, call `scope.suggest`, then **ask the user** which patterns to add via `scope.add`. Never test out of scope.
- `sitemap.tree` and `history.search_safe`: use already-captured redacted traffic to map endpoints.
- `history.get_safe`: retrieve one safe request. Only use raw `history.search` / `history.get` when the user explicitly needs cookies, tokens, or full bodies.
- `identity.list`: list saved actors for cross-account testing.
- `matchreplace.list`: know which rules may rewrite your outgoing requests.

## 3. Hunt With Playbooks

Map each hypothesis to a captured endpoint from `sitemap.tree`, test with `repeater.send`, and **confirm with an oracle**. A lead is not a finding until an oracle says so.

### IDOR / BOLA

Broken object-level authorization is the highest-yield play. Find a request that reads one object by id, such as `GET /api/orders/1001`, then:

1. `poc.analyze`: confirm the request is IDOR-shaped and extract the object id plus collection endpoint.
2. `poc.prove` with a **second account's `owner_token`**: Crusader asks that account what it owns live, then tries to read it with the captured account.

`verdict:"confirmed"` means a real cross-account read. The owned id is discovered per run, so the proof generalizes across accounts, does not hardcode an id, and bakes in no credentials.

### Broken Function-Level Authz

Replay the same privileged request as a lower-privilege identity with `identity.replay`, then compare with `oracle.actor_diff`. A low-priv actor receiving the high-priv response is the bug.

### Blind / Out-of-Band Bugs

For blind SSRF, OOB RCE, blind SQLi, XXE, or blind XSS:

1. `beacon.mint` a unique payload.
2. Inject it with `repeater.send`.
3. Confirm with `beacon.poll`.

A callback is proof.

### Race Conditions

Use `repeater.race` for single-packet bursts. This is the right tool for limit-overrun, double-spend, and TOCTOU hypotheses.

### XSS

Inject a marker, then confirm reflection or execution with `browser.confirm_xss`.

### OSS-Aware Prioritization

The moment you fingerprint a known open-source component from headers, generator tags, JS libraries, or versions, call `context.get` then `context.leads` on its public repo. Treat each returned sink, severity, and file:line as a hunting hypothesis. Leads are candidates, not findings. Map each one to a live endpoint and confirm.

## 4. Confirm, Then Record

Only after an oracle confirms:

- `findings.create`
- `findings.submit_verdict`

A `real` verdict requires proof. Write compact, redacted notes with `agent.memory.write` so the next session, and `agent.next_move`, build on this one.

## 5. Package the Proof

Turn a confirmed bug into something a triager reproduces in one click:

- **IDOR**: `poc.prove` returns the manifest and a `crusader://poc/` link.
- **Anything else**: `poc.build` with the request. The host is templated and every credential is stripped into a slot the triager fills with their own account.
- `poc.link` turns any manifest into a self-contained `crusader://poc/` link. The whole POC rides inside the link; nothing is hosted. Paste it into the report, and the triager clicks it, binds their own account, and Crusader re-proves the bug against the live target.

## CLI Alternative

Unlike proxies that need a bundled client shim, Crusader is agent-native: every capability is also a `crusader` CLI verb if your agent prefers shell commands to MCP calls.

- `crusader history search "<query>"`: query captured traffic.
- `crusader poc prove --owner <token> [--link] < request.txt`: prove an IDOR end-to-end from a captured read-by-id request; prints the verdict and a shareable `crusader://poc/` link.
- `crusader poc build --title "<t>" [--link] < request.txt`: package any request as a POC.
- `crusader poc run <poc.json|crusader://poc/...> --id name=TOKEN`: reproduce a POC.
- `crusader mcp tools`: list the full MCP toolset.
- `crusader --help`: list all verbs.

Prefer MCP when available: `agent.guide` gives the richest, self-describing contract.

## Rules

- **Scope-locked**: never send to a host outside `scope.list`; POCs can only fire at their declared target.
- **Confirm before claiming**: require an oracle callback, diff, or verdict, not a hunch.
- **Rate-limited**: `settings.rate_limit` governs outgoing rate; do not hammer a target.
- **Pristine by default**: `repeater.send` applies no active identity and inherits no cookies unless you pass `identity_id`, `apply_active_identity=true`, or `inherit_cookies=true`.
- **One at a time**: the stdio MCP server handles calls sequentially; do not pipeline long-running active tools.
- **Authorization first**: if scope is not set or the user has not confirmed the target is in scope for testing, ask before running any active tool.
