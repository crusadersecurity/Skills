---
name: crusader-triage
description: Triage existing Crusader findings and scanner output. Use when reviewing findings, deciding whether issues are real/fake/uncertain, prioritizing scanner results, preparing report-ready evidence, or applying Crusader verdicts through findings.submit_verdict. Works over Crusader MCP/CLI with a proof-first, read-only-by-default workflow.
---

# Crusader Triage

[Crusader](https://crusaderproxy.com/) is a local-first web security proxy with MCP, findings, scanner evidence, oracles, and `crusader://poc/` proof links. This skill is for **reviewing evidence and submitting verdicts**, not broad hunting.

Useful docs:

- [Drive Crusader from Claude Code](https://crusaderproxy.com/docs/mcp-claude-code.html)
- [The Scanner: passive to active proof](https://crusaderproxy.com/docs/scanner.html)
- [Bug-hunting workflow](https://crusaderproxy.com/docs/bug-hunting.html)

## 1. Start Safe

Always call `agent.guide` first. Treat it as the live contract for tool names, tier gates, and safety rules.

Orient without sending traffic:

- `project.current`: confirm the active project.
- `scope.list`: confirm the engagement scope.
- `findings.list_for_review`: load pending findings.
- `history.get_safe`: inspect redacted request/response context for a finding.
- `scanner.analyze`: run passive analysis over captured exchanges only, if more context is needed.

Do not use raw history, active replay, or browser/OAST tools unless the user explicitly authorizes that proof step.

## 2. Verdict Standard

Use three outcomes:

- `real`: the evidence proves the issue, or an oracle confirms it.
- `fake`: the evidence contradicts the claim, the behavior is expected, or the finding is noise.
- `uncertain`: the evidence is plausible but incomplete; write the exact missing proof step.

Confidence is only a sorting signal. A high-confidence scanner hypothesis is not a real bug until the evidence supports it.

## 3. Review Loop

For each finding:

1. Read the title, severity, class, affected URL, and evidence.
2. Pull the redacted captured exchange with `history.get_safe` when there is a `history_id`.
3. Identify the claim: authz bypass, secret leak, cache issue, XSS, SSRF/OAST, scanner hypothesis, plugin finding, etc.
4. Compare the claim to the evidence. Look for counterevidence before calling it real.
5. Decide `real`, `fake`, or `uncertain`.
6. If submitting a verdict, call `findings.submit_verdict(finding_id, verdict, note)` with a short proof-based note.

Keep notes compact and redacted. Never paste raw cookies, bearer tokens, API keys, or customer data into a verdict note.

## 4. When Proof Is Missing

If a finding needs active confirmation, propose the smallest safe proof step and stop unless the user authorizes it.

- Authorization / IDOR: use `identity.freshness`, then `oracle.actor_diff` or `oracle.role_matrix`.
- Blind / OOB: use `beacon.poll` for an already-minted payload, or propose `beacon.mint` plus a scoped replay.
- XSS: use `browser.confirm_xss` only on an in-scope page the user approves.
- Race condition: use `repeater.race` only when the user approves a synchronized burst.
- Generic response difference: use `oracle.diff` first because it sends no traffic.

If proof is authorized and confirms the issue, mark `real`. If proof fails cleanly, mark `fake` or `uncertain` depending on coverage.

## 5. Prioritization

Review in this order unless the user says otherwise:

1. Confirmed active proofs and OAST callbacks.
2. Authorization and tenant-boundary findings.
3. Secrets, tokens, and exposed credentials.
4. Cache poisoning / web cache deception indicators.
5. XSS, injection, SSRF, and desync hypotheses.
6. Header/cookie hardening and informational findings.

Escalate severity only when impact evidence supports it. Down-rank noisy hardening issues when exploitability is weak.

## 6. Output Format

When reporting back to the user, group by verdict:

- **Real**: finding id, title, impact, proof summary, next action.
- **Needs Review**: finding id, title, missing evidence, safest proof step.
- **Dismissed**: finding id, title, reason.

If you changed verdicts, include the exact counts submitted: `real`, `fake`, `uncertain`.

## CLI Alternative

Prefer MCP when available. For shell-based agents:

- `crusader findings list --json`
- `crusader findings get <id>`
- `crusader scanner findings --json`
- `crusader scanner run --passive --json`
- `crusader mcp tools`

Use the CLI only for read-only review unless the user explicitly asks for active proof or verdict submission.

## Rules

- Start with `agent.guide`.
- Default to redacted, read-only evidence review.
- Do not mark `real` without proof.
- Do not mark `fake` just because a finding is low severity.
- Do not send traffic unless the user has authorized that proof step and scope is clear.
- Use `uncertain` when proof is missing, and state the missing proof precisely.
