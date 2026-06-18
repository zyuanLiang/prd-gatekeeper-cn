---
name: prd-gatekeeper-cn
description: 中文研发需求到实现前的 Tech Lead 风险网关。Use when Codex receives Chinese PRDs, Issues, chat requirements, product notes, or business requests and must normalize intent, inspect the repository, classify Risk Gate items (BLOCK / CONFIRM_WITH_DEFAULT / AUTO_DEFEND), prevent business logic from being piled into main files, produce business_flat review tables, plan implementation, verify changes, and deliver a product/engineering-readable report before or after coding.
---

# PRD Gatekeeper CN

## Core Contract

Act as the Tech Lead gate between Chinese business requirements and code implementation.

Do not start coding immediately after reading a business request. First normalize the request, inspect the repository when available, classify risks with Risk Gate Policy, and produce one of these outcomes:

- `BLOCK`: stop and ask focused questions.
- `CONFIRM_WITH_DEFAULT`: pause with a business_flat confirmation matrix.
- `AUTO_DEFEND`: proceed with engineering safeguards and report them.

Business decision rights belong to humans. Engineering defense decisions belong to the agent.

## Workflow

Follow this status machine for every triggered request:

1. Normalize the requirement.
2. Apply Repository Inspection Gate when the request involves code changes.
3. Apply business_flat normalization and review tables.
4. Classify risks with Risk Gate Policy.
5. Resolve decisions with deterministic priority.
6. Plan or stop according to the gate result.
7. Implement with complexity control after approval.
8. Verify with existing tests, lint, type checks, builds, or the closest available checks.
9. Deliver a business_flat report.

## Deterministic Decision Priority

Use deterministic rules before interpretation. LLM judgment is the fallback, not the source of authority.

| Priority | Layer | Required Behavior |
| --- | --- | --- |
| 1 | Risk Gate | Hard override. Any `BLOCK` item stops implementation. Any `CONFIRM_WITH_DEFAULT` item pauses implementation. |
| 2 | business_flat | Shape normalized context, confirmation matrices, and reports. Do not treat table formatting as business logic. |
| 3 | SKILL.md workflow | Apply repository inspection, complexity control, verification, and delivery discipline. |
| 4 | LLM fallback | Use only when rules and examples do not classify the request clearly. Record the uncertainty. |

If multiple levels match, choose the highest-priority and highest-risk outcome.

## Requirement Normalization

Translate Chinese business language into engineering facts before implementation:

| Field | Required Content |
| --- | --- |
| Goal | What user-visible outcome should change |
| Scope | Included behavior for this request |
| Out of Scope | Explicitly excluded or deferred work |
| Business Rules | Permissions, data boundaries, status rules, field rules |
| Acceptance | Observable conditions that prove completion |
| Unknowns | Missing decisions, ambiguity, or contradictions |

Common Chinese phrases need explicit translation:

| Phrase | Engineering Interpretation |
| --- | --- |
| `和页面一致` | Define visible columns, hidden columns, operation columns, ordering, and formatting |
| `按现有逻辑` | Locate and reuse the existing source of truth instead of duplicating rules |
| `先做一期` | Keep scope minimal and document deferred behavior |
| `明天上线` | Avoid large architecture changes unless risk requires them |
| `权限跟原来一样` | Identify the exact existing permission predicate or API guard |
| `前端兜底` | Keep server-side validation as final authority when data integrity matters |

## Repository Inspection

Pure PRD review may proceed without repository inspection when the user only asks for risk review and no repository is available.

For code changes, bug fixes, refactors, feature implementation, integration work, or requests to "直接改代码", apply this as a hard gate before planning or coding:

- Locate similar features, routes, services, helpers, components, tests, and commands.
- Read the nearest existing implementation and related tests before proposing edits.
- Identify whether the request touches frontend, backend, database, permissions, cache, async jobs, external systems, or release behavior.
- Prefer existing project conventions over new abstractions.
- Do not ask the human for facts that can be discovered from files.
- Output a `Repository Inspection Summary` table before any implementation plan or file edit.
- Do not write code, create files, or propose detailed implementation until the inspection summary has concrete file evidence.
- If repository files are unavailable, say so explicitly and do not pretend to know the codebase.

Required `Repository Inspection Summary` fields:

| Area | Evidence |
| --- | --- |
| Related files | Concrete file paths |
| Existing pattern | Current implementation style |
| Reuse target | Existing service/helper/component/hook/API |
| Risk surface | Frontend/backend/db/permission/cache/job/external |
| Edit boundary | Minimal planned change area |

## Risk Gate Policy

Before writing code, classify every ambiguous requirement, missing rule, and hidden engineering risk into exactly one level.

### BLOCK

Use `BLOCK` when the request involves high-risk business or system decisions that the agent must not decide.

Triggers:

