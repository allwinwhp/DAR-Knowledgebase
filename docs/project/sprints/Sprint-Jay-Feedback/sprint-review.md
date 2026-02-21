# Sprint-Jay-Feedback — Sprint Review

**Owner**: SM + PM  
**Purpose**: What was delivered; demo notes; stakeholder feedback.

---

## What was delivered

| Deliverable | Status | Notes |
|-------------|--------|--------|
| Sprint Package artifacts | Done | risk-log, carryover, cicd-impact, deployment-plan, uat-checklist, test-matrix (incl. Test Guide). |
| BE-1 (US-J1) | Done | Incentive API: Doctype NOT IN ('03','01'); optional icode. Activity already Active=1. |
| Sucker Pruning incentive (US-J4 partial) | Done | Frontend: getIncentivesByIcode('RP13'); form uses RP13 only. |
| TS-SERVER | Done | Server dropdown on login; Config.baseURL; persistence via SharedPreferences. |
| execution-summary.md | Done | Scope, deviations, decisions, risks, test coverage. |
| Remaining BE/FE tasks | Pending | Header fixes (Survey, Gouging, ERAD), Cycle/Sqno, Chemical Mixing, Gen Services Group, Popcount HAS, and other form changes per tasks-by-agent. |

---

## Demo notes

- **Server selection**: Open app → Login screen shows Server dropdown (UAT, Local emulator, localhost). Select server → login and sync/report use that base URL; selection persisted for next launch.
- **Incentive filter**: Sync incentives (no opcode): response excludes Doctype 03/01. Sucker Pruning form shows only RP13 after sync.
- **Backend**: GET /api/incentive returns filtered list; GET /api/activity unchanged (already Active=1).

---

## Stakeholder feedback

*(To be filled after UAT / review.)*

---

## Done vs not done

- **Done**: US-J1 (filters), TS-SERVER (server selection), US-J4 (Sucker Pruning incentive RP13 only), full Sprint Package and Test Guide.
- **Not done**: US-J2–J3, J4 (Cycle/Sqno + backend), J5–J12 backend and frontend work; full TDD with automated unit tests.
