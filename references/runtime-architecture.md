# Runtime Architecture

Use this reference when a request asks how the skill should behave as a production-grade decision runtime, or when decision priority, traceability, output contract, or future platform evolution matter.

## Runtime Layers

| Layer | Responsibility | Output |
| --- | --- | --- |
| Input | Accept PRD, Issue, chat request, tool call, agent event, or API-like payload | Raw request |
| Pre-processing | Normalize business language, enrich context from repository facts, validate input, identify idempotency needs | Canonical request |
| Routing | Decide whether `prd-gatekeeper-cn`, Risk Gate examples, or business_flat templates are relevant | Selected guidance |
| Rule Evaluation | Apply `SKILL.md`, `risk-gate-examples.md`, and business_flat rules | Decision candidates |
| Decision Resolver | Apply deterministic priority and choose final gate result | Final decision |
| Safety Enforcement | Enforce `BLOCK`, permission, compliance, audit, and external-side-effect boundaries | Policy-approved decision |
| Output Contract | Render stable decision fields and business_flat tables | Reviewable response |
| Post-processing | Record trace in the response; do not create stores or metrics unless asked | Decision trace |

## Deterministic Priority

| Priority | Rule | Meaning |
| --- | --- | --- |
| 1 | Risk Gate hard override | `BLOCK` and `CONFIRM_WITH_DEFAULT` override model preference and implementation eagerness |
| 2 | business_flat structure | Tables shape context and review output; they do not decide business policy |
| 3 | Skill workflow | Repository inspection, complexity control, verification, and delivery discipline guide execution |
| 4 | LLM fallback | Use only when deterministic rules and examples do not classify the request clearly |

80-95 percent of routine decisions should come from deterministic rules and examples. Use LLM judgment only for ambiguous gaps, and include that uncertainty in `decision_trace`.

## Decision Contract

| Field | Description |
| --- | --- |
| `decision` | `BLOCK`, `CONFIRM_WITH_DEFAULT`, or `AUTO_DEFEND` |
| `risk_level` | `HIGH`, `MEDIUM`, or `LOW` |
| `actions` | Concrete next actions, safeguards, or questions |
| `explanation` | Short reason for the final decision |
| `decision_trace` | Applied rules, references, repository facts, and assumptions |
| `idempotency_key` | Stable key when repeated external effects, exports, retries, or workflow replays matter |

## Design Boundaries

| Boundary | Rule |
| --- | --- |
| No platform runtime by default | Do not add DSL engines, databases, metrics stores, registries, or orchestration code in this MVP |
| No hidden decision mutation | Human overrides to defaults must be reflected in the confirmed defaults and trace |
| No business_flat overreach | Treat business_flat as normalization and presentation, not the source of decisions |
| No LLM override of safety | Model preference cannot downgrade `BLOCK` or skip `CONFIRM_WITH_DEFAULT` |

## Future Phases

| Phase | Candidate Capability |
| --- | --- |
| Phase 1 | Output contract, deterministic priority engine, decision trace |
| Phase 2 | IF/THEN rule DSL, registry versioning |
| Phase 3 | Multi-skill chain orchestration, A/B decision policy, hot-swap rules |
