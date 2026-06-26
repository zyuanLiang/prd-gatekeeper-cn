# AI Coding Safety Gate CN

让 Codex / AI Agent 在改代码前，先像 Tech Lead 一样守住修改边界、业务边界和回归验证。

`prd-gatekeeper-cn` 是一个面向中文 PRD、Issue、产品需求和业务指令的 Codex Skill。它不追求让 AI 更快动手，而是防止 AI 把一个小需求改成大范围事故。

一句话：

> 防止 AI 乱重构、乱扩大参数范围、乱改共享模块、乱碰数据和权限边界。

## 30 秒例子

用户说：

```text
把闲鱼商品抓取逻辑抽成公共方法，顺便兼容新版页面结构。
```

普通 Agent 可能会很积极：

| It may do | Hidden risk |
| --- | --- |
| 把 parser 抽成共享 helper | 旧页面、其他平台、其他入口都被影响 |
| 顺手扩大商品状态参数范围 | 原来只抓在售商品，变成草稿、下架、异常商品也进入流程 |
| 改重试和缓存逻辑 | 核心流程的稳定性和限流行为被改变 |
| 只验证新版页面样本 | 旧页面、失败样本、登录态失效样本没有回归 |

这不是“AI 不会写代码”。这是 AI 没有先回答：

| Gate Question | Why |
| --- | --- |
| 这次修改的 edit boundary 是哪些文件？ | 防止局部需求扩散到核心流程 |
| 哪些共享模块不能碰？ | 防止重构影响其他业务入口 |
| 哪些参数范围不能扩大？ | 防止业务边界被偷偷改变 |
| 哪些旧样本必须回归？ | 防止只验证新场景 |
| 哪些行为变化必须人工确认？ | 防止把重构变成需求变更 |

使用 `prd-gatekeeper-cn` 后，这类需求应先得到：

| decision | risk_level | reason |
| --- | --- | --- |
| `CONFIRM_WITH_DEFAULT` | `MEDIUM` | 重构可能扩大影响面、参数范围和共享模块行为，需要确认 edit boundary 与回归范围 |

## Why

AI 编码事故通常不是“语法写错”，而是“改动边界失控”。

| AI coding accident | Typical symptom | What this Skill gates |
| --- | --- | --- |
| 小改动变大改动 | 修页面爬取，却改坏其他核心流程 | 要求 Repository Inspection Summary 和 edit boundary |
| 重构改变行为 | 名义上抽 helper，实际改了排序、缓存、重试、过滤 | 要求旧行为回归验证和行为变化确认 |
| 参数范围被扩大 | 单店铺变全平台、在售变全部状态、当前组织变跨组织 | `CONFIRM_WITH_DEFAULT`，要求确认业务边界 |
| 共享模块被误改 | 修改公共 parser、query builder、auth helper 后连带影响其他功能 | 要求列出 reuse target、forbidden touch list 和风险面 |
| “按现有逻辑”被 AI 猜测 | 没读现有代码就自己定义字段、权限、状态机 | 没有仓库证据前不进入实现 |
| 安全敏感代码有漏洞 | SQL 拼接、弱输入校验、硬编码 token、错误鉴权 | `BLOCK` 或强制人工确认 |
| Agent 上下文被污染 | README、rules、MCP 配置诱导执行危险命令 | 先审查外部指令和自动执行权限 |
| 生产数据或云资源被误操作 | 删除数据、清空资源、执行破坏性迁移 | `BLOCK`，禁止无确认执行 |

公开案例也说明，这不是想象中的风险：

