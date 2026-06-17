# business_flat Templates

Use these templates for pre-execution confirmation, implementation review, verification, and delivery reports. Keep every table flat: one row equals one decision, module, file, risk, or check.

## Rules

| Rule | Required Behavior |
| --- | --- |
| No nested cells | Do not put bullet lists, numbered lists, or multi-paragraph notes inside table cells |
| Concrete defaults | Prefer `sync export max 10000 rows` over `add reasonable limit` |
| Short columns | Use short labels such as `Gap`, `Default`, `Reason`, `Area`, `Decision` |
| One row one fact | Split unrelated decisions into separate rows |
| Pause on confirm | Stop after a `CONFIRM_WITH_DEFAULT` matrix until the human approves |

## Pre-Execution Confirmation

Use this when any item is classified as `CONFIRM_WITH_DEFAULT`.

| Risk Level | Gap | Proposed Default | Reason | Affected Area | Decision |
| --- | --- | --- | --- | --- | --- |
| CONFIRM_WITH_DEFAULT | Missing rule | Safe default action | Why this is safe | Module/API/UX | Needs approval |
| AUTO_DEFEND | Engineering risk | Automatic safeguard | Why this does not change business meaning | File/module | Auto apply |

Stop after this table and ask for approval before implementation.

## BLOCK Warning

Use this when any item is classified as `BLOCK`.

| Risk Level | Blocker | Why It Must Be Human-Decided | Minimum Question | Owner |
| --- | --- | --- | --- | --- |
| BLOCK | High-risk decision | Business/security/compliance impact | Specific question needed to continue | Product/Tech Lead/Security |

Do not include an implementation plan in the same response.

## Requirement Normalization

| Item | Result |
| --- | --- |
| Goal | Normalized goal |
| Scope | Included behavior |
| Out of Scope | Deferred or excluded behavior |
| Business Rules | Confirmed rules |
| Confirmed Defaults | Approved defaults |
| Unknowns | Remaining gaps |

## Engineering Change Matrix

| Module | File Path | Core Change | Risk Defense | Status |
| --- | --- | --- | --- | --- |
| Module name | `path/to/file` | Concrete edit | Boundary or safeguard | Planned/Done |

## Verification Results

| Check | Command or Method | Result | Notes |
| --- | --- | --- | --- |
| Type Check | `command` | Passed/Failed/Not available | Short note |
| Tests | `command` | Passed/Failed/Not available | Short note |
| Build | `command` | Passed/Failed/Not available | Short note |

## Action Required

| Item | Reason | Owner | Required Before |
| --- | --- | --- | --- |
| Manual decision | Why it is still needed | Product/Tech Lead/QA | Coding/Release/Follow-up |
