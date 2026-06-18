# prd-gatekeeper-cn

`prd-gatekeeper-cn` 是一个面向中文研发场景的 Codex Skill。

它不是 PRD 生成器，也不是完整运行时系统。它的作用是在 Codex、AI Agent 或研发流程准备执行需求之前，先对需求做风险审查，并输出结构化决策结果。

一句话：

> 它是 AI 执行前的风险网关。

## 适合放在哪些动作之前

- PRD 评审
- 需求拆解
- Codex 自动改代码前
- AI Agent 自动执行前
- 批量数据操作前
- 跨组织 / 多租户数据访问前
- 删除、覆盖、状态变更等高风险操作前

核心目标：

> 防止 AI 或研发流程在需求不清楚、权限不明确、数据边界不清、风险不可逆的情况下直接执行。

## 它解决什么问题

真实研发里，很多需求表面很小：

- 批量删除 200 条工单
- 导出 100 条商品数据
- 切换组织查看数据
- 覆盖导入一批 Excel
- 自动发布商品
- 修复线上脏数据
- 让 AI Agent 自动处理异常任务

但这些需求背后可能隐藏风险：

- 是否会影响其他组织的数据？
- 是否是物理删除？
- 是否可恢复？
- 是否需要审计日志？
- 是否会绕过权限？
- 是否会触发不可逆状态变更？
- 是否需要业务负责人确认？
- 是否应该默认阻断，而不是继续执行？

`prd-gatekeeper-cn` 会把这些隐含风险显式化，并给出三类决策：

```text
BLOCK                  直接阻断，不允许执行
CONFIRM_WITH_DEFAULT   需要确认，并给出安全默认策略
AUTO_DEFEND            可以自动执行，但需要保留工程防护
```

## 用了以后会发生什么

Skill 会输出 ADRS-style contract，让决策可解释、可渲染、可回放。这里的 ADRS 是输出协议，不代表当前 Skill 已经包含生产级运行时、数据库、真实执行器或审计存储：

| Field | Meaning |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `execution_plan` | 决策执行路径 |
| `ui_render_spec` | UI 呈现方式，例如 `HardStopPanel` |
| `decision_trace` | 命中的规则、覆盖关系、假设和最终路径 |
| `replay_id` | 用于审计或后续回放的稳定标识 |

## 典型使用场景

### 场景 1：PRD / 需求进入开发前

当你拿到一个需求，不确定它是否有隐藏风险时，可以先让 Skill 审核。

```text
请使用 prd-gatekeeper-cn skill 审核下面这个需求。

需求：
我要批量删除 200 条工单，其中可能包含其他组织的数据。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

预期结果：

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

这个结果表示：需求不能直接进入开发或执行，必须先确认权限、组织边界、删除方式和审计要求。

### 场景 2：Codex 自动改代码前

当你准备让 Codex 直接改代码时，可以先让 Skill 判断这次修改是否安全，并强制它先检查仓库。没有代码检查证据前，不应该进入实现计划或写代码。

```text
请使用 prd-gatekeeper-cn skill 审核这次代码修改请求。

请求：
把工单表里状态为 failed 的数据全部删除，避免页面展示异常。

要求：
1. 先输出 Repository Inspection Summary
2. 没有检查证据前不要写代码
3. 复用现有模式，保持最小改动
4. 修改后运行测试或构建
```

可能输出：

```json
{
  "decision": "BLOCK",
  "risk_level": "HIGH",
  "decision_trace": [
    "rule: Risk Gate.BLOCK.physical_deletion matched",
    "rule: Risk Gate.BLOCK.irreversible_status_or_data_change matched",
    "final: BLOCK"
  ]
}
```

说明：这个需求不能直接做“删除”。更安全的方案可能是隐藏展示、标记为 archived、软删除、保留审计日志，或增加恢复窗口。

### 场景 2.1：要求 AI 先看代码再实现

如果需求允许继续实现，Skill 的输出顺序应该是：

1. 需求归一化
2. 仓库检查证据
3. 风险判断：`BLOCK` / `CONFIRM_WITH_DEFAULT` / `AUTO_DEFEND`
4. 如果允许继续，再给实现计划
5. 修改后输出验证和交付报告

推荐 prompt：

```text
请使用 prd-gatekeeper-cn 审核并实现下面需求：
{需求内容}

要求：
1. 先输出 Repository Inspection Summary
2. 没有检查证据前不要写代码
3. 复用现有模式，保持最小改动
4. 修改后运行测试或构建
```

`Repository Inspection Summary` 至少应该包含：

| Area | Evidence |
| --- | --- |
| Related files | 找到的相关文件路径 |
| Existing pattern | 当前代码已有实现方式 |
| Reuse target | 应复用的 service/helper/component/hook/API |
| Risk surface | 涉及前端、后端、DB、权限、缓存、任务或外部系统 |
| Edit boundary | 预计最小修改范围 |

### 场景 3：AI Agent 自动执行业务动作前

如果系统里有 AI 店长、自动运维 Agent、自动审核 Agent，在它准备执行动作前，可以先过 Gatekeeper。

```text
请使用 prd-gatekeeper-cn skill 审核下面的 Agent 动作。

