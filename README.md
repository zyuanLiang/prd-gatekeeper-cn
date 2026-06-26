# AI Coding Safety Gate CN

让 Codex / AI Agent 在改代码或执行业务动作前，先像 Tech Lead 一样做风险审查。

`prd-gatekeeper-cn` 是一个面向中文 PRD、Issue、产品需求和业务指令的 Codex Skill。它的目标不是让 AI 更快动手，而是防止 AI 在权限不清、数据边界不清、批量操作不可逆时直接动手。

一句话：

> 防止 AI 乱改业务代码、乱删数据、乱越权执行。

## 30 秒例子

用户说：

```text
把工单表里状态为 failed 的数据全部删除，避免页面展示异常。
```

普通 Agent 可能会直接去找表、改接口、写删除逻辑。

使用 `prd-gatekeeper-cn` 后，期望先得到风险判断：

| decision | risk_level | reason |
| --- | --- | --- |
| `BLOCK` | `HIGH` | 物理删除、状态数据不可逆、可能破坏审计链路 |

它会先追问最小必要问题：

| Question | Why |
| --- | --- |
| 是物理删除、软删除、归档，还是仅隐藏展示？ | 删除语义决定是否可恢复 |
| 是否只影响当前组织的数据？ | 防止跨组织 / 多租户越权 |
| 谁有权限执行这个操作？ | 防止绕过角色和数据权限 |
| 是否需要记录操作人、时间、原因和影响 ID？ | 保留审计与回滚线索 |

这就是这个 Skill 的核心价值：**在 AI 动手前，先把高风险业务决策拦下来。**

## Why

真实研发里，很多需求看起来很小：

- 批量删除 200 条工单
- 覆盖导入一批 Excel
- 切换组织查看数据
- 修复线上脏数据
- 自动发布商品
- 给订单列表增加批量导出
- 让 Agent 自动处理异常任务

但这些需求背后经常藏着高风险问题：

- 是否会影响其他组织的数据？
- 是否是不可恢复的物理删除？
- 是否会绕过权限？
- 是否会改变订单、支付、库存、履约等关键状态？
- 是否需要审计日志和回滚方案？
- 是否应该先 dry-run，而不是直接执行？
- 是否应该要求业务负责人确认？

`prd-gatekeeper-cn` 把这些隐含风险显式化，并输出稳定的决策结果。

## Quick Start

