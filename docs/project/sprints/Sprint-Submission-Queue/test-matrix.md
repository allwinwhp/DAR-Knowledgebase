# Submission Queue — Test Matrix

**Feature**: Submission Queue (concurrent report submission control)
**Repos**: DAR_Middleware (backend), main_dar_app (frontend) — Branch `MDAG-339`
**Author**: QA Lead
**Date**: 2026-04-14
**Version**: 1.0

---

## 1. Test Strategy

### 1.1 Objective

Verify that the Submission Queue prevents race conditions when multiple supervisors submit DAR reports concurrently. The queue must enforce strict FIFO ordering, guarantee that only one submission pipeline is ACTIVE at a time, handle stale/crashed sessions, and surface queue position accurately to the user.

### 1.2 Test Levels

| Level | Scope | Tooling | Owner |
|-------|-------|---------|-------|
| **Unit** | Individual functions: enqueue logic, state transitions, reaper interval, idempotency guard, position calculation | Jest (backend), Flutter test (frontend widgets/providers) | Dev (Mid/Junior) |
| **Integration** | API contract: endpoint request/response, SQL locking behavior, state machine transitions across calls, concurrent access | Supertest + MSSQL test DB (backend); Flutter integration_test with mock/real API | QA (Mid) + Backend |
| **E2E** | Full user flow: Send button → enqueue → poll → pipeline → complete/fail; multi-user FIFO; stale reaper recovery | Manual + scripted (concurrent curl/k6 for load; two physical devices or emulators for multi-user) | QA (Senior/Lead) |
| **UAT** | Stakeholder validation: supervisor experience during queue wait, error messaging, resume after background | Manual on target device (field tablet/phone) against UAT server | BA / PO / PM |

### 1.3 Risk-Based Prioritization

| Risk | Impact | Priority | Mitigation |
|------|--------|----------|------------|
| Two submissions run simultaneously (race condition) | Data corruption — duplicate/partial DAR records | **Critical** | SQL UPDLOCK/HOLDLOCK locking tests; concurrent integration tests |
| Stale ACTIVE blocks queue indefinitely | All supervisors blocked from submitting | **Critical** | Reaper unit + integration tests; timeout E2E |
| Double-tap creates duplicate queue entries | Confusing UX; potential data issues | **High** | Idempotency backend test; debounce frontend test |
| Poll timeout with no feedback | Supervisor thinks app is frozen | **High** | Frontend timeout test; error dialog UAT |
| App backgrounded loses poll | Submission never completes | **Medium** | Resume-on-foreground test |

### 1.4 Environment Strategy

| Environment | Purpose | Data |
|-------------|---------|------|
| **Local (dev)** | Unit + integration tests; SQL locking verification | Seeded `tbl_SubmissionQueue` rows |
| **MDAG-UAT (Cavite/Davao)** | E2E and UAT; multi-device concurrency | Shared UAT DB; test supervisor accounts |
| **MDAG-PROD** | Post-release smoke only | Production data; read-only verification |

### 1.5 Entry / Exit Criteria

| Gate | Criteria |
|------|----------|
| **Entry** | `tbl_SubmissionQueue` migration applied; all four endpoints deployed; Flutter queue UI merged to MDAG-339 |
| **Exit — Dev** | All unit tests pass; integration tests pass on local DB |
| **Exit — QA** | All Critical/High test cases pass; no open P1 defects |
| **Exit — UAT** | Stakeholder sign-off per §5; all UAT scenarios executed |
| **Exit — Release** | QA Lead + PM + Dev-Senior agree per release-playbook |

---

## 2. Test Cases

### 2.1 Backend — API & Database

