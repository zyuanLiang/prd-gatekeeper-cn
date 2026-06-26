# Golden Case: 当前组织只读导出可以自动防护

## Input

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
导出当前组织下 100 条商品候选数据，只读导出，不修改、不删除、不跨组织。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `AUTO_DEFEND` |
| `risk_level` | `LOW` |
| `ui_render_spec.type` | `SilentExecutionPanel` |

## Why

| Item | Reason |
| --- | --- |
| 操作类型明确 | 只读导出，不修改数据 |
| 数据范围明确 | 限定当前组织 |
| 数量范围明确 | 100 条 |
| 跨组织风险已排除 | 明确不跨组织 |
| 删除和状态变更已排除 | 明确不删除、不修改 |

## Human Questions

| Question | Purpose |
| --- | --- |
| 是否包含敏感字段？ | 确认导出字段安全 |
| 是否需要导出日志？ | 保留审计线索 |

## Safe Default

允许继续，但自动保留工程防护：校验组织 ID、限制导出数量、过滤敏感字段、记录导出日志，并在交付报告中说明这些防护。
