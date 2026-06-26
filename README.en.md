# AI Coding Safety Gate CN

`prd-gatekeeper-cn` is a Codex Skill for Chinese PRDs, issues, product requests, and business instructions.

It makes Codex / AI agents pause before changing code or executing risky business actions. The goal is simple: prevent AI from deleting data, bypassing permissions, or making irreversible changes when the requirement is ambiguous.

## 30-Second Example

User request:

```text
Delete every work order whose status is failed so the page no longer shows errors.
```

A normal agent may start editing code immediately.

With this Skill, the expected first result is:

| decision | risk_level | reason |
| --- | --- | --- |
| `BLOCK` | `HIGH` | Physical deletion, irreversible data change, possible audit damage |

The agent should ask focused questions before implementation:

| Question | Why |
| --- | --- |
| Is this physical delete, soft delete, archive, or hide-only? | Deletion semantics decide recoverability |
| Is the operation limited to the current organization? | Prevent cross-tenant data damage |
| Which role can perform this action? | Prevent permission bypass |
| Should operation logs record actor, time, reason, and affected IDs? | Preserve audit and rollback context |

## Quick Start

Clone or copy this Skill into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex:

```text
Use the prd-gatekeeper-cn skill to review this request.

Request:
Delete every work order whose status is failed so the page no longer shows errors.

Requirements:
Output decision, risk_level, execution_plan, ui_render_spec, decision_trace, and replay_id.
```

Expected result:

```json
{
  "decision": "BLOCK",
  "risk_level": "HIGH",
  "ui_render_spec": {
    "type": "HardStopPanel",
    "theme": "red",
    "interaction_mode": "blocked"
  }
}
```

## When To Use

| Scenario | What the Skill enforces |
| --- | --- |
| Before Codex changes business code | Repository inspection before implementation |
| Batch delete, overwrite, or status changes | Risk Gate classification before execution |
| Cross-organization or tenant access | Permission and data-boundary confirmation |
| Ambiguous PRDs entering development | Requirement normalization and confirmation matrix |
| Low-risk engineering safeguards | Automatic null safety, loading state, reuse, and tests |

## Decision Model

| decision | Meaning |
| --- | --- |
| `BLOCK` | Stop. The agent must not implement or execute until human decisions are supplied. |
| `CONFIRM_WITH_DEFAULT` | Pause with a safe default and wait for explicit approval. |
| `AUTO_DEFEND` | Continue, but apply engineering safeguards and report them. |

Decision priority:

| Priority | Layer |
| --- | --- |
| 1 | Risk Gate |
| 2 | business_flat confirmation/report tables |
| 3 | Skill workflow |
| 4 | LLM fallback |

## Copy-Paste Cases

### Case 1: Delete failed work orders

Expected: `BLOCK`, `HIGH`, `HardStopPanel`.

### Case 2: Read-only export within current organization

Expected: `AUTO_DEFEND`, `LOW`, `SilentExecutionPanel`.

### Case 3: Switch organization without authorization context

Expected: `CONFIRM_WITH_DEFAULT`, `MEDIUM`, `ConfirmDialog`.

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
