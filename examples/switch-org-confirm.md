# Golden Case: Switch Organization Context

## Prompt

```text
使用 $prd-gatekeeper-cn 审查这个需求：

用户切换组织后，页面数据也要跟着切过去。
旧组织页面缓存怎么处理先按你觉得合理的来。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `ui_render_spec` | `ConfirmDialog` |

## Why

| Gap | Proposed Default | Reason | Affected Area | Decision |
| --- | --- | --- | --- | --- |
| Old organization cache behavior is undefined | Clear organization-scoped UI cache on switch | Avoid showing stale cross-organization data | Frontend state and query cache | Needs approval |
| Refetch behavior is undefined | Refetch all organization-bound queries after switch | Keeps visible data aligned with active organization | API query layer | Needs approval |
| Server authority must remain clear | Keep server-side organization filtering as final authority | Frontend cache clearing cannot be the security boundary | Backend API and data access | Needs approval |

## Expected Handling

Pause before implementation. Present the confirmation matrix and wait for explicit approval.
