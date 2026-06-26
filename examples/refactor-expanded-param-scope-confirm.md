# Golden Case: 重构导致参数范围扩大需要确认

## Input

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把商品抓取里的状态过滤逻辑抽成公共方法，顺便支持更多商品状态。

要求：
必须识别参数范围变化、调用方影响面、edit boundary、回归验证范围，并输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
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
| 参数范围扩大 | 旧逻辑可能只处理在售商品，新逻辑可能引入草稿、下架或异常状态 |
| 调用方影响面未知 | 公共方法会影响多个抓取入口或后续处理流程 |
| 重构改变行为 | 名义上是抽象，实际可能改变过滤、排序、缓存或重试 |
| 回归范围不明 | 需要确认旧状态、旧页面和失败样本是否回归 |

## Safe Default

默认保持原参数范围不变。新增状态支持必须作为显式业务变更确认，并列出所有调用方和回归样本。