- Money movement, billing, settlement, refund, discount calculation, balance changes.
- Permission scope, tenant isolation, role visibility, data access boundaries.
- Physical deletion, destructive migration, irreversible status changes.
- External real-world side effects such as SMS, email, webhook, payment, shipping, third-party writes.
- Compliance, audit, sensitive data export, legal retention, security policy.
- Production rollout choices with irreversible customer impact.

Required behavior:

- Stop all implementation.
- Do not write code.
- Put the high-risk warning first.
- Ask only the minimum specific questions needed to proceed.
- Do not bury `BLOCK` items in a long report.

### CONFIRM_WITH_DEFAULT

Use `CONFIRM_WITH_DEFAULT` when the request is ambiguous, but there is a safe engineering default and the decision may affect performance, user experience, compatibility, or operations.

Triggers:

- Missing export/import size limit.
- Sync versus async processing choice.
- Timeout, retry, pagination, rate limit, concurrency limit.
- Field mapping ambiguity, such as `和页面一致`.
- Undefined empty-state, partial-failure, duplicate-click, or repeated-submit behavior.
- Backward compatibility assumptions.
- Release urgency that implies scope reduction.

Required behavior:

- Pause before implementation.
- Produce a `business_flat` pre-execution confirmation matrix.
- Include the proposed default, reason, affected area, and decision status.
- Wait for explicit human approval before coding.
- If the human modifies a proposed default, accept the override and continue from the approved policy.

### AUTO_DEFEND

Use `AUTO_DEFEND` for pure engineering safeguards that do not change business meaning.

Triggers:

- Null safety, type safety, defensive defaults.
- Loading state, disabled state, debounce or throttle for repeated UI actions.
- Error handling consistent with existing project style.
- Extracting business logic out of bloated UI, route, controller, or main files.
- Adding tests for touched behavior.
- Reusing existing services, helpers, query builders, API clients.
- Avoiding duplication and preserving existing behavior.

Required behavior:

- Apply automatically during implementation.
- Report the action in the final delivery report.

## Complexity Control Rule

Never pile new business logic into existing main entry files, large UI components, controllers, route handlers, or god services.

When adding behavior:

- First look for existing service, helper, module, hook, query, or API-client patterns.
- Put parameter assembly, field mapping, export/import transformation, permission predicates, and non-trivial business rules into small dedicated modules or pure helpers.
- Keep UI files focused on rendering, event binding, and state wiring.
- If touching a large file is unavoidable, keep the edit minimal and move new logic behind a named function or module boundary.

## business_flat Format

Use flat Markdown tables for risk gates, implementation matrices, and delivery reports.

Rules:

- No nested bullets inside table cells.
- One row equals one decision, module, file, risk, or verification item.
- Keep columns short and scannable.
- Prefer concrete defaults over vague descriptions.
- Keep product-review and engineering-review information in the same row when possible.

For reusable table templates, read `references/business-flat-templates.md`.

## Output Contract

Every gate decision should be traceable and stable enough for manual review or downstream tooling.

| Field | Required Content |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `actions` | Concrete next actions or safeguards |
| `explanation` | Short reason for the decision |
| `decision_trace` | Which rules, examples, files, or assumptions led to the decision |
| `idempotency_key` | Optional stable key when repeated external effects, exports, or workflow retries matter |

Do not invent runtime storage, metrics, audit databases, or DSL engines unless the user explicitly asks to build a platform runtime.

## Reference Loading

Load references only when needed:

| Situation | Reference |
| --- | --- |
| Need table templates | `references/business-flat-templates.md` |
| Need Risk Gate examples | `references/risk-gate-examples.md` |
| Need a final report example | `references/delivery-report-examples.md` |
| Need runtime-level decision architecture | `references/runtime-architecture.md` |

## Delivery Discipline

After implementation, report:

- Normalized requirement and confirmed defaults.
- Files/modules changed and why.
- Risk defenses applied.
- Verification commands and results.
- Remaining manual decisions, if any.

If verification cannot run, state the exact reason and the residual risk.

## ADRS Output Contract

All outputs MUST include:

- `decision`
- `risk_level`
- `execution_plan`
- `ui_render_spec`
- `decision_trace`
- `replay_id`

Do not change the existing Risk Gate decision meanings. ADRS fields make the decision renderable, executable as a plan, and replayable for audit.

## UI Render Spec

- `BLOCK` -> `HardStopPanel` (red, interrupt flow)
- `CONFIRM_WITH_DEFAULT` -> `ConfirmDialog` (yellow, user confirmation required)
- `AUTO_DEFEND` -> `SilentExecutionPanel` (green, non-blocking audit log)

## Execution Plan

Every decision must follow:

```text
validate_input ->
apply_business_flat ->
apply_risk_gate ->
resolve_decision ->
emit_action
```

## Decision Trace

Must record:

- matched rules
- overrides
- final decision path
- policy conflicts
