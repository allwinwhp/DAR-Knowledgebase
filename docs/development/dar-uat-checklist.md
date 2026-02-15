# DAR UAT Test Case Checklist — Client-Side (Supervisor Persona)

**Purpose**: Client-side UAT test case checklist for the DAR system. Use with [dar-deployment-mop.md](dar-deployment-mop.md) and [dar-user-journey-and-form-specifications.md](../project/dar-user-journey-and-form-specifications.md). Stakeholder manual testing as a regular user; no release sign-off until UAT is done (or explicitly deferred with PM approval).

**References**: [design-sprint-facilitator](../../skills/design-sprint-facilitator/SKILL.md) (DISCOVER → DESIGN → DECIDE), [delivery-orchestrator](../../agents/delivery-orchestrator.md), [dar-system-context.md](../project/dar-system-context.md)

---

## 1. Feedback Baseline (Intake)

**Primary feedback doc**: [dar-mobile-feedback-baseline.md](../project/dar-mobile-feedback-baseline.md) — consolidated DAR Mobile Findings, Feb 2, Feb 5 (Cilan adds), Feb 6. **Whole team must review** per role assignments.

**Instructions**: Record additional stakeholder, QA, and pilot feedback here. Use the baseline doc + this table to derive UAT test cases. Update after each feedback cycle.

| # | Feedback source | Date | Summary | UAT test case(s) derived | Status |
|---|-----------------|------|---------|--------------------------|--------|
| FB-001 | DAR Mobile Findings Summary | Jan–Feb 2026 | Backend batchno/docno per user; UI filters; operation-specific rules | See dar-mobile-feedback-baseline.md §A–G | Baseline |
| FB-002 | Feb 2 Feedback | 2026-02-02 | Fruit care Area/Parcel/Layout queries; Survey/Erad parcel S; Gang/CPMS status | TC-OP01–OP05, TC-G04 | Baseline |
| FB-003 | Feb 5 Cilan adds | 2026-02-05 | Field saving, sync dialog, group employee recognition, Qty calcs, batch hours | TC-I06, TC-O03, TC-G05, TC-OP* | Baseline |
| FB-004 | Feb 6 Register/Send | 2026-02-06 | 1:1 user–account; Send to Server validation flow | TC-L02, TC-S03 | Baseline |
| FB-005 | *(Add next feedback)* | | | | Open |

### Feedback categories to capture

- [ ] **Usability** — Navigation, clarity of labels, form flow
- [ ] **Functional** — Login, sync, report creation, submission, data integrity
- [ ] **Offline / connectivity** — Behavior when offline, reconnection
- [ ] **Performance** — Load times, responsiveness on field devices
- [ ] **Data validation** — Required fields, format, edge cases
- [ ] **Integration** — API responses, error handling, sync behavior

---

## 2. Team Input — E2E Test Suite (Supervisor Persona)

**Instructions**: QA Lead, BA, Frontend, Backend, and SM add inputs below for end-to-end test scenarios. Each role contributes from their perspective; QA Lead consolidates into the final E2E suite.

### 2.1 QA Lead input

| # | E2E scenario | Preconditions | Steps (high-level) | Expected outcome | Priority |
|---|--------------|---------------|--------------------|------------------|----------|
| QA-001 | | | | | |
| QA-002 | | | | | |

### 2.2 BA / Product input

| # | User story / acceptance criteria | E2E scenario to cover |
|---|----------------------------------|------------------------|
| BA-001 | | |
| BA-002 | | |

### 2.3 Frontend (Flutter) input

| # | Screen / flow | Edge case or regression risk | E2E scenario |
|---|---------------|------------------------------|--------------|
| FE-001 | | | |
| FE-002 | | | |

### 2.4 Backend / API input

| # | Endpoint / contract | E2E scenario to validate |
|---|----------------------|---------------------------|
| BE-001 | | |
| BE-002 | | |

### 2.5 SM input

| # | Definition of Done item | UAT checkpoint |
|---|--------------------------|----------------|
| SM-001 | | |
| SM-002 | | |

