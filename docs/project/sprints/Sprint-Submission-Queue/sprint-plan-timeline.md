# Sprint Plan & Timeline — Submission Queue

**Sprint**: Sprint-Submission-Queue
**Attribution**: [PM]
**Date**: 2026-04-14
**Branch**: `feature/submission-queue` from `dev` (DAR_Middleware) | `feature/submission-queue` from `MDAG-331` (main_dar_app).
**Repos**: DAR_Middleware (backend), main_dar_app (frontend)

---

## 1. Sprint Goal

Eliminate concurrent report submission race conditions by implementing a backend-managed submission queue with session locking, stale session reaping, and a frontend queue-aware send flow with waiting UI — ensuring data integrity when multiple supervisors submit simultaneously.

---

## 2. Sprint Structure Recommendation

### Verdict: **One focused sprint (10 working days)** — not two.

**Rationale:**

| Factor | Assessment |
|--------|------------|
| Total tasks | ~12–14 development tasks + QA — within single-sprint capacity |
| Cross-repo dependency | Frontend integration depends on backend endpoints, but service layer stubs can start Day 2 using the API contract |
| Parallelism | Backend and frontend work can overlap ~60% of the sprint via contract-first development |
| Team size | backend-squad, frontend-squad, dev-mid, dev-junior, qa-lead, qa-mid, architect, devops — sufficient bench for parallel tracks |
| Feature isolation | Single coherent feature; splitting across sprints adds inter-sprint integration risk and overhead |

Splitting into two sprints would add unnecessary ceremony (two planning sessions, two retros, one sprint with idle frontend, one with idle backend) for what is a single atomic feature. The dependency between backend and frontend is managed by starting frontend stubs in parallel and deferring integration to Phase 3.

---

## 3. Timeline — Phased Execution

### Phase 1: Foundation & Backend (Days 1–4)

| Day | Activity | Who |
|-----|----------|-----|
| Day 1 | Sprint kickoff; architect finalizes API contract (endpoint signatures, request/response schemas); QA Lead begins test strategy | architect, qa-lead, PM |
| Day 1 | DB migration script: create submission queue table (`tblSubmissionQueue` or equivalent) with locking columns | backend-squad |
| Day 2 | Endpoint 1: Acquire session lock (POST) | backend-squad |
| Day 2 | Endpoint 2: Check queue position / status (GET) | dev-mid |
| Day 3 | Endpoint 3: Release session lock (POST/DELETE) | backend-squad |
| Day 3 | Endpoint 4: Admin/diagnostic — list active sessions (GET) | dev-mid |
| Day 3 | Stale session reaper (background job / interval-based cleanup) | backend-squad |
| Day 4 | Mount all routes in `server.js`; Swagger/OpenAPI documentation | dev-mid |
| Day 4 | Backend code review (dev-senior equivalent review by architect) | architect |

### Phase 2: Frontend (Days 2–7, overlapping)

| Day | Activity | Who |
|-----|----------|-----|
| Day 2 | Create config endpoints file (queue API base URLs, timeouts, retry config) | dev-junior |
| Day 3 | Create API service file (HTTP calls to 4 queue endpoints, using contract from Day 1) | frontend-squad |
| Day 4 | Create session service file (state management: acquire, poll, release, error handling) | frontend-squad |
| Day 5 | Modify `review_action` send flow — integrate queue: acquire lock → submit → release | frontend-squad |
| Day 5–6 | Queue waiting UI dialog (position indicator, estimated wait, cancel option) | dev-junior |
| Day 6–7 | Frontend integration with live backend endpoints (replace stubs with real calls) | frontend-squad |
| Day 7 | Frontend code review | architect |

### Phase 3: Integration Testing (Days 7–8)

| Day | Activity | Who |
|-----|----------|-----|
| Day 7 | QA test cases finalized (test-matrix.md) | qa-lead, qa-mid |
| Day 7–8 | End-to-end integration testing: happy path, concurrent submissions, stale reaping, error recovery | qa-mid |
| Day 8 | Bug fixes from integration testing | backend-squad, frontend-squad |