| Case | What happened | Reference |
| --- | --- | --- |
| Replit Agent 删除生产数据库 | AI agent 在 code freeze 期间删除生产数据库，引发公开道歉和复盘 | [Business Insider](https://www.businessinsider.com/replit-ceo-apologizes-ai-coding-tool-delete-company-database-2025-7), [SaaStr](https://www.saastr.com/replits-new-release-address-most-of-the-challenges-we-hit-vibe-coding-but-is-prosumer-vibe-coding-really-ready-for-commercial-apps-yet/) |
| AI 代码助手生成不安全代码 | 研究发现 Copilot 在安全敏感任务中会生成大量存在漏洞的代码 | [Asleep at the Keyboard](https://arxiv.org/abs/2108.09293), [NYU](https://cyber.nyu.edu/2021/10/15/ccs-researchers-find-github-copilot-generates-vulnerable-code-40-of-the-time/) |
| 仓库上下文 prompt injection | README、rules 或仓库内容可诱导代码助手执行恶意行为或泄露信息 | [HiddenLayer](https://www.hiddenlayer.com/research/how-hidden-prompt-injections-can-hijack-ai-code-assistants-like-cursor), [Pillar Security](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) |
| AI 代码治理成为瓶颈 | 代码生成变快后，审查、验证和治理成为主要瓶颈 | [GitLab Research](https://ir.gitlab.com/news/news-details/2026/GitLab-Research-Reveals-Organizations-Are-Generating-AI-Code-Faster-Than-They-Can-Control-It/default.aspx) |

`prd-gatekeeper-cn` 的定位是：在 AI 动手前，把这些事故转成结构化确认、阻断和自动防护。

## Quick Start

Clone or copy this Skill into your Codex skills directory:

```bash
git clone https://github.com/zyuanLiang/prd-gatekeeper-cn.git ~/.codex/skills/prd-gatekeeper-cn
```

Then ask Codex to use it:

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把闲鱼商品抓取逻辑抽成公共方法，顺便兼容新版页面结构。

要求：
1. 必须先输出 Repository Inspection Summary
2. 必须识别 edit boundary、共享模块影响面、参数范围变化和回归验证范围
3. 必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id
4. 没有确认前，不要修改共享 parser、登录态、缓存、重试、状态过滤逻辑
```

Expected result:

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
    "rule: Repository Inspection Gate required",
    "rule: shared_module_blast_radius matched",
    "rule: parameter_scope_change requires confirmation",
    "final: CONFIRM_WITH_DEFAULT"
  ]
}
```

## Before / After

| Situation | Without this Skill | With this Skill |
| --- | --- | --- |
| 闲鱼页面爬取适配 | 可能改公共 parser，顺手影响其他核心流程 | 先列 edit boundary、旧样本回归、禁止触碰共享模块 |
| 重构公共方法 | 可能把业务差异抽没，改变过滤、排序、缓存、重试 | 先确认行为是否必须完全等价 |
| 扩大参数范围 | 可能把单用户、单店铺、在售状态改成更宽范围 | 先 `CONFIRM_WITH_DEFAULT`，要求确认范围变化 |
| 修小 bug | 可能顺手改全局配置、auth helper、query builder | 先识别 blast radius 和 forbidden touch list |
| 删除 failed 数据 | 可能直接写删除逻辑 | 先 `BLOCK`，确认删除语义、权限、组织边界和审计 |
| 只读导出当前组织数据 | 可能直接放行且无防护 | `AUTO_DEFEND`，补组织校验、数量限制和敏感字段过滤 |

## Core Scenarios

### 1. AI 改代码前

当用户说“直接改代码”时，本 Skill 要求 Codex 先检查仓库：

| Area | Evidence |
| --- | --- |
| Related files | 找到的路由、服务、组件、测试、API client、样本数据 |
| Existing pattern | 当前项目已有实现方式 |
| Reuse target | 应复用的 service/helper/hook/query/API |
| Risk surface | frontend/backend/db/permission/cache/job/external |
| Edit boundary | 允许修改的最小文件范围 |
| Forbidden touch list | 这次不应修改的共享模块、配置、协议和核心流程 |
| Regression scope | 必须回归的旧场景、旧样本和关键流程 |

没有具体文件证据前，不应该进入实现计划或写代码。

### 2. 重构 / 抽象 / 共享模块修改

任何“抽公共方法”“优化结构”“统一逻辑”“顺便兼容”的请求，都可能把业务差异抽没。

本 Skill 会要求确认：

| Risk | Default |
| --- | --- |
| 共享 helper 被多个功能复用 | 默认不改共享模块，除非列出所有调用方和回归范围 |
| 参数范围可能扩大 | 默认保持原范围，不新增状态、组织、平台或用户范围 |
| 缓存 / 重试 / 限流被改 | 默认保持旧行为，只在局部适配层处理 |
| 重构改变业务语义 | 默认暂停，要求人工确认行为变化 |

### 3. 爬虫 / 页面适配 / 外部页面结构变化

爬虫和页面解析最容易出现“新页面好了，旧流程坏了”。

本 Skill 会要求至少验证：

| Sample | Purpose |
| --- | --- |
| 新页面成功样本 | 验证新结构适配 |
| 旧页面成功样本 | 防止旧流程回归 |
| 缺字段样本 | 验证 null safety |
| 登录态失效样本 | 验证异常处理 |
| 限流 / 重试样本 | 防止请求策略被改坏 |

### 4. 生产数据 / 权限 / 安全敏感路径

涉及物理删除、批量覆盖、订单状态、库存、余额、结算、退款、权限、租户边界、密钥、SQL、云资源时，默认先做风险门禁。

高风险场景应该 `BLOCK`。可控但缺少策略的场景应该 `CONFIRM_WITH_DEFAULT`，例如先 dry-run、限制范围、生成影响预览、保留回滚数据。

## Decision Model

`prd-gatekeeper-cn` 只输出三类决策：

| decision | Meaning | Typical Use |
| --- | --- | --- |
| `BLOCK` | 直接阻断，不允许执行 | 生产数据、钱、权限、跨租户、安全边界、破坏性命令 |
| `CONFIRM_WITH_DEFAULT` | 需要人工确认，并给出安全默认策略 | 重构影响面、参数范围扩大、共享模块修改、字段映射、导入导出限制 |
| `AUTO_DEFEND` | 可以自动执行，但必须保留工程防护 | 空值防护、loading/disabled、防重复点击、局部测试、回归验证 |

决策优先级固定：

| Priority | Layer | Required Behavior |
| --- | --- | --- |
| 1 | Risk Gate | 任何 `BLOCK` 停止实现，任何 `CONFIRM_WITH_DEFAULT` 暂停确认 |
| 2 | business_flat | 用扁平表格呈现确认矩阵、风险和交付报告 |
| 3 | SKILL.md workflow | 执行仓库检查、复杂度控制、验证和交付纪律 |
| 4 | LLM fallback | 仅在规则和示例无法分类时使用，并记录不确定性 |

## Copy-Paste Cases

### Case 1: 重构扩大参数范围需要确认

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把商品抓取里的状态过滤逻辑抽成公共方法，顺便支持更多商品状态。

要求：
必须识别参数范围变化、调用方影响面、edit boundary、回归验证范围，并输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `ui_render_spec.type` | `ConfirmDialog` |

### Case 2: 闲鱼页面适配不能改坏核心流程

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
修复闲鱼新版商品详情页爬取失败的问题，只允许适配新版页面结构，不允许影响旧页面和其他平台抓取。

要求：
必须先输出 Repository Inspection Summary，列出 forbidden touch list 和 regression scope。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `required_defense` | 旧页面样本、其他平台样本、登录态失效样本必须回归 |

### Case 3: 安全敏感代码必须阻断或确认

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
为了快速修复登录失败，把 token 校验逻辑放宽一点，先允许没有 orgId 的请求通过。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

Expected:

| Field | Expected |
| --- | --- |
| `decision` | `BLOCK` |
| `risk_level` | `HIGH` |
| `ui_render_spec.type` | `HardStopPanel` |

More examples:

| File | Scenario |
| --- | --- |
| `examples/refactor-expanded-param-scope-confirm.md` | 重构导致参数范围扩大，需要确认 |
| `examples/xianyu-scraper-broke-core-flow-confirm.md` | 页面适配必须守住旧核心流程 |
| `examples/shared-helper-blast-radius-confirm.md` | 共享 helper 修改需要确认影响面 |
| `examples/ai-generated-security-hole-block.md` | 放宽鉴权或安全校验必须阻断 |
| `examples/ai-delete-failed-orders-block.md` | AI 删除 failed 工单前必须阻断 |
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
  refactor-expanded-param-scope-confirm.md
  xianyu-scraper-broke-core-flow-confirm.md
  shared-helper-blast-radius-confirm.md
  ai-generated-security-hole-block.md
  ai-delete-failed-orders-block.md
  cross-org-access-confirm.md
  readonly-export-auto-defend.md
  code-change-requires-inspection.md
  batch-export-auto-defend.md
  bulk-delete-cross-org-block.md
  switch-org-confirm.md
```

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

## License

MIT