---

## 3. UAT Test Strategy (Design Sprint Aligned)

Per [design-sprint-facilitator](../../skills/design-sprint-facilitator/SKILL.md) and [reference.md](../../skills/design-sprint-facilitator/reference.md):

| Test type | Scope | Owner | Notes |
|-----------|-------|-------|-------|
| **Functional** | Core flows per user journey | QA | Login, create report, submit, sync |
| **Integration** | API + Flutter; middleware + DB | QA / Backend | Form submission sequence, sync endpoints |
| **Regression** | Prior sprint fixes | QA | Re-test known issues from feedback baseline |
| **UAT** | Stakeholder as regular user | BA / PO / PM | Manual testing per this checklist |

### Traceability

| User journey (dar-user-journey) | UAT section | TC-ids |
|---------------------------------|-------------|--------|
| Login & entry | §4.1 | TC-L01 … |
| Create report (individual) | §4.2 | TC-I01 … |
| Create report (gang/group) | §4.3 | TC-G01 … |
| Operators & report sync | §4.4 | TC-O01 … |
| Review, log, send | §4.5 | TC-S01 … |

---

## 4. UAT Test Cases — Supervisor Persona (Client-Side)

### 4.1 Login & Entry

| TC-id | Scenario | Steps | Expected result | Pass / Fail | Notes |
|-------|----------|-------|-----------------|-------------|-------|
| TC-L01 | Splash loads | Open app | Splash screen displays, then Login (or Sign up) | ☐ | |
| TC-L02 | Login with valid credentials | Enter Username, Password → Login | POST /user/login succeeds; Home displays | ☐ | |
| TC-L03 | Login with invalid credentials | Enter wrong credentials → Login | Error message; no navigation to Home | ☐ | |
| TC-L04 | Home report type filter | On Home, verify Individual vs Gang/Group buttons | Both options visible; user can select | ☐ | |
| TC-L05 | Session persistence | Login → background app → return | Session retained or graceful re-login prompt | ☐ | |

### 4.2 Create Report — Individual (Solo)

| TC-id | Scenario | Steps | Expected result | Pass / Fail | Notes |
|-------|----------|-------|-----------------|-------------|-------|
| TC-I01 | Navigate to create Individual report | Home → Create (Individual) → Operators | Operators screen loads; individual operators list | ☐ | |
| TC-I02 | Set individual operators | Add/edit operators on Operators screen | Operators saved (local or via save-report) | ☐ | |
| TC-I03 | Create report — Document | Navigate to Document; pick date, doctype | IndividualReport created in Hive; form flow available | ☐ | |
| TC-I04 | Fill timekeeping | Enter AM in/out, PM in/out | Timekeeping saved; maps to dtls | ☐ | |
| TC-I05 | Fill main details | Farm, cost center, dates | Header fields populated correctly | ☐ | |
| TC-I06 | Fill accomplishments | Enter operation-specific accomplishments | Accomplishments saved; validates max per config | ☐ | |
| TC-I07 | Fill materials (if required) | Enter materials for op types that need them | Materials saved; validation applied | ☐ | |
| TC-I08 | Submit Individual report | Submit full form | Batch → hdr → dtls → accomp → materials sequence completes; success feedback | ☐ | |
| TC-I09 | Offline create (Individual) | Create report offline | Local save; queue for sync when online | ☐ | |

### 4.3 Create Report — Gang/Group

| TC-id | Scenario | Steps | Expected result | Pass / Fail | Notes |
|-------|----------|-------|-----------------|-------------|-------|
| TC-G01 | Navigate to create Gang report | Home → Create (Gang/Group) → Group | Group screen loads; groups and operators | ☐ | |
| TC-G02 | Create group and assign operators | Create group; assign operators to group | Group with operators saved | ☐ | |
| TC-G03 | Create GroupReport | Select group, date, doctype; create report | GroupReport created; form flow available | ☐ | |
| TC-G04 | Fill form for group | Timekeeping, details, accomplishments, materials | All fields map correctly for group context | ☐ | |
| TC-G05 | Submit Group report | Submit full form | Submission sequence completes; data in UAT DB | ☐ | |
| TC-G06 | Offline create (Group) | Create group report offline | Local save; queue for sync | ☐ | |