### Phase 4: UAT & Deployment Prep (Days 9–10)

| Day | Activity | Who |
|-----|----------|-----|
| Day 9 | UAT execution (farm supervisor persona, concurrent submission scenarios) | qa-mid, qa-lead |
| Day 9 | UAT bug fixes (if any) | backend-squad, frontend-squad |
| Day 10 | Deployment plan finalized; risk log updated; Definition of Done review | devops, PM |
| Day 10 | Sprint review; PM sign-off; merge readiness | PM, architect |

---

## 4. Milestones

| Milestone | Target | Gate Criteria | Owner |
|-----------|--------|---------------|-------|
| **M0 — Sprint Kickoff** | Day 1 | API contract signed off; test strategy drafted; sprint backlog confirmed | PM, architect |
| **M1 — Backend Complete** | Day 4 EOD | All 4 endpoints functional; migration applied to dev DB; stale reaper running; Swagger updated; code reviewed | backend-squad, architect |
| **M2 — Frontend Stubs Ready** | Day 4 EOD | Config, API service, session service files created (stubbed against contract) | frontend-squad, dev-junior |
| **M3 — Frontend Integrated** | Day 7 EOD | Send flow modified; queue dialog functional; end-to-end calls against live backend | frontend-squad |
| **M4 — Integration Tested** | Day 8 EOD | All test cases pass; no P1/P2 bugs open; test-matrix.md complete | qa-lead, qa-mid |
| **M5 — UAT Passed** | Day 9 EOD | UAT scenarios pass; stakeholder sign-off on queue behavior | qa-lead, PM |
| **M6 — Sprint Complete** | Day 10 EOD | DoD met; deployment plan ready; sprint review done; branch merge-ready | PM |

---

## 5. Resource Allocation

| Role | Agent | Assigned Tasks | Days Active |
|------|-------|----------------|-------------|
| **backend-squad** | backend-squad | DB migration, endpoints 1/3, stale reaper, integration bug fixes | Days 1–4, 8–9 |
| **dev-mid** | dev-mid | Endpoints 2/4, server.js mount, Swagger docs | Days 2–4 |
| **frontend-squad** | frontend-squad | API service, session service, review_action modification, integration | Days 3–7, 8–9 |
| **dev-junior** | dev-junior | Config endpoints file, queue dialog UI | Days 2–6 |
| **architect** | architect | API contract, code reviews (BE Day 4, FE Day 7), technical decisions | Days 1, 4, 7, 10 |
| **qa-lead** | qa-lead | Test strategy (Day 1), test-matrix.md, UAT checklist, QA sign-off | Days 1, 7, 9–10 |
| **qa-mid** | qa-mid | Test case authoring, integration testing execution, UAT execution | Days 5–9 |
| **devops** | devops | Deployment plan, CI/CD impact assessment, environment prep | Days 8–10 |
| **PM** | PM | Kickoff, milestone tracking, risk management, DoD review, sprint sign-off | Days 1, 4, 8, 10 |

### Utilization Notes

- **backend-squad** is front-loaded (Days 1–4) then available for bug fixes. If backend finishes early, can assist frontend-squad with integration.
- **dev-junior** has a lighter load — can assist qa-mid with test data preparation on Days 7–8 if ahead of schedule.
- **frontend-squad** ramps up as backend delivers endpoints. Contract-first approach means no idle waiting.

---

## 6. Definition of Done

### Code
- [ ] All 4 queue API endpoints implemented, tested, and documented (Swagger)
- [ ] DB migration script committed and verified against dev MSSQL instance
- [ ] Stale session reaper runs on configurable interval; cleans sessions older than threshold
- [ ] `server.js` mounts queue routes at agreed path prefix
- [ ] Frontend config, API service, and session service files created and integrated
- [ ] `review_action` send flow acquires lock before submit and releases after
- [ ] Queue waiting dialog shows position, estimated wait, and allows cancel
- [ ] Code reviewed by architect (backend Day 4, frontend Day 7)

