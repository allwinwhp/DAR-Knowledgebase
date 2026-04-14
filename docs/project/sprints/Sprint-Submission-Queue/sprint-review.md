# Sprint Review — Submission Queue

**Sprint**: Sprint-Submission-Queue
**Date**: 2026-04-14 (started)
**Status**: Implementation in progress

---

## What Was Delivered

### Backend (DAR_Middleware)

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | `tbl_SubmissionQueue` Knex migration | ✅ Implemented |
| 2 | Raw SQL setup script (`07_create_tbl_SubmissionQueue.sql`) | ✅ Implemented |
| 3 | `POST /queue/enqueue` — idempotent enqueue | ✅ Implemented |
| 4 | `POST /queue/check-turn` — atomic activation with UPDLOCK/HOLDLOCK | ✅ Implemented |
| 5 | `POST /queue/complete` — mark session completed | ✅ Implemented |
| 6 | `POST /queue/fail` — mark session failed | ✅ Implemented |
| 7 | `expire-stale.js` — shared reaper utility | ✅ Implemented |
| 8 | Queue router + server.js mount + reaper startup | ✅ Implemented |
| 9 | Swagger JSDoc annotations for all endpoints | ✅ Implemented |
| 10 | Integration tests (`tests/queue/queue.test.js`) | ✅ Implemented |

### Frontend (main_dar_app)

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | Queue endpoint constants in `config.dart` | ✅ Implemented |
| 2 | Queue UI string constants in `constants.dart` | ✅ Implemented |
| 3 | `QueueApiService` — HTTP service layer | ✅ Implemented |
| 4 | `QueueSessionService` — polling orchestrator | ✅ Implemented |
| 5 | `QueueWaitingDialog` — queue position UI | ✅ Implemented |
| 6 | `review_action.dart` — send flow wrapped with queue | ✅ Implemented |

### Database

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | `tbl_SubmissionQueue` table schema | ✅ Designed |
| 2 | `IX_SubmissionQueue_Status_SessionId` index | ✅ Designed |
| 3 | `IX_SubmissionQueue_SupervisorId` index | ✅ Designed |

---

## Demo Notes

- **Single user flow**: Tap Send → enqueue → immediate ACTIVE (no queue wait) → pipeline → complete
- **Multi-user flow**: Tap Send → enqueue → WAITING with position display → poll → turn arrives → pipeline → complete
- **Failure recovery**: App crash → reaper expires stale session → next user proceeds
- **Concurrency guarantee**: SQL UPDLOCK + HOLDLOCK ensures exactly one ACTIVE session at a time

---

## Stakeholder Feedback

*(To be filled during UAT)*

---

## Pending

- [ ] Deploy backend to physical server
- [ ] Run database migration
- [ ] Build and distribute new APK
- [ ] Execute UAT scenarios (uat-checklist.md)
- [ ] Stakeholder sign-off