### 4.4 Operators & Report Sync

| TC-id | Scenario | Steps | Expected result | Pass / Fail | Notes |
|-------|----------|-------|-----------------|-------------|-------|
| TC-O01 | Save operators to server | Operators screen → Save | POST /report/save-report with individual_operators and/or group_operators | ☐ | |
| TC-O02 | Load operators from server | Open Operators (or sync) | POST /report/get-report; individual_operators, group_operators displayed | ☐ | |
| TC-O03 | Sync reference data | Trigger sync (or on login) | Operation config, locations, parcels, materials, etc. synced from /api/* | ☐ | |
| TC-O04 | Sync failure handling | Simulate API failure | Clear error; no crash; retry option | ☐ | |

### 4.5 Review, Log, Send

| TC-id | Scenario | Steps | Expected result | Pass / Fail | Notes |
|-------|----------|-------|-----------------|-------------|-------|
| TC-S01 | Review before send | Open Review; inspect report(s) | Report(s) displayed; user can verify before send | ☐ | |
| TC-S02 | Log — view sent/draft reports | Open Log | Sent and draft reports visible; status correct | ☐ | |
| TC-S03 | Send report | Select report(s) → Send | Full submission sequence; success; status updated | ☐ | |
| TC-S04 | Send failure handling | Simulate API failure during send | Error shown; report remains in queue; retry possible | ☐ | |

### 4.6 Operation-Specific (Sample — Expand per MVP)

| TC-id | Operation | Scenario | Expected result | Pass / Fail | Notes |
|-------|-----------|----------|-----------------|-------------|-------|
| TC-OP01 | Survey | Create and submit Survey report | Survey form submits; accomp/materials correct | ☐ | |
| TC-OP02 | MPS | Create and submit MPS report (any subtype) | MPS form submits; bagging/bud-capping/etc. | ☐ | |
| TC-OP03 | Fertilization | Create and submit Fertilization report | Fertilizer form submits | ☐ | |
| TC-OP04 | Chemical Mixing | Create and submit Chem-mixer report | Materials and mixing ops validated | ☐ | |
| TC-OP05 | Harvest | Create and submit Harvest report | Harvest hdr/dtls/accomp/materials | ☐ | |

---

## 5. End-to-End Test Suite — Supervisor Persona (Consolidated)

**Use after team inputs (§2) are gathered.** QA Lead consolidates into executable E2E scenarios.

| E2E-id | Scenario name | Journey | Steps | Traceability |
|--------|---------------|---------|-------|--------------|
| E2E-S01 | Full Individual flow | Login → Create Individual → Operators → Report → Submit → Review/Log | TC-L01, L02, L04, I01–I08, O01, S01–S03 | |
| E2E-S02 | Full Gang flow | Login → Create Gang → Group → Report → Submit → Log | TC-L01, L02, L04, G01–G05, O01, S01–S03 | |
| E2E-S03 | Sync and offline resilience | Login → Sync → Offline create → Online send | TC-L02, O01, O03, O04, I09, S04 | |
| E2E-S04 | Multi-operation day | Login → Individual Survey → Individual MPS → Send both | TC-I08, OP01, OP02, S03 | |

---

## 6. Sign-Off

| Role | Responsibility | Sign-off |
|------|----------------|----------|
| **QA Lead** | UAT executed per checklist; defects logged; go/no-go | |
| **PM** | Release accepted; handover or next phase | |
| **BA / PO** | Acceptance criteria met for agreed scope | |

**UAT completion**: All critical (P1) test cases passed; known issues documented; PM confirms go-live or deferral.

---

## Document control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | (fill) | QA Lead + PM | Initial UAT checklist; feedback baseline; team input structure; supervisor persona E2E. |
