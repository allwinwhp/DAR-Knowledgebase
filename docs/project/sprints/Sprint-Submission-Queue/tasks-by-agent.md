# Tasks by Agent — Sprint-Submission-Queue

---

## Backend Squad

| ID | Task | Effort | Days |
|----|------|--------|------|
| BE-SQ01 | Knex migration — `tbl_SubmissionQueue` | S | Day 1 |
| BE-SQ02 | Raw SQL setup script (`07_create_tbl_SubmissionQueue.sql`) | S | Day 1 |
| BE-SQ03 | Expire-stale shared function (`queue/expire-stale.js`) | S | Day 2 |
| BE-SQ04 | Enqueue endpoint (`queue/enqueue.js`) | M | Day 2 |
| BE-SQ05 | Check-turn endpoint with UPDLOCK/HOLDLOCK locking (`queue/check-turn.js`) | L | Day 3 |
| BE-SQ06 | Complete endpoint (`queue/complete.js`) | S | Day 2–3 |
| BE-SQ07 | Fail endpoint (`queue/fail.js`) | S | Day 2–3 |
| BE-SQ08 | Queue router & mount in `server.js` | S | Day 4 |
| BE-SQ09 | Swagger JSDoc annotations | M | Day 4 |
| BE-SQ10 | Integration tests | L | Day 4–5 |

Full detail: [backend-tasks.md](backend-tasks.md)

---

## Frontend Squad

| ID | Task | Effort | Days |
|----|------|--------|------|
| FE-SQ01 | Add queue endpoint constants to `Config` | S | Day 2 |
| FE-SQ02 | Add queue UI string constants | S | Day 2 |
| FE-SQ03 | Create `QueueApiService` | M | Day 3 |
| FE-SQ04 | Create `QueueSessionService` (polling orchestrator) | M | Day 4 |
| FE-SQ05 | Create queue waiting dialog widget | M | Day 5 |
| FE-SQ06 | Modify `review_action.dart` — wrap send flow with queue | L | Day 5–6 |
| FE-SQ07 | Manual QA — end-to-end queue flow | M | Day 7 |

Full detail: [frontend-tasks.md](frontend-tasks.md)

---

## Architect

| Task | Days |
|------|------|
| Finalize API contract (endpoint signatures, request/response schemas) | Day 1 |
| Backend code review (post BE-SQ05) | Day 4 |
| Frontend code review (post FE-SQ06) | Day 7 |
| Technical design validation | Day 1, ongoing |

---

## QA Lead

| Task | Days |
|------|------|
| Test strategy and test-matrix.md | Day 1–2 |
| UAT checklist (in test-matrix §4) | Day 7 |
| Integration test oversight | Day 7–8 |
| UAT execution and sign-off | Day 9 |

---

## Dev-Mid

| Task | Days |
|------|------|
| Assist with BE-SQ06, BE-SQ07 (simple endpoints) | Day 2–3 |
| BE-SQ08 (router mount + swagger config) | Day 4 |

---

## Dev-Junior

| Task | Days |
|------|------|
| FE-SQ01, FE-SQ02 (config + constants) | Day 2 |
| FE-SQ05 (queue waiting dialog) | Day 5 |
| Assist qa-mid with test data preparation | Day 7–8 |

---

## QA-Mid

| Task | Days |
|------|------|
| Test case authoring (from test-matrix) | Day 5–6 |
| Integration testing execution | Day 7–8 |
| UAT execution | Day 9 |

---

## DevOps

| Task | Days |
|------|------|
| CI/CD impact assessment | Day 8 |
| Deployment plan finalization | Day 8–9 |
| Environment prep (server env vars, PM2 config) | Day 9 |

---

## PM

| Task | Days |
|------|------|
| Sprint kickoff | Day 1 |
| Milestone tracking (M0–M6) | Ongoing |
| DoD review and sprint sign-off | Day 10 |

---

## SM (Development Scrum Master)

| Task | Days |
|------|------|
| Sprint backlog validation | Day 1 |
| Daily standups / blocker resolution | Ongoing |
| DoD validation and carryover | Day 10 |

---

## Execution Order

```
Day 1:  Kickoff → API contract (Architect) → DB migration (Backend) → Test strategy (QA)
Day 2:  Endpoints (Backend: enqueue, complete, fail) → Config (Frontend: Junior)
Day 3:  Check-turn endpoint (Backend) → QueueApiService (Frontend)
Day 4:  Router mount + Swagger (Backend) → QueueSessionService (Frontend) → Backend review
Day 5:  Backend integration tests → Queue dialog (Frontend) → review_action wrap begins
Day 6:  Frontend integration with live backend
Day 7:  Frontend review → Integration testing → Test cases finalized
Day 8:  Bug fixes → Deployment plan → CI/CD impact
Day 9:  UAT execution → Bug fixes
Day 10: Sprint review → DoD → Sign-off
```
