# UAT Checklist — Submission Queue

**Sprint**: Sprint-Submission-Queue
**Stakeholder**: Farm operations supervisor (primary tester)
**Environment**: Physical server (DAR_Middleware) + Android device (main_dar_app)
**Prerequisite**: Backend deployed with queue endpoints; new APK installed on test device

---

## Pre-UAT Setup

- [ ] `tbl_SubmissionQueue` migration applied on server
- [ ] DAR_Middleware running with `/queue/*` endpoints live
- [ ] New APK (with queue feature) installed on at least 2 test devices
- [ ] Test supervisor accounts provisioned (minimum 2 different supervisors)
- [ ] Reports created locally on each device, ready to send

---

## Scenarios

### Single User — Happy Path

| # | Scenario | Steps | Expected | Pass/Fail | Notes |
|---|----------|-------|----------|-----------|-------|
| UAT-01 | Submit with empty queue | 1. Login as Sup A 2. Create report 3. Tap "Send to Server" | Report submits immediately; success shown; no queue wait visible | ☐ | |
| UAT-02 | No queue dialog flash | Same as UAT-01 | No "waiting in queue" dialog appears (or < 1 second); pipeline runs normally | ☐ | |

### Two Supervisors — Queue Wait

| # | Scenario | Steps | Expected | Pass/Fail | Notes |
|---|----------|-------|----------|-----------|-------|
| UAT-03 | Second user sees queue position | 1. Sup A taps Send (ACTIVE) 2. Sup B taps Send (WAITING) 3. Observe B's screen | B sees "You are #N in line"; position updates every ~3 seconds | ☐ | |
| UAT-04 | Second user proceeds after first | Continue: 4. A's pipeline completes 5. Observe B | B's dialog updates → pipeline starts → submits successfully | ☐ | |
| UAT-05 | Position updates feel responsive | During UAT-03, time the updates | Position updates within 3–5 seconds | ☐ | |

### Failure & Recovery

| # | Scenario | Steps | Expected | Pass/Fail | Notes |
|---|----------|-------|----------|-----------|-------|
| UAT-06 | First user's app crashes | 1. A sends (ACTIVE) 2. B sends (WAITING) 3. Force-close A 4. Wait ~5 min 5. Observe B | B eventually becomes ACTIVE after reaper fires | ☐ | |
| UAT-07 | Pipeline fails; retry works | 1. Trigger failure (e.g., disconnect network) 2. See error 3. Reconnect, tap Send again | Error dialog shown; retry succeeds; no duplicate data | ☐ | |
| UAT-08 | App backgrounded during wait | 1. A waits in queue 2. Switch apps 30 sec 3. Return | Queue dialog shows current position; no error | ☐ | |

### Edge Cases

| # | Scenario | Steps | Expected | Pass/Fail | Notes |
|---|----------|-------|----------|-----------|-------|
| UAT-09 | Double-tap Send | Rapidly double-tap "Send to Server" | Only one queue entry created; single flow | ☐ | |
| UAT-10 | Send while offline | 1. Disable network 2. Tap Send | Error shown; no queue entry; report stays unsent | ☐ | |
| UAT-11 | Three supervisors queue | 1. A sends (ACTIVE) 2. B sends (WAITING #1) 3. C sends (WAITING #2) 4. Complete in FIFO order | All three complete in order; data integrity verified | ☐ | |

---

## Sign-Off

| Role | Status | Signature | Date |
|------|--------|-----------|------|
| QA Lead | ☐ Pending | | |
| PM | ☐ Pending | | |
| PO / BA | ☐ Pending | | |
| Dev-Senior | ☐ Pending | | |
