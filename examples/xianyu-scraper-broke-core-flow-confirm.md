# Golden Case: 闲鱼页面适配必须守住旧核心流程

## Input

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
修复闲鱼新版商品详情页爬取失败的问题，只允许适配新版页面结构，不允许影响旧页面和其他平台抓取。

要求：
必须先输出 Repository Inspection Summary，列出 forbidden touch list、regression scope、decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `ui_render_spec.type` | `ConfirmDialog` |

## Why

| Risk | Reason |
| --- | --- |
| 页面适配影响 parser | 解析逻辑可能被多个页面或平台复用 |
| 旧流程回归风险 | 新页面修好了，旧页面可能被改坏 |
| 登录态和异常处理风险 | 爬虫常依赖登录态、限流、重试和缺字段处理 |
| edit boundary 不清 | Agent 可能顺手修改核心抓取入口或共享 helper |

## Safe Default

默认只新增新版页面适配层，不修改公共 parser、登录态、缓存、重试、状态过滤和其他平台入口。必须回归新页面成功样本、旧页面成功样本、缺字段样本、登录态失效样本和限流样本。