### Quality
- [ ] Test strategy and test-matrix.md delivered by qa-lead
- [ ] All test cases in test-matrix pass (integration)
- [ ] Concurrent submission scenarios tested (minimum 3 simultaneous users)
- [ ] Stale session reaping verified (session abandoned → reaped within configured interval)
- [ ] Error recovery tested (network drop during queue wait, server restart)
- [ ] No open P1 or P2 bugs

### Delivery
- [ ] UAT passed with farm supervisor persona scenarios
- [ ] Deployment plan documented (devops)
- [ ] Risk log updated
- [ ] Sprint branch merge-ready (reviewed, no conflicts with `MDAG-339`)
- [ ] PM sign-off on sprint review

---

## 7. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|------------|--------|------------|
| R1 | **Backend delays push frontend integration past Day 6** | Medium | High | Contract-first development: frontend builds against agreed API contract from Day 1. Backend targets M1 by Day 4; Day 5 is buffer before frontend needs live endpoints on Day 6. |
| R2 | **MSSQL locking behavior differs from expectations under concurrency** | Medium | High | Architect validates locking strategy (row-level vs table-level) on Day 1 during contract definition. Load test with 5+ concurrent writers during integration phase. |
| R3 | **Stale reaper interval tuning** — too aggressive = premature session drops; too conservative = queue stalls | Low | Medium | Default to conservative interval (e.g., 5 min); make configurable via environment variable. Test both extremes during QA. |
| R4 | **Flutter queue dialog UX friction** — supervisors unfamiliar with "waiting" concept in the field | Low | Medium | Keep dialog simple (spinner + position + cancel). dev-junior prototypes early (Day 5); qa-mid tests with realistic field scenario on Day 9 UAT. |
| R5 | **Scope creep — additional queue features requested mid-sprint** (priority queuing, admin dashboard, etc.) | Medium | Medium | Sprint scope locked at kickoff (M0). Any additions go to carryover.md for next sprint. PM enforces. |
| R6 | **Branch conflicts with ongoing `MDAG-339` work** | Low | Low | Sprint branch created from latest `MDAG-339` at kickoff. Daily rebase or merge from `MDAG-339` if other work lands. |
| R7 | **Dev environment MSSQL availability** — shared DB may have contention during testing | Low | Medium | devops confirms dedicated dev DB instance or isolated schema for queue table testing by Day 1. |

### Dependency Map

```
Migration (BE) ──┐
                  ├──► Endpoints 1-4 (BE) ──► server.js mount ──► Backend Complete (M1)
Stale Reaper (BE)─┘                                                       │
                                                                          ▼
API Contract (Day 1) ──► FE Stubs (Days 2-4) ──► FE Integration (Days 5-7) ──► M3
                                                                                 │
                                                                                 ▼
                                                          Integration Testing (Days 7-8) ──► UAT (Day 9) ──► Done (Day 10)
```

**Critical path**: API contract → Backend endpoints → Frontend integration → Integration testing → UAT. Any slip on the backend critical path compresses testing. The 1-day buffer between M1 (Day 4) and frontend integration start (Day 5–6) is the primary schedule protection.

---

## 8. Sprint Artifacts (Mandatory)

Per sprint conventions from Sprint-Jay-Feedback:

| # | Artifact | Owner | Due |
|---|----------|-------|-----|
| 1 | `sprint-plan-timeline.md` (this document) | PM | Day 1 |
| 2 | `scope.md` | PO / Business | Day 1 |
| 3 | `tasks-by-agent.md` | Dev squads / SM | Day 1 |
| 4 | `test-matrix.md` | QA Lead | Day 7 |
| 5 | `code-deep-dive.md` | Architect | Day 1 |
| 6 | `uat-checklist.md` | QA Lead + PM | Day 8 |
| 7 | `deployment-plan.md` | DevOps | Day 8 |
| 8 | `risk-log.md` | SM / PM | Day 1 (updated Day 10) |
| 9 | `carryover.md` | SM | Day 10 |
| 10 | `execution-summary.md` | PM | Day 10 |

---

*[PM] — Sprint-Submission-Queue plan approved for kickoff. Single 10-day sprint with phased parallel execution. Next action: architect to finalize API contract and QA Lead to begin test strategy.*