| TC-ID | Category | Scenario | Preconditions | Steps | Expected Result | Priority |
|-------|----------|----------|---------------|-------|-----------------|----------|
| TC-SQ001 | Enqueue | Enqueue with no existing queue → immediate ACTIVE | `tbl_SubmissionQueue` has no WAITING or ACTIVE rows for any supervisor | POST `/queue/enqueue` with `{ supervisorId, reportType }` | 201; response contains `sessionId`, `status: "ACTIVE"`, `position: 0`; row inserted with state ACTIVE and `created_at` timestamp | Critical |
| TC-SQ002 | Enqueue | Enqueue when another session is ACTIVE | Supervisor A has an ACTIVE session in `tbl_SubmissionQueue` | POST `/queue/enqueue` with Supervisor B's `supervisorId` | 201; response contains `sessionId`, `status: "WAITING"`, `position: 1`; row inserted with state WAITING | Critical |
| TC-SQ003 | Enqueue | Double enqueue same supervisorId returns existing session (idempotent) | Supervisor A already has a WAITING or ACTIVE session | POST `/queue/enqueue` with same `supervisorId` | 200; returns the existing `sessionId` and current status; no new row created | High |
| TC-SQ004 | Check-turn | Check-turn returns `isMyTurn: true` when first WAITING and no ACTIVE exists | One WAITING row for Supervisor A; no ACTIVE rows (previous completed/expired) | POST `/queue/check-turn` with `{ sessionId }` | 200; `isMyTurn: true`, `position: 0`; row state transitions to ACTIVE | Critical |
| TC-SQ005 | Check-turn | Check-turn returns `isMyTurn: false` with correct position | Supervisor A is ACTIVE; Supervisor B is WAITING (pos 1); Supervisor C is WAITING (pos 2) | POST `/queue/check-turn` with Supervisor C's `sessionId` | 200; `isMyTurn: false`, `position: 2` | Critical |
| TC-SQ006 | Check-turn | Check-turn with position update after preceding completes | A is ACTIVE, B is WAITING (pos 1), C is WAITING (pos 2); A completes | POST `/queue/check-turn` with Supervisor C's `sessionId` after A completes | 200; `position: 1` (moved up); `isMyTurn: false` (B is now eligible before C) | High |
| TC-SQ007 | Complete | Complete marks session COMPLETED; next WAITING becomes eligible | Supervisor A is ACTIVE; Supervisor B is WAITING | POST `/queue/complete` with A's `sessionId` | 200; A's row → COMPLETED with `completed_at` timestamp; next `check-turn` for B returns `isMyTurn: true` | Critical |
| TC-SQ008 | Fail | Fail marks session FAILED; next WAITING becomes eligible | Supervisor A is ACTIVE; Supervisor B is WAITING | POST `/queue/fail` with A's `sessionId` and `{ reason }` | 200; A's row → FAILED with `failed_at` and `reason`; next `check-turn` for B returns `isMyTurn: true` | Critical |
| TC-SQ009 | Reaper | Stale session reaper marks old ACTIVE as EXPIRED | Supervisor A is ACTIVE with `created_at` > 5 minutes ago; Supervisor B is WAITING | Wait for reaper interval (or trigger manually) | A's row → EXPIRED; next `check-turn` for B returns `isMyTurn: true` | Critical |
| TC-SQ010 | Validation | Invalid sessionId returns 404 | No row exists for the given `sessionId` | POST `/queue/check-turn` with non-existent `sessionId` | 404; `{ error: "Session not found" }` | High |
| TC-SQ011 | Validation | Invalid sessionId on complete | No row exists for the given `sessionId` | POST `/queue/complete` with non-existent `sessionId` | 404; `{ error: "Session not found" }` | High |
| TC-SQ012 | Validation | Invalid sessionId on fail | No row exists for the given `sessionId` | POST `/queue/fail` with non-existent `sessionId` | 404; `{ error: "Session not found" }` | High |
| TC-SQ013 | Concurrency | Concurrent check-turn calls — only one becomes ACTIVE (locking) | Two WAITING sessions (B, C); no ACTIVE; both poll simultaneously | Two parallel POST `/queue/check-turn` requests (B and C) at the same instant | Exactly one returns `isMyTurn: true` (ACTIVE); the other returns `isMyTurn: false`; DB has exactly one ACTIVE row | Critical |
| TC-SQ014 | Concurrency | Concurrent enqueue from different supervisors | No existing queue | Two parallel POST `/queue/enqueue` from Supervisors A and B | Exactly one gets ACTIVE, the other gets WAITING; no deadlock; both receive valid `sessionId` | Critical |
| TC-SQ015 | State integrity | Complete on already-COMPLETED session | Session already in COMPLETED state | POST `/queue/complete` with already-completed `sessionId` | 409 Conflict or idempotent 200; state remains COMPLETED; no side effects | Medium |
| TC-SQ016 | State integrity | Check-turn on COMPLETED/FAILED/EXPIRED session | Session in terminal state | POST `/queue/check-turn` with completed `sessionId` | 400 or 410; clear error indicating session is no longer active | Medium |
| TC-SQ017 | Reaper | Reaper does not touch WAITING sessions | Multiple WAITING sessions older than 5 minutes; no ACTIVE | Reaper runs | WAITING sessions remain WAITING; only ACTIVE sessions older than threshold are expired | High |
| TC-SQ018 | Edge | Enqueue with missing/invalid supervisorId | No `supervisorId` in request body | POST `/queue/enqueue` with `{}` | 400; validation error | Medium |
| TC-SQ019 | Edge | Queue processes correctly after reaper clears stale ACTIVE | ACTIVE expired by reaper; 3 WAITING sessions remain | Reaper fires; next `check-turn` calls arrive | First WAITING (by FIFO/created_at) becomes ACTIVE; others retain correct positions | High |
| TC-SQ020 | Performance | 10 concurrent enqueue requests | Empty queue | 10 simultaneous POST `/queue/enqueue` from 10 different supervisors | Exactly 1 ACTIVE, 9 WAITING; positions 0–9 assigned correctly; no deadlocks; response time < 2s each | High |

