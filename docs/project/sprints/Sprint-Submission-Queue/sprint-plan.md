# Sprint Plan — Submission Queue

**Sprint**: Sprint-Submission-Queue  
**Goal**: Eliminate concurrent report submission race conditions by implementing a backend-managed FIFO submission queue with session locking, stale session reaping, and a frontend queue-aware send flow with waiting UI.  
**Timeline**: 10 working days (single sprint)  
**Branch**: `feature/submission-queue` from `dev` (DAR_Middleware) | `feature/submission-queue` from `MDAG-331` (main_dar_app)

---

## 1. Sprint goal

- **Business**: Prevent data corruption (duplicate batches, duplicate document numbers, corrupted totals) when multiple farm supervisors submit DAR reports simultaneously.
- **Feature**: Database-backed FIFO queue that serializes the entire send pipeline so only one user's submission runs at a time, with queue position visibility and stale session recovery.
- **Quality**: All changes covered by 40 test cases; UAT scenarios for concurrent submission verified on physical devices.

---

## 2. Scope summary

| Category | Items |
|----------|--------|
| **Epic** | Epic-SQ — Submission Queue |
| **User stories** | US-SQ01–US-SQ07 (enqueue on send, queue position UI, one-at-a-time enforcement, complete/fail lifecycle, stale expiry, idempotent enqueue, frontend timeout) |
| **Technical stories** | TS-SQ01–TS-SQ05 (DB table, queue API endpoints, frontend service layer, review_action wrap, stale reaper) |
| **Out of scope** | Refactoring tblbatchno/tblDARbatch internals; DB row-level locking; multi-server queue; offline queue; priority queuing |

Full scope: [scope.md](scope.md)

---

## 3. Sprint structure — One Sprint (10 days)

| Phase | Days | Activities |
|-------|------|-----------|
| **Phase 1: Foundation & Backend** | Days 1–4 | API contract, DB migration, 4 endpoints, stale reaper, Swagger, backend code review |
| **Phase 2: Frontend** | Days 2–7 | Config endpoints, API service, session service, queue dialog, review_action wrap (overlaps Phase 1 via contract-first) |
| **Phase 3: Integration Testing** | Days 7–8 | E2E testing, concurrent submission tests, bug fixes |
| **Phase 4: UAT & Deployment** | Days 9–10 | UAT execution, deployment plan, sprint review, sign-off |

Full timeline: [sprint-plan-timeline.md](sprint-plan-timeline.md)

---

## 4. Execution alignment

Per [delivery-orchestrator](../../../agents/delivery-orchestrator.md) (DAR context):

- **Frontend**: main_dar_app (Flutter)
- **Backend**: DAR_Middleware (Node/Express, MSSQL, Knex)
- **QA first**: Test strategy and test cases before dev implementation
- **DB migration before code deploy**: Strict deployment order

---

## 5. Deliverables (mandatory)

| # | Artifact | Owner | Status |
|---|----------|--------|--------|
| 1 | [sprint-plan.md](sprint-plan.md) | PM | Done |
| 2 | [sprint-plan-timeline.md](sprint-plan-timeline.md) | PM | Done |
| 3 | [scope.md](scope.md) | PO / Business | Done |
| 4 | [tasks-by-agent.md](tasks-by-agent.md) | Dev squads / SM | Done |
| 5 | [deliverables.md](deliverables.md) | Dev squads | Done |
| 6 | [test-matrix.md](test-matrix.md) | QA Lead | Done |
| 7 | [technical-design.md](../../specs/technical/submission-queue-technical-design.md) | Architect | Done |
| 8 | [uat-checklist.md](uat-checklist.md) | QA Lead + PM | Done (in test-matrix.md §4) |
| 9 | [cicd-impact.md](cicd-impact.md) | DevOps | Done |
| 10 | [deployment-plan.md](deployment-plan.md) | DevOps | Done |
| 11 | [risk-log.md](risk-log.md) | SM / PM | Done |
| 12 | [carryover.md](carryover.md) | SM | Done |

---

## 6. Key references

- [Technical design](../../specs/technical/submission-queue-technical-design.md) — Full architecture, API contract, locking strategy, sequence diagram
- [Backend tasks](backend-tasks.md) — BE-SQ01–BE-SQ10
- [Frontend tasks](frontend-tasks.md) — FE-SQ01–FE-SQ07
- [dar-system-context](../../dar-system-context.md)
- [delivery-orchestrator](../../../agents/delivery-orchestrator.md)
