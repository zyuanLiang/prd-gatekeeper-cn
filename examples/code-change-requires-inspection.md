# Golden Case: 代码修改必须先检查仓库

## Prompt

```text
请使用 prd-gatekeeper-cn 审核并实现下面需求：

需求：
给订单列表增加批量导出功能，勾选就导出勾选订单，没勾选就按当前筛选全部导出，字段和页面一致。

要求：
1. 先输出 Repository Inspection Summary
2. 没有检查证据前不要写代码
3. 复用现有模式，保持最小改动
4. 修改后运行测试或构建
```

## Wrong Handling

| Problem | Why It Is Wrong |
| --- | --- |
| 直接新增导出按钮和接口调用 | 没有确认现有列表、导出服务、权限和字段映射 |
| 自己定义导出字段 | `字段和页面一致` 是业务歧义，必须确认可见列、隐藏列和操作列 |
| 把导出逻辑堆到列表组件 | 违反复杂度控制，容易让页面组件承载业务转换逻辑 |
| 不查测试和构建命令 | 交付时无法证明修改没有破坏现有行为 |

## Expected Repository Inspection Summary

| Area | Evidence |
| --- | --- |
| Related files | Concrete order list, export service, API client, permission guard, and test paths found by repository search |
| Existing pattern | Current list filtering, selected-row handling, export request, and error/loading style |
| Reuse target | Existing export helper, order query builder, table column config, API client, or permission predicate |
| Risk surface | Frontend/backend/permission/export size/field mapping/repeated click |
| Edit boundary | Minimal files needed for button wiring, export parameter assembly, service reuse, and focused tests |

## Expected Risk Gate

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| `字段和页面一致` is ambiguous | CONFIRM_WITH_DEFAULT | Export visible business columns only; exclude selection and operation columns unless approved |
| Full filtered export has no row limit | CONFIRM_WITH_DEFAULT | Propose a concrete max row limit or async export policy |
| Duplicate export clicks | AUTO_DEFEND | Add loading/disabled state or debounce |
| Export mapping logic location | AUTO_DEFEND | Keep mapping in a helper/service, not inside the list component |

## Expected Handling

Do not produce an implementation plan until the Repository Inspection Summary has concrete file evidence. If repository files are unavailable, state that and avoid inventing paths or patterns. If any `CONFIRM_WITH_DEFAULT` item remains, pause with a business_flat confirmation matrix before coding.