### 2.2 Frontend — Flutter (main_dar_app)

| TC-ID | Category | Scenario | Preconditions | Steps | Expected Result | Priority |
|-------|----------|----------|---------------|-------|-----------------|----------|
| TC-SQ101 | Send flow | Send button triggers enqueue before pipeline | Report completed locally; ready to send | Tap "Send to Server" | App calls POST `/queue/enqueue` before starting batch → hdr → dtls → accomp → materials pipeline; queue waiting dialog appears if not immediately ACTIVE | Critical |
| TC-SQ102 | Queue UI | Queue waiting dialog shows position and updates | Enqueue returned WAITING with position > 0 | Observe queue dialog | Dialog shows "You are #N in queue" (or equivalent); position updates every 3 seconds via poll | Critical |
| TC-SQ103 | Queue UI | When `isMyTurn: true`, pipeline starts normally | Poll returns `isMyTurn: true` | Observe after poll | Queue dialog dismisses; existing DAR submission pipeline (batch → hdr → dtls → accomp → materials → totals) executes normally | Critical |
| TC-SQ104 | Pipeline | Pipeline success calls complete | Full pipeline executes without error | Observe after last pipeline step succeeds | App calls POST `/queue/complete` with `sessionId`; success feedback shown to user; report marked as sent | Critical |
| TC-SQ105 | Pipeline | Pipeline failure calls fail | Pipeline encounters an error (e.g., hdr submission fails) | Observe after pipeline error | App calls POST `/queue/fail` with `sessionId` and error reason; error dialog shown; report remains unsent for retry | Critical |
| TC-SQ106 | Timeout | Poll timeout shows error message | Enqueue returned WAITING; poll runs for > configured timeout (e.g., 5 minutes) without becoming ACTIVE | Wait beyond timeout threshold | App stops polling; shows timeout error dialog ("Queue timed out — please try again"); no queue/complete or queue/fail called; user can retry | High |
| TC-SQ107 | Lifecycle | App backgrounded during poll resumes correctly | App is polling (WAITING state); user switches to another app | Return to DAR app | Polling resumes; correct position displayed; if turn arrived while backgrounded, pipeline starts on return | High |
| TC-SQ108 | Debounce | Double-tap send button doesn't create duplicate queue entry | Report ready to send | Rapidly tap "Send to Server" twice | Only one POST `/queue/enqueue` fires; second tap is debounced or blocked; single `sessionId` tracked | High |
| TC-SQ109 | Queue UI | Queue dialog cancel/back behavior | Waiting in queue (WAITING state) | Tap back or cancel on queue dialog | Confirmation prompt: "Leave queue?"; if confirmed, app calls POST `/queue/fail` to release slot; if dismissed, polling continues | Medium |
| TC-SQ110 | Queue UI | Immediate ACTIVE — no queue dialog flash | No other sessions in queue | Tap "Send to Server" | Enqueue returns ACTIVE; pipeline starts immediately; no visible queue dialog (or brief "Connecting…" then pipeline) | Medium |
| TC-SQ111 | Error | Enqueue API failure (network/server error) | Server unreachable or returns 500 | Tap "Send to Server" | Error dialog: "Could not connect to submission queue"; report stays unsent; retry available | High |
| TC-SQ112 | Error | Check-turn API failure during poll | Network drops mid-poll | Observe during polling | App retries with backoff; after N failures, shows "Connection lost" error; does not silently abandon queue | Medium |

