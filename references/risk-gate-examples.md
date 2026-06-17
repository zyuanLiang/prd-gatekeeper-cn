# Risk Gate Examples

Use these examples to classify common Chinese product and engineering requests. The goal is to separate human business decisions from automatic engineering defense.

## Excel Export

User request: `订单列表加批量导出，勾选就导出勾选的，没勾选就按当前筛选全部导出，字段和页面一致，明天上线。`

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| Full export has no row limit | CONFIRM_WITH_DEFAULT | Propose sync export max 10000 rows; ask approval |
| `字段和页面一致` is ambiguous | CONFIRM_WITH_DEFAULT | Export visible business columns only; exclude selection and operation columns |
| Duplicate export clicks | AUTO_DEFEND | Add loading/disabled state or debounce |
| Export mapping logic location | AUTO_DEFEND | Extract to service/helper instead of adding logic to list component |

## Batch Delete

User request: `后台加一个批量删除历史订单的按钮，选中后直接删除。`

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| Physical deletion of orders | BLOCK | Ask whether this is soft delete, archive, cancel, or irreversible delete |
| Permission to delete | BLOCK | Ask which roles can operate and whether tenant isolation applies |
| Audit requirement | BLOCK | Ask whether delete action must be audited |

Do not produce an implementation plan until the human answers.

## Tenant Isolation Bug

User request: `用户切换组织后还能看到旧组织数据，尽快修一下。`

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| Cross-tenant data visibility | BLOCK | Ask for intended isolation rule if not discoverable from code |
| Cache invalidation | AUTO_DEFEND | Inspect frontend store, localStorage, request cache, and query keys |
| Backend query boundary | AUTO_DEFEND | Verify tenant/org condition is enforced server-side |

If code clearly defines the tenant rule, proceed to fix using the existing rule; otherwise block for clarification.

## Membership Benefits

User request: `给会员增加等级权益，高级会员可以多用一些功能，价格后面再说。`

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| Entitlement boundaries | BLOCK | Ask exact feature limits by level |
| Billing or price impact | BLOCK | Ask whether payment, renewal, or entitlement migration is in scope |
| Backward compatibility | CONFIRM_WITH_DEFAULT | Propose no change to existing users until rules are confirmed |

## External Notification

User request: `审批通过后给客户发短信和邮件提醒。`

| Item | Risk Level | Recommended Handling |
| --- | --- | --- |
| SMS/email side effect | BLOCK | Ask trigger timing, recipients, templates, opt-out, and retry policy |
| Third-party provider write | BLOCK | Ask provider and environment boundary |
| Duplicate submission | AUTO_DEFEND | Add idempotency or duplicate-send guard after side-effect rules are approved |
