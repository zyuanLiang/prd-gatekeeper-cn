# AI Coding Safety Gate CN

`prd-gatekeeper-cn` is a Codex Skill for Chinese PRDs, issues, product requests, and business instructions.

It makes Codex / AI agents pause before changing code or executing risky business actions. The main risk is not that AI cannot write code; the main risk is that it expands a small change into a large behavioral accident.

## 30-Second Example

User request:

```text
Extract the ecommerce product-detail scraping logic into a shared helper and make it compatible with the new page structure.
```

A normal agent may eagerly:

| It may do | Hidden risk |
| --- | --- |
| Extract a shared parser | Old pages, other platforms, and other entrypoints may break |
| Expand product status filters | Draft, removed, or abnormal items may enter a flow that used to handle active items only |
| Change retry or cache logic | Stability and rate-limit behavior may change |
| Test only the new page sample | Old pages, failure samples, and expired-login samples are not verified |

With this Skill, the expected first result is:

| decision | risk_level | reason |
| --- | --- | --- |
| `CONFIRM_WITH_DEFAULT` | `MEDIUM` | Refactor may expand blast radius, parameter scope, and shared-module behavior |

## Why

AI coding accidents are often boundary failures, not syntax failures.

| AI coding accident | Typical symptom | What this Skill gates |
| --- | --- | --- |
| Small change becomes large change | Fixing one scraper page breaks other core flows | Repository inspection and edit boundary |
| Refactor changes behavior | Helper extraction changes filtering, sorting, cache, or retries | Regression scope and behavior confirmation |
| Parameter scope expands | Single shop becomes all shops, active-only becomes all statuses | Confirmation before business-boundary changes |
| Shared module is modified | Public parser, query builder, or auth helper affects unrelated features | Blast-radius review and forbidden touch list |
| Existing logic is guessed | The agent invents fields, permissions, or state rules without reading code | Repository evidence before implementation |
| Security-sensitive code is weakened | SQL injection, weak validation, hardcoded tokens, incorrect auth | `BLOCK` or explicit approval |
| Repository context is hostile | README/rules/MCP config injects hidden instructions | Review external instructions before execution |
| Production data or cloud resources are touched | Data deletion, destructive migration, resource cleanup | `BLOCK` before execution |

## Quick Start

Clone or copy this Skill into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex:

```text
Use the prd-gatekeeper-cn skill to review this request.

Request:
Extract the ecommerce product-detail scraping logic into a shared helper and make it compatible with the new page structure.

Requirements:
First output Repository Inspection Summary. Identify edit boundary, shared-module blast radius, parameter-scope changes, regression scope, decision, risk_level, execution_plan, ui_render_spec, decision_trace, and replay_id.
```

Expected: `CONFIRM_WITH_DEFAULT`, `MEDIUM`, `ConfirmDialog`.

## Decision Model

| decision | Meaning |
| --- | --- |
| `BLOCK` | Stop. The agent must not implement or execute until human decisions are supplied. |
| `CONFIRM_WITH_DEFAULT` | Pause with a safe default and wait for explicit approval. |
| `AUTO_DEFEND` | Continue, but apply engineering safeguards and report them. |

## Copy-Paste Cases

| File | Scenario |
| --- | --- |
| `examples/refactor-expanded-param-scope-confirm.md` | Refactor expands parameter scope |
| `examples/ecommerce-scraper-core-flow-confirm.md` | Scraper page adaptation may break old core flows |
| `examples/shared-helper-blast-radius-confirm.md` | Shared helper changes need blast-radius review |
| `examples/ai-generated-security-hole-block.md` | Weakening auth or security checks must be blocked |

## Output Contract

All outputs must include:

| Field | Meaning |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `execution_plan` | Decision execution path |
| `ui_render_spec` | Render target such as `HardStopPanel` |
| `decision_trace` | Matched rules, overrides, assumptions, and final path |
| `replay_id` | Stable ID for audit or replay |

## Scope

`v0.1.0` is a Skill-first MVP.

In scope: Codex Skill package, Chinese PRD gatekeeping, repository inspection discipline, Risk Gate, business_flat reports, ADRS-style output contract.

Out of scope: MCP server, CLI, runtime engine, database, durable replay storage, real permission enforcement, replacing tests or security audit.

## License

MIT