### 2.3 Integration / E2E

| TC-ID | Category | Scenario | Preconditions | Steps | Expected Result | Priority |
|-------|----------|----------|---------------|-------|-----------------|----------|
| TC-SQ201 | E2E single | Full flow: single user enqueue → active → pipeline → complete | UAT server deployed; test supervisor account; report ready | 1. Tap Send → enqueue (ACTIVE immediately) 2. Pipeline runs (batch → hdr → dtls → accomp → materials) 3. Complete called | Queue session COMPLETED in DB; DAR data persisted correctly; report marked sent in app | Critical |
| TC-SQ202 | E2E multi | Full flow: 2 users — second waits, first completes, second proceeds | Two devices/emulators; both logged in as different supervisors; reports ready | 1. User A taps Send → ACTIVE 2. User B taps Send → WAITING (position 1) 3. User A pipeline completes → complete 4. User B polls → `isMyTurn: true` → pipeline runs → complete | Both sessions COMPLETED; both DAR reports persisted correctly; User B's wait time reflected in UI; FIFO order maintained | Critical |
| TC-SQ203 | E2E reaper | Full flow: active user crashes, stale reaper expires, next proceeds | Two users; User A is ACTIVE; User B is WAITING | 1. User A's app is force-killed (simulating crash) 2. Wait > 5 minutes for reaper 3. User B polls → becomes ACTIVE → pipeline → complete | User A's session EXPIRED in DB; User B's session COMPLETED; User B's DAR data persisted; no data corruption from User A's partial state | Critical |
| TC-SQ204 | E2E FIFO | 5 concurrent users maintain FIFO order | 5 devices/emulators; all logged in as different supervisors; reports ready | 1. All 5 tap Send within ~2 seconds 2. Observe queue positions 3. Each user's pipeline runs in FIFO order 4. All complete | 1 ACTIVE + 4 WAITING initially; each proceeds in enqueue order; all 5 sessions reach COMPLETED; all 5 DAR reports persisted; total queue time scales linearly | Critical |
| TC-SQ205 | E2E failover | Active user fails mid-pipeline, next user proceeds | Two users; User A is ACTIVE | 1. User A's pipeline fails at hdr step → fail called 2. User B polls → becomes ACTIVE → pipeline → complete | User A FAILED; User B COMPLETED; User A's partial data does not corrupt User B's submission; User A can retry (new enqueue) | High |
| TC-SQ206 | E2E retry | Failed user re-enqueues after failure | User A previously failed; User B is now ACTIVE | 1. User A taps Send again → new enqueue (WAITING behind B) 2. B completes 3. A becomes ACTIVE → pipeline → complete | New session for A; both sessions traceable in DB; A's report submitted successfully on second attempt | High |
| TC-SQ207 | E2E mixed ops | Queue handles different operation types | User A submitting Survey; User B submitting MPS (Bagging) | 1. A enqueues → ACTIVE 2. B enqueues → WAITING 3. A completes Survey pipeline 4. B proceeds with MPS pipeline | Both reports persisted with correct operation-specific data; queue is operation-agnostic | Medium |
| TC-SQ208 | E2E reaper chain | Reaper fires twice; third user eventually proceeds | A (ACTIVE, stale), B (WAITING, will also stale), C (WAITING) | 1. A expires by reaper 2. B becomes ACTIVE but also stales 3. B expires by reaper 4. C becomes ACTIVE → completes | A and B both EXPIRED; C COMPLETED; C's data intact | Medium |

---

## 3. Traceability Matrix

### 3.1 User Stories

