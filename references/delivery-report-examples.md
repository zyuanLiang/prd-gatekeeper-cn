# Delivery Report Examples

Use this reference when producing the final delivery report after approved implementation and verification.

## Example: Pending Task Excel Export

### Requirement Normalization

| Item | Result |
| --- | --- |
| Goal | Support Excel export from the pending task list |
| Scope | Export selected rows when checked; otherwise export rows matching current filters |
| Out of Scope | No async export queue, email delivery, or new audit table in MVP |
| Business Rules | Export visible business columns only; exclude selection, operation, and detail columns |
| Confirmed Defaults | Sync export max 10000 rows; over-limit asks user to narrow filters |

### Engineering Change Matrix

| Module | File Path | Core Change | Risk Defense | Status |
| --- | --- | --- | --- | --- |
| Frontend UI | `views/task-list/index.vue` | Add export button and loading state | UI only wires events and state | Done |
| Frontend Logic | `services/taskExport.ts` | Add export parameter and column mapping helper | Business mapping extracted from UI file | Done |
| Backend API | `api/taskExportController.ts` | Add export endpoint reusing list filters | Permission guard matches list query | Done |
| Backend Query | `services/taskQuery.ts` | Reuse existing filter builder with export limit | Force max 10000 rows | Done |

### Verification Results

| Check | Command or Method | Result | Notes |
| --- | --- | --- | --- |
| Type Check | `npm run typecheck` | Passed | No new type errors |
| Tests | `npm test -- task-list` | Passed | List query and export mapping covered |
| Build | `npm run build` | Passed | Production build completed |

### Action Required

| Item | Reason | Owner | Required Before |
| --- | --- | --- | --- |
| Export row limit | 10000 rows is an approved MVP default but may need product tuning | Product/Tech Lead | Release review |
| Audit log | Export may be sensitive in some deployments | Security/Product | Follow-up |

## Example: BLOCK Response

| Risk Level | Blocker | Why It Must Be Human-Decided | Minimum Question | Owner |
| --- | --- | --- | --- | --- |
| BLOCK | Batch physical delete of orders | Irreversible data loss and audit/compliance risk | Should this be soft delete, archive, cancel, or permanent delete? | Product/Tech Lead |
| BLOCK | Customer SMS/email notification | External real-world side effect with duplicate-send and consent risk | Who receives which template, and when should retries stop? | Product/Ops |

Implementation is stopped until these decisions are confirmed.