Agent 准备：
自动发布 20 个商品，并跳过人工确认。

上下文：
商品素材已生成，但价格建议部分失败，部分商品缺少主图。
```

可能输出：

```json
{
  "decision": "CONFIRM_WITH_DEFAULT",
  "risk_level": "MEDIUM",
  "ui_render_spec": {
    "type": "ConfirmDialog",
    "theme": "yellow",
    "interaction_mode": "user_confirm"
  }
}
```

含义：这类动作不是绝对不能执行，但不应该完全自动跳过人工确认。默认策略可以是缺主图的不发布、价格建议失败的不发布、仅发布素材完整且价格完整的商品，并要求用户确认最终发布列表。

### 场景 4：批量导出 / 只读操作

不是所有批量操作都应该阻断。如果是只读导出、当前组织内、无权限越界、无数据修改，可以自动放行。

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
导出当前组织下 100 条商品候选数据，只读导出，不修改、不删除、不跨组织。
```

预期输出：

```json
{
  "decision": "AUTO_DEFEND",
  "risk_level": "LOW",
  "ui_render_spec": {
    "type": "SilentExecutionPanel",
    "theme": "green",
    "interaction_mode": "auto_execute"
  }
}
```

含义：可以继续执行，但仍然建议保留基础工程防护，例如限制导出范围、保留操作日志、校验组织 ID、限制导出数量、避免导出敏感字段。

### 场景 5：切换组织 / 多租户数据隔离

多组织、多租户系统里，“切换组织查看数据”是非常典型的灰色场景。

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
用户要从当前组织切换到另一个组织查看数据，但当前请求没有提供明确授权信息。
```

预期输出：

```json
{
  "decision": "CONFIRM_WITH_DEFAULT",
  "risk_level": "MEDIUM",
  "ui_render_spec": {
    "type": "ConfirmDialog",
    "theme": "yellow",
    "interaction_mode": "user_confirm"
  }
}
```

含义：不应该直接放行，也不一定直接 `BLOCK`。更合理的默认策略是要求确认授权、校验用户是否属于目标组织、校验跨组织查看权限、默认不展示目标组织数据，并记录切换日志。

### 场景 6：Excel 导入 / 数据覆盖

Excel 导入经常看起来是普通功能，但实际可能覆盖生产数据。

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
运营上传一个 Excel，系统根据 Excel 内容批量覆盖已有商品价格、库存和标题。
```

可能输出：

```json
{
  "decision": "CONFIRM_WITH_DEFAULT",
  "risk_level": "MEDIUM",
  "ui_render_spec": {
    "type": "ConfirmDialog",
    "theme": "yellow",
    "interaction_mode": "user_confirm"
  },
  "decision_trace": [
    "rule: batch_update matched",
    "rule: data_overwrite matched",
    "final: CONFIRM_WITH_DEFAULT"
  ]
}
```

建议默认策略：先 dry-run 预览影响范围，展示将被修改的数据数量和字段 diff，禁止直接覆盖主键、组织 ID、状态机字段，保留导入记录和回滚文件，用户确认后再执行。

### 场景 7：线上问题快速修复

线上问题修复时，研发很容易为了快而直接写 SQL 或改逻辑。这个 Skill 适合做上线前风险提醒。

```text
请使用 prd-gatekeeper-cn skill 审核下面修复方案。

方案：
线上有 300 条异常订单状态错误，准备直接把它们全部更新为 completed。
```

可能输出：

```json
{
  "decision": "CONFIRM_WITH_DEFAULT",
  "risk_level": "MEDIUM",
  "decision_trace": [
    "rule: batch_status_change matched",
    "rule: irreversible_status_or_data_change matched",
    "final: CONFIRM_WITH_DEFAULT"
  ]
}
```

建议默认策略：先导出受影响数据，明确筛选条件，做 dry-run 统计，确认是否影响财务、履约、售后、审计，保留回滚 SQL，小批量执行。如果状态变更不可逆，或者影响跨组织数据，则应升级为 `BLOCK`。

### 场景 8：需求太大、范围不清