| Story ID | Title | Acceptance Criteria Summary |
|----------|-------|-----------------------------|
| US-SQ01 | Enqueue submission | Supervisor's report is placed in a queue before pipeline starts; if no queue, immediate ACTIVE |
| US-SQ02 | Queue position visibility | Supervisor sees their position in queue and it updates in real-time |
| US-SQ03 | Automatic turn activation | When it's the supervisor's turn, the pipeline starts without manual intervention |
| US-SQ04 | Pipeline completion releases queue | Successful pipeline marks session COMPLETED and releases the slot for the next supervisor |
| US-SQ05 | Pipeline failure releases queue | Failed pipeline marks session FAILED and releases the slot; supervisor can retry |
| US-SQ06 | Stale session recovery | Crashed/abandoned ACTIVE sessions are expired after 5 minutes; queue unblocks automatically |
| US-SQ07 | Idempotent enqueue | Double-tap or duplicate enqueue returns existing session, not a new entry |
| US-SQ08 | Concurrent submission safety | Only one pipeline executes at a time; SQL locking prevents race conditions |
| US-SQ09 | Queue timeout handling | If a supervisor waits beyond the configured timeout, the app shows an error and allows retry |
| US-SQ10 | App lifecycle resilience | Backgrounding/foregrounding the app during queue wait does not lose the session |

### 3.2 Story → Test Case Mapping

| User Story | Test Cases (Backend) | Test Cases (Frontend) | Test Cases (Integration/E2E) | Priority |
|------------|----------------------|-----------------------|------------------------------|----------|
| US-SQ01 | TC-SQ001, TC-SQ002, TC-SQ018 | TC-SQ101, TC-SQ110 | TC-SQ201 | Critical |
| US-SQ02 | TC-SQ005, TC-SQ006 | TC-SQ102 | TC-SQ202, TC-SQ204 | Critical |
| US-SQ03 | TC-SQ004 | TC-SQ103 | TC-SQ201, TC-SQ202 | Critical |
| US-SQ04 | TC-SQ007 | TC-SQ104 | TC-SQ201, TC-SQ202, TC-SQ204 | Critical |
| US-SQ05 | TC-SQ008 | TC-SQ105 | TC-SQ205, TC-SQ206 | Critical |
| US-SQ06 | TC-SQ009, TC-SQ017, TC-SQ019 | — | TC-SQ203, TC-SQ208 | Critical |
| US-SQ07 | TC-SQ003 | TC-SQ108 | — | High |
| US-SQ08 | TC-SQ013, TC-SQ014, TC-SQ020 | — | TC-SQ204 | Critical |
| US-SQ09 | — | TC-SQ106 | — | High |
| US-SQ10 | — | TC-SQ107 | — | Medium |

### 3.3 Coverage Summary

| Priority | Backend TCs | Frontend TCs | E2E TCs | Total |
|----------|-------------|--------------|---------|-------|
| Critical | 8 | 4 | 4 | 16 |
| High | 8 | 4 | 2 | 14 |
| Medium | 4 | 4 | 2 | 10 |
| **Total** | **20** | **12** | **8** | **40** |

---

## 4. UAT Scenarios — Manual Stakeholder Testing

**Persona**: Farm supervisor (primary); Operations staff (observer).
**Environment**: MDAG-UAT server; physical device (field tablet or phone).
**Prerequisite**: Queue feature deployed; test supervisor accounts provisioned; at least 2 devices available.

### 4.1 Single Supervisor — Happy Path

| UAT-ID | Scenario | Steps | Expected Result | Pass / Fail | Tester | Notes |
|--------|----------|-------|-----------------|-------------|--------|-------|
| UAT-SQ01 | Submit report with empty queue | 1. Login as Supervisor A 2. Create Individual report (any operation) 3. Fill all required fields 4. Tap "Send to Server" | Report submits immediately with no visible wait; success confirmation displayed; report appears in Log as sent | ☐ | | Equivalent to pre-queue behavior |
| UAT-SQ02 | Verify no queue dialog when alone | Same as UAT-SQ01 | No "waiting in queue" dialog appears (or appears for < 1 second); pipeline runs smoothly | ☐ | | UX: should feel seamless |

### 4.2 Two Supervisors — Queue Wait

| UAT-ID | Scenario | Steps | Expected Result | Pass / Fail | Tester | Notes |
|--------|----------|-------|-----------------|-------------|--------|-------|
| UAT-SQ03 | Second supervisor sees queue position | 1. Supervisor A taps Send (goes ACTIVE) 2. Supervisor B taps Send (goes WAITING) 3. Observe B's screen | B sees "You are #1 in queue" (or equivalent); position updates while waiting | ☐ | | Requires 2 devices |
| UAT-SQ04 | Second supervisor proceeds after first completes | Continue from UAT-SQ03: 4. A's pipeline completes 5. Observe B's screen | B's queue dialog updates → pipeline starts → submits successfully; both reports in DB | ☐ | | Verify data integrity for both |
| UAT-SQ05 | Queue position feels responsive | During UAT-SQ03, time the updates on B's device | Position updates within ~3–5 seconds; no stale display | ☐ | | Poll interval = 3s |

