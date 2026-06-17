# Golden Case: Batch Delete Work Orders

## Prompt

```text
使用 $prd-gatekeeper-cn 审查这个需求：

后台工单列表加一个批量删除按钮，运营选中多个工单后直接删除。
删除时允许按当前组织筛选批量处理，最好今天上线。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `BLOCK` |
| `risk_level` | `HIGH` |
| `ui_render_spec` | `HardStopPanel` |

## Why

| Blocker | Reason | Minimum Question |
| --- | --- | --- |
| Physical deletion | Irreversible data loss and audit risk | Is this soft delete, archive, cancel, or permanent delete? |
| Permission scope | Batch operation may cross role or organization boundaries | Which roles can delete which work orders? |
| Organization boundary | Cross-organization operation may expose or mutate unauthorized data | Must the operation be limited to the active organization only? |
| Audit requirement | Destructive actions usually need traceability | Should operator, timestamp, reason, and affected IDs be recorded? |

## Expected Handling

Stop implementation. Ask only the minimum business, permission, tenant-boundary, and audit questions needed to continue.
