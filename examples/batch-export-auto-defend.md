# Golden Case: Batch Export Data

## Prompt

```text
使用 $prd-gatekeeper-cn 审查这个需求：

在客户列表增加批量导出按钮，只导出当前勾选的客户。
字段固定为客户名称、手机号、创建时间。
导出接口已经有 5000 行限制，权限沿用当前列表查询权限。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `AUTO_DEFEND` |
| `risk_level` | `LOW` |
| `ui_render_spec` | `SilentExecutionPanel` |

## Why

| Item | Classification | Reason |
| --- | --- | --- |
| Export scope | Defined | Only selected rows are exported |
| Export fields | Defined | Field list is explicit |
| Row limit | Defined | Existing API limit is 5000 rows |
| Permission boundary | Defined | Reuse current list query permission |
| Duplicate click | AUTO_DEFEND | Add loading or disabled state |
| Mapping location | AUTO_DEFEND | Keep export field mapping outside the UI component |

## Expected Handling

Proceed with implementation safeguards. Report the safeguards in the final delivery report.