### 4.3 Failure & Recovery

| UAT-ID | Scenario | Steps | Expected Result | Pass / Fail | Tester | Notes |
|--------|----------|-------|-----------------|-------------|--------|-------|
| UAT-SQ06 | First supervisor's app crashes; second unblocks | 1. A taps Send (ACTIVE) 2. B taps Send (WAITING) 3. Force-close A's app 4. Wait ~5 minutes 5. Observe B | B eventually becomes ACTIVE (after reaper); pipeline runs; report submits | ☐ | | Validate 5-min reaper timeout |
| UAT-SQ07 | Pipeline fails; retry works | 1. Trigger a pipeline failure (e.g., disconnect network mid-send) 2. Observe error dialog 3. Reconnect and tap Send again | Error dialog is clear and actionable; second attempt succeeds; no duplicate/corrupt data | ☐ | | Check tbl_SubmissionQueue for FAILED row |
| UAT-SQ08 | App backgrounded and resumed | 1. A taps Send (WAITING behind another user) 2. Switch to another app for 30 seconds 3. Return to DAR app | Queue dialog reappears with current position; no error; if turn arrived, pipeline starts | ☐ | | Field-realistic scenario |

### 4.4 Edge Cases & Stress

| UAT-ID | Scenario | Steps | Expected Result | Pass / Fail | Tester | Notes |
|--------|----------|-------|-----------------|-------------|--------|-------|
| UAT-SQ09 | Double-tap Send button | 1. Report ready 2. Rapidly double-tap "Send to Server" | Only one queue entry created; no duplicate dialogs; single submission flow | ☐ | | |
| UAT-SQ10 | Send while offline | 1. Disable network 2. Tap Send | Clear error: "No connection" or similar; no queue entry; report stays unsent | ☐ | | |
| UAT-SQ11 | 3+ supervisors queue simultaneously | 1. A sends (ACTIVE) 2. B sends (WAITING #1) 3. C sends (WAITING #2) 4. A completes → B proceeds → B completes → C proceeds | All three complete in order; data integrity for all reports | ☐ | | Needs 3 devices |

### 4.5 UAT Sign-Off

| Role | Responsibility | Status | Signature | Date |
|------|----------------|--------|-----------|------|
| **QA Lead** | All UAT scenarios executed; defects logged; go/no-go recommendation | ☐ Pending | | |
| **PM** | Release accepted; queue feature approved for production | ☐ Pending | | |
| **BA / PO** | Acceptance criteria met for US-SQ01 through US-SQ10 | ☐ Pending | | |
| **Dev-Senior** | Technical sign-off; SQL locking and reaper verified | ☐ Pending | | |

---

## 5. Defect & Resolution Flow

| Stage | Action | Owner |
|-------|--------|-------|
| **Logged** | Defect raised with severity (P1–P4), repro steps, and TC-ID reference | QA (any level) |
| **Triaged** | Assigned to backend-squad or frontend-squad; priority confirmed | QA Lead + Dev-Senior |
| **In Progress** | Fix developed on MDAG-339; unit test added | Dev (assigned) |
| **Verified** | Fix re-tested against original TC-ID; regression check on related TCs | QA (Mid/Senior) |
| **Closed** | Defect marked resolved; traceability updated | QA Lead |

### Severity Definitions

| Severity | Definition | SLA |
|----------|-----------|-----|
| **P1 — Blocker** | Queue allows concurrent ACTIVE sessions; data corruption; complete queue failure | Fix before next QA cycle |
| **P2 — Critical** | Reaper doesn't fire; position display wrong; pipeline doesn't start on turn | Fix within same sprint |
| **P3 — Major** | Queue dialog UX issues; timeout message unclear; minor position lag | Fix within next sprint |
| **P4 — Minor** | Cosmetic; logging gaps; edge case with no user impact | Backlog |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-04-14 | QA Lead | Initial test matrix for Submission Queue feature: strategy, 40 test cases, traceability, UAT scenarios |