Clone or copy this Skill into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex to use it:

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把工单表里状态为 failed 的数据全部删除，避免页面展示异常。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
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
  },
  "decision_trace": [
    "rule: Risk Gate.BLOCK.physical_deletion matched",
    "rule: Risk Gate.BLOCK.irreversible_status_or_data_change matched",
    "final: BLOCK"
  ]
}
```

## Before / After

| Situation | Without this Skill | With this Skill |
| --- | --- | --- |
| 删除 failed 工单 | 可能直接写删除逻辑 | 先 `BLOCK`，确认删除语义、权限、组织边界和审计 |
| 导出订单字段和页面一致 | 可能自行猜字段 | 先识别字段映射歧义，输出确认矩阵 |
| 切换组织查看数据 | 可能只做前端切换 | 先确认授权、成员关系、跨组织权限和日志 |
| 只读导出当前组织数据 | 可能直接放行且无防护 | `AUTO_DEFEND`，补组织校验、数量限制和敏感字段过滤 |

## Core Scenarios

### 1. AI 改代码前

当用户说“直接改代码”时，本 Skill 要求 Codex 先检查仓库：

| Area | Evidence |
| --- | --- |
| Related files | 找到的路由、服务、组件、测试、API client |
| Existing pattern | 当前项目已有实现方式 |
| Reuse target | 应复用的 service/helper/hook/query/API |
| Risk surface | frontend/backend/db/permission/cache/job/external |
| Edit boundary | 最小计划修改范围 |

没有具体文件证据前，不应该进入实现计划或写代码。

### 2. 批量删除 / 覆盖 / 状态变更

涉及物理删除、批量覆盖、订单状态、库存、余额、结算、退款、发布等动作时，默认先做风险门禁。

高风险场景应该 `BLOCK`。可控但缺少策略的场景应该 `CONFIRM_WITH_DEFAULT`，例如先 dry-run、限制范围、生成影响预览、保留回滚数据。

### 3. 跨组织 / 权限 / 多租户边界

任何“切换组织”“查看其他团队数据”“按原权限处理”的需求，都不能靠 Agent 猜。

本 Skill 会要求明确：

| Risk | Default |
| --- | --- |
| 目标组织授权缺失 | 默认不展示目标组织数据 |
| 用户成员关系不明确 | 校验用户是否属于目标组织 |
| 跨组织查看权限不明确 | 要求确认权限策略 |
| 日志要求不明确 | 默认记录切换日志 |

### 4. 复杂 PRD 进入开发前

当需求范围太大、验收不清、业务规则散落在聊天里，本 Skill 会先做需求归一化：

| Field | Required Content |
| --- | --- |
| Goal | 用户可见结果 |
| Scope | 本次包含行为 |
| Out of Scope | 明确不做或延期 |
| Business Rules | 权限、数据边界、状态规则、字段规则 |
| Acceptance | 可观察验收条件 |
| Unknowns | 缺失决策、歧义、冲突 |

## Decision Model

`prd-gatekeeper-cn` 只输出三类决策：

| decision | Meaning | Typical Use |
| --- | --- | --- |
| `BLOCK` | 直接阻断，不允许执行 | 钱、权限、跨租户、物理删除、不可逆状态、合规安全 |
| `CONFIRM_WITH_DEFAULT` | 需要人工确认，并给出安全默认策略 | 字段映射、导入导出限制、分页、重试、兼容性、范围不清 |
| `AUTO_DEFEND` | 可以自动执行，但必须保留工程防护 | 空值防护、loading/disabled、防重复点击、复用 helper、加测试 |

决策优先级固定：

| Priority | Layer | Required Behavior |
| --- | --- | --- |
| 1 | Risk Gate | 任何 `BLOCK` 停止实现，任何 `CONFIRM_WITH_DEFAULT` 暂停确认 |
| 2 | business_flat | 用扁平表格呈现确认矩阵、风险和交付报告 |
| 3 | SKILL.md workflow | 执行仓库检查、复杂度控制、验证和交付纪律 |
| 4 | LLM fallback | 仅在规则和示例无法分类时使用，并记录不确定性 |

## Copy-Paste Cases

### Case 1: 高风险删除应该阻断

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把工单表里状态为 failed 的数据全部删除，避免页面展示异常。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `BLOCK` |
| `risk_level` | `HIGH` |
| `ui_render_spec.type` | `HardStopPanel` |

### Case 2: 只读导出可以自动防护

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
导出当前组织下 100 条商品候选数据，只读导出，不修改、不删除、不跨组织。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `AUTO_DEFEND` |
| `risk_level` | `LOW` |
| `ui_render_spec.type` | `SilentExecutionPanel` |

### Case 3: 跨组织授权不明需要确认

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
用户要从当前组织切换到另一个组织查看数据，但当前请求没有提供明确授权信息。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `ui_render_spec.type` | `ConfirmDialog` |

More examples:

| File | Scenario |
| --- | --- |
| `examples/ai-delete-failed-orders-block.md` | AI 删除 failed 工单前必须阻断 |
| `examples/cross-org-access-confirm.md` | 跨组织访问授权不明需要确认 |
| `examples/readonly-export-auto-defend.md` | 当前组织只读导出可以自动防护 |
| `examples/code-change-requires-inspection.md` | 代码修改前必须先检查仓库 |

## Advanced: ADRS Output Contract

所有输出必须包含：

| Field | Meaning |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `execution_plan` | 决策执行路径 |
| `ui_render_spec` | UI 呈现方式，例如 `HardStopPanel` |
| `decision_trace` | 命中的规则、覆盖关系、假设和最终路径 |
| `replay_id` | 用于审计或后续回放的稳定标识 |

UI render mapping:

| decision | UI |
| --- | --- |
| `BLOCK` | `HardStopPanel` red interrupt flow |
| `CONFIRM_WITH_DEFAULT` | `ConfirmDialog` yellow user confirmation |
| `AUTO_DEFEND` | `SilentExecutionPanel` green audit log |

Execution path:

```text
validate_input ->
apply_business_flat ->
apply_risk_gate ->
resolve_decision ->
emit_action
```

## Repository Layout

```text
SKILL.md
README.md
README.en.md
agents/openai.yaml
references/
  business-flat-templates.md
  delivery-report-examples.md
  risk-gate-examples.md
  runtime-architecture.md
examples/
  ai-delete-failed-orders-block.md
  cross-org-access-confirm.md
  readonly-export-auto-defend.md
  code-change-requires-inspection.md
  batch-export-auto-defend.md
  bulk-delete-cross-org-block.md
  switch-org-confirm.md
```

The last three files are earlier golden cases kept for compatibility with existing links and local tests.

## Version Scope

`v0.1.0` is a Skill-first MVP.

| In Scope | Out of Scope |
| --- | --- |
| Codex Skill package | MCP server |
| Chinese PRD and requirement gatekeeping | CLI |
| Repository inspection discipline | Runtime engine |
| Risk Gate, business_flat, ADRS output contract | Database or durable replay storage |
| Explainable decision trace | Real permission enforcement |

This Skill is not a PRD generator, not a complete runtime, and not a replacement for authorization, testing, audit, or human business ownership. It is a safety gate before AI-assisted implementation or execution.

## Recommended GitHub Topics

```text
codex-skill
ai-agent
ai-coding
prd-review
risk-gate
requirements-engineering
chinese
```

## v0.1.0 Release Checklist

| Item | Status |
| --- | --- |
| Clear first-screen positioning | Done |
| Copy-paste quick start | Done |
| Three golden cases | Done |
| MIT License | Done |
| English README | Done |
| Core Risk Gate semantics unchanged | Done |

## License

MIT
