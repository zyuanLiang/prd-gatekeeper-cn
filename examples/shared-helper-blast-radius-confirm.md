# Golden Case: 共享 helper 修改需要确认影响面

## Input

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
把订单列表和商品列表里重复的查询参数组装逻辑抽到一个 shared helper，顺便统一默认分页和排序。

要求：
必须识别共享 helper 调用方、默认参数变化、排序变化、分页变化、回归验证范围，并输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
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
| blast radius 扩大 | shared helper 会影响多个列表、导出和搜索入口 |
| 默认分页变化 | 页面性能、导出结果和兼容性可能改变 |
| 默认排序变化 | 用户看到的数据顺序和业务优先级可能改变 |
| 参数组装语义不一致 | 订单和商品可能有不同权限、状态和字段规则 |

## Safe Default

默认不改变分页、排序、过滤和权限语义。抽 helper 只能消除纯重复代码，任何默认参数变化都必须单独确认。
