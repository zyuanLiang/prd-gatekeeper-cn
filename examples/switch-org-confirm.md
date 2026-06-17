# Golden Case: 切换组织但授权不明

## Prompt

```text
请使用 prd-gatekeeper-cn skill 审核下面需求。

需求：
用户要从当前组织切换到另一个组织查看数据，但当前请求没有提供明确授权信息。

要求：
必须输出 decision、risk_level、execution_plan、ui_render_spec、decision_trace、replay_id。
```

## Expected Decision

| Field | Expected Value |
| --- | --- |
| `decision` | `CONFIRM_WITH_DEFAULT` |
| `risk_level` | `MEDIUM` |
| `ui_render_spec.type` | `ConfirmDialog` |

## Why

| Gap | Proposed Default | Reason | Affected Area | Decision |
| --- | --- | --- | --- | --- |
| 目标组织授权信息缺失 | 默认不展示目标组织数据 | 防止越权访问 | Backend API / frontend route guard | Needs approval |
| 用户组织成员关系不明确 | 校验用户是否属于目标组织 | 多租户隔离不能靠前端假设 | Auth / tenant boundary | Needs approval |
| 跨组织查看权限不明确 | 要求确认是否具备跨组织查看权限 | 防止扩大数据可见范围 | Permission policy | Needs approval |
| 切换日志要求不明确 | 默认记录切换日志 | 保留审计线索 | Audit / event log | Needs approval |

## Expected Handling

Pause before implementation. Present the confirmation matrix and wait for explicit approval before changing code or allowing the action.
