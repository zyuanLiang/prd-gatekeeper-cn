# prd-gatekeeper-cn

`prd-gatekeeper-cn` is a Codex Skill for Chinese PRD, requirement, and high-risk engineering request review.

It acts as a Tech Lead gate before an AI agent or Codex starts implementation. The goal is simple: prevent an agent from directly executing dangerous or ambiguous requests such as batch deletion, cross-organization data access, irreversible state changes, external notifications, billing changes, or unclear export behavior.

## What It Solves

When Chinese product or business requests are sent directly to an AI coding agent, the agent may implement before clarifying business risk. This skill forces a review step first:

| Problem | Skill Behavior |
| --- | --- |
| High-risk business decision | Stop with `BLOCK` |
| Ambiguous but safely defaultable behavior | Pause with `CONFIRM_WITH_DEFAULT` |
| Pure engineering safeguard | Proceed with `AUTO_DEFEND` |

## Who It Is For

This skill is for engineering teams using Codex or AI agents to analyze requirements, modify code, generate implementation plans, or automate delivery from Chinese PRDs, Issues, chat requirements, and product notes.

## Outputs

The skill produces stable decisions that are explainable, renderable, and replayable through the ADRS-style contract:

| Field | Meaning |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `execution_plan` | Ordered decision flow |
| `ui_render_spec` | Suggested UI treatment such as `HardStopPanel` |
| `decision_trace` | Matched rules, overrides, assumptions, and final path |
| `replay_id` | Stable identifier for audit or later replay |

## Minimal Prompts

### Example 1: Batch Delete Work Orders -> BLOCK

```text
使用 $prd-gatekeeper-cn 审查这个需求：

后台工单列表加一个批量删除按钮，运营选中多个工单后直接删除，最好今天上线。
```

Expected decision: `BLOCK`

Why: physical deletion, permission scope, audit behavior, and irreversible data loss must be human-decided before implementation.

### Example 2: Batch Export Data -> AUTO_DEFEND

```text
使用 $prd-gatekeeper-cn 审查这个需求：

在客户列表增加批量导出按钮，只导出当前勾选的客户，字段固定为客户名称、手机号、创建时间。导出接口已经有 5000 行限制，权限沿用当前列表查询权限。
```

Expected decision: `AUTO_DEFEND`

Why: business scope, fields, row limit, and permission boundary are already defined. The agent should still add engineering safeguards such as loading state, duplicate-click prevention, reuse of list permission checks, and keeping export mapping outside the UI file.

### Example 3: Switch Organization Context -> CONFIRM_WITH_DEFAULT

```text
使用 $prd-gatekeeper-cn 审查这个需求：

用户切换组织后，页面数据也要跟着切过去。旧组织页面缓存怎么处理先按你觉得合理的来。
```

Expected decision: `CONFIRM_WITH_DEFAULT`

Why: cross-organization visibility is sensitive, but a safe proposed default can be reviewed first: clear organization-scoped UI cache, refetch all organization-bound queries, and keep server-side organization filtering as final authority.

## Version Scope

`v0.1.0` is a Skill-only MVP.

| In Scope | Out of Scope |
| --- | --- |
| Codex Skill package | Runtime engine |
| Chinese PRD and requirement gatekeeping | Database |
| Risk Gate, business_flat, and delivery report references | Real replay storage |
| ADRS-style output contract | CLI |
| Explainable decision trace | Platform registry |

## Repository Layout

```text
SKILL.md
agents/openai.yaml
references/
  business-flat-templates.md
  delivery-report-examples.md
  risk-gate-examples.md
  runtime-architecture.md
examples/
  bulk-delete-cross-org-block.md
  batch-export-auto-defend.md
  switch-org-confirm.md
```

## Install

Clone or copy this repository into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex to use `$prd-gatekeeper-cn` when reviewing a Chinese PRD, Issue, or product request before implementation.
