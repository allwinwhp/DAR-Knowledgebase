# Deliverables — Sprint-Submission-Queue

---

## Backend Deliverables (DAR_Middleware)

| # | Deliverable | Files | Owner |
|---|-------------|-------|-------|
| 1 | `tbl_SubmissionQueue` migration | `db/migrations/20260414_create_tbl_SubmissionQueue.js` | Backend |
| 2 | Raw SQL setup script | `db/sqlsetup/07_create_tbl_SubmissionQueue.sql` | Backend |
| 3 | Expire-stale utility | `queue/expire-stale.js` | Backend |
| 4 | POST `/queue/enqueue` | `queue/enqueue.js` | Backend |
| 5 | POST `/queue/check-turn` | `queue/check-turn.js` | Backend |
| 6 | POST `/queue/complete` | `queue/complete.js` | Backend |
| 7 | POST `/queue/fail` | `queue/fail.js` | Backend |
| 8 | Queue router + server mount | `queue/routes/index.js`, `server.js` | Backend |
| 9 | Swagger documentation | `queue/*.js` (JSDoc), `swagger.js` | Backend |
| 10 | Integration tests | `tests/queue/*.test.js` | Backend |

### New directory structure

```
queue/
├── routes/
│   └── index.js
├── enqueue.js
├── check-turn.js
├── complete.js
├── fail.js
└── expire-stale.js
```

---

## Frontend Deliverables (main_dar_app)

| # | Deliverable | Files | Owner |
|---|-------------|-------|-------|
| 1 | Queue endpoint constants | `lib/core/presentation/theme/config.dart` (modified) | Frontend |
| 2 | Queue UI string constants | `lib/core/presentation/theme/constants.dart` (modified) | Frontend |
| 3 | Queue API service | `lib/app/service/queue/queue_api_service.dart` (new) | Frontend |
| 4 | Queue session orchestrator | `lib/app/service/queue/queue_session_service.dart` (new) | Frontend |
| 5 | Queue waiting dialog | `lib/core/presentation/widgets/common/dialogs/queue_waiting_dialog.dart` (new) | Frontend |
| 6 | Review action integration | `lib/app/screen/review/widgets/action/review_action.dart` (modified) | Frontend |

### New directory structure

```
lib/app/service/queue/
├── queue_api_service.dart
└── queue_session_service.dart

lib/core/presentation/widgets/common/dialogs/
└── queue_waiting_dialog.dart
```

---

## Database Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 1 | `tbl_SubmissionQueue` table | MSSQL table with IDENTITY SessionId, Status state machine, indexes |
| 2 | `IX_SubmissionQueue_Status_SessionId` index | Composite index for queue ordering queries |
| 3 | `IX_SubmissionQueue_SupervisorId` index | Index for idempotent enqueue lookups |

---

## Documentation Deliverables

| # | Deliverable | Location |
|---|-------------|----------|
| 1 | Technical design | `docs/specs/technical/submission-queue-technical-design.md` |
| 2 | Sprint plan | `docs/project/sprints/Sprint-Submission-Queue/sprint-plan.md` |
| 3 | Timeline | `docs/project/sprints/Sprint-Submission-Queue/sprint-plan-timeline.md` |
| 4 | Scope | `docs/project/sprints/Sprint-Submission-Queue/scope.md` |
| 5 | Backend tasks | `docs/project/sprints/Sprint-Submission-Queue/backend-tasks.md` |
| 6 | Frontend tasks | `docs/project/sprints/Sprint-Submission-Queue/frontend-tasks.md` |
| 7 | Test matrix | `docs/project/sprints/Sprint-Submission-Queue/test-matrix.md` |
| 8 | CI/CD impact | `docs/project/sprints/Sprint-Submission-Queue/cicd-impact.md` |
| 9 | Deployment plan | `docs/project/sprints/Sprint-Submission-Queue/deployment-plan.md` |
| 10 | Risk log | `docs/project/sprints/Sprint-Submission-Queue/risk-log.md` |
| 11 | Carryover | `docs/project/sprints/Sprint-Submission-Queue/carryover.md` |
| 12 | Technical sequence diagram (PlantUML) | `docs/project/sprints/Sprint-Submission-Queue/submission-queue-sequence-diagram.puml` |