这个 Skill 也可以用来识别“需求没有收口”的情况。

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
重构整个商品生产线，让 AI 自动完成选品、素材、文案、发布和异常处理。
```

可能输出：

```json
{
  "decision": "CONFIRM_WITH_DEFAULT",
  "risk_level": "MEDIUM",
  "decision_trace": [
    "rule: scope_too_large matched",
    "rule: automation_boundary_unclear matched",
    "final: CONFIRM_WITH_DEFAULT"
  ]
}
```

建议默认策略：拆成多个阶段，先定义不可自动执行的动作，先做只读分析和 dry-run，再做人工确认后的半自动执行，最后再考虑全自动闭环。

## 三类决策含义

### BLOCK

表示：不能继续执行。

适用情况：

- 批量物理删除
- 跨组织 / 跨租户数据操作
- 不可逆状态变更
- 权限不明确
- 可能破坏审计链路
- 可能影响其他用户或组织
- 可能让 AI 越权执行

UI 表现：

```json
{
  "type": "HardStopPanel",
  "theme": "red",
  "interaction_mode": "blocked"
}
```

系统行为：不执行、不生成代码修改、不进入自动化流程，要求补充确认信息，并必须有负责人确认。

### CONFIRM_WITH_DEFAULT

表示：可以继续讨论，但不能直接执行。

适用情况：

- 需求存在灰色风险
- 上下文缺失
- 权限需要确认
- 批量修改但可控
- 可以通过默认安全策略降低风险
- 需要人工确认执行范围

UI 表现：

```json
{
  "type": "ConfirmDialog",
  "theme": "yellow",
  "interaction_mode": "user_confirm"
}
```

系统行为：弹确认，给出默认安全方案，要求用户确认，支持 dry-run 和影响范围预览。

### AUTO_DEFEND

表示：可以自动执行，但必须保留工程防护。

适用情况：

- 只读操作
- 当前组织内
- 不修改数据
- 不删除数据
- 不改变状态
- 有明确输入范围
- 风险较低

UI 表现：

```json
{
  "type": "SilentExecutionPanel",
  "theme": "green",
  "interaction_mode": "auto_execute"
}
```

系统行为：可以继续执行，但要自动加保护、保留日志、限制范围，并输出执行摘要。

## 推荐使用方式

### 方式 1：作为 Codex Skill 使用

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

要求：
1. 必须输出 ADRS Output Contract
2. 必须包含 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id
3. 必须遵守 Risk Gate > business_flat > SKILL.md workflow > LLM fallback 的优先级
4. 不允许跳过 BLOCK / CONFIRM_WITH_DEFAULT / AUTO_DEFEND 判断

需求：
{你的需求内容}
```

### 方式 2：作为需求评审模板使用

```text
请按 prd-gatekeeper-cn 的规则审查这个 PRD。
重点检查：
- 是否有不可逆操作
- 是否有跨组织数据风险
- 是否有批量修改风险
- 是否需要审计日志
- 是否应该默认 BLOCK 或 CONFIRM
```

### 方式 3：作为 AI Agent 执行前置网关

```text
Agent 准备执行动作：
{action}

上下文：
{context}

请使用 prd-gatekeeper-cn 判断是否允许执行。
```

根据结果处理：

```text
BLOCK                  不执行
CONFIRM_WITH_DEFAULT   请求人工确认
AUTO_DEFEND            自动执行并保留日志
```

### 方式 4：作为 AI 编码前的仓库检查闸门

```text
请使用 prd-gatekeeper-cn 审核并实现下面需求：
{需求内容}

要求：
1. 先输出 Repository Inspection Summary
2. 没有检查证据前不要写代码
3. 复用现有模式，保持最小改动
4. 修改后运行测试或构建
```

这个方式适合修 bug、改页面、接接口、重构、补导入导出、调整权限等代码修改场景。纯 PRD 风险评审可以不强制检查仓库。

## 最小验证案例

你可以用下面三个案例快速验证 Skill 是否有效。

### Case 1：高风险批量删除

```text
我要批量删除 200 条工单，其中可能包含其他组织的数据。
```

期望：

```text
decision = BLOCK
risk_level = HIGH
ui_render_spec.type = HardStopPanel
```

### Case 2：安全只读导出

```text
我要导出当前组织下 100 条商品候选数据，只读导出，不修改、不删除、不跨组织。
```

期望：

```text
decision = AUTO_DEFEND
risk_level = LOW
ui_render_spec.type = SilentExecutionPanel
```

### Case 3：切换组织但授权不明

```text
用户要从当前组织切换到另一个组织查看数据，但当前请求没有提供明确授权信息。
```

期望：

```text
decision = CONFIRM_WITH_DEFAULT
risk_level = MEDIUM
ui_render_spec.type = ConfirmDialog
```

## 不适合做什么

`prd-gatekeeper-cn` 不是完整运行时系统，也不是权限系统本身。

它不负责：

- 真实数据库权限校验
- 真实删除 / 导出 / 发布动作
- 真实 replay 存储
- 业务系统鉴权
- 自动修复所有 PRD
- 替代测试用例
- 替代安全审计
- 替代人工最终责任

它更适合作为：

> AI Agent / Codex / 研发流程执行前的结构化风险网关。

## Version Scope

`v0.1.0` is a Skill-first MVP. ADRS fields are protocol outputs for traceability; production runtime, durable replay, UI rendering, and real executors remain out of scope unless explicitly built later.

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
  code-change-requires-inspection.md
```

## Install

Clone or copy this repository into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex to use `$prd-gatekeeper-cn` when reviewing a Chinese PRD, Issue, or product request before implementation.

## 一句话总结

`prd-gatekeeper-cn` 的价值不是让 AI 更会写代码，而是让 AI 在执行需求前更稳、更可控、更可解释。

它适合用来回答：

```text
这个需求能不能直接做？
这个动作能不能让 AI 自动执行？
这个批量操作是否应该阻断？
这个 PRD 里有没有隐藏的数据、权限、审计和不可逆风险？
```

最终目标：

> 在 AI Agent 和真实业务系统之间，加一层可解释、可审计、可渲染的风险决策网关。
