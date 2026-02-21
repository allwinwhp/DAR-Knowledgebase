# Sprint-Jay-Feedback — UAT Checklist

**Purpose**: Scenarios for **stakeholder manual testing** as a regular user (farm supervisor persona).  
**Owner**: QA Lead + PM.  
**Reference**: [test-matrix.md](test-matrix.md), [scope.md](scope.md), [baseline-expected-results.md](baseline-expected-results.md), [dar-sync-queries-baseline.md](../../dar-sync-queries-baseline.md).

---

## Environment

- **Backend**: Use current live ngrok setup (per deployment-plan.md). Ensure app base URL points to this backend during UAT.
- **Frontend**: Latest build from sprint branch (main_dar_app); emulator or device.

---

## Pre-conditions

- [ ] Backend (DAR_Middleware) running with ngrok DB/config.
- [ ] App installed and launchable.
- [ ] If server selection is implemented: select the UAT server before login.
- [ ] **Sync completed** at least once (e.g. from Home) so local reference data (activity, incentive, etc.) matches backend.

---

## UAT scenarios

### 1. Server selection (TS-SERVER)

| Step | Action | Expected |
|------|--------|----------|
| 1.1 | Open app; locate server dropdown (before or on login screen). | Dropdown visible with list of servers. |
| 1.2 | Select a server (e.g. UAT). | Selection stored as base URL. |
| 1.3 | Enter credentials; login. | Login request goes to selected base URL. |
| 1.4 | After login, trigger sync or send a report. | Same base URL used. |

### 2. General Services — costcenter and incentive (US-J1)

| Step | Action | Expected |
|------|--------|----------|
| 2.1 | Open an operation that uses costcenter (e.g. General Services). | Costcenter list loads (from tblActivity Active=1). |
| 2.2 | Open incentive dropdown where applicable. | Incentives shown exclude Harvest/Fruit care (Doctype 03/01). |

### 3. PD Survey (US-J2)

| Step | Action | Expected |
|------|--------|----------|
| 3.1 | Create PD Survey; add up to 5 materials. | Up to 5 materials allowed. |
| 3.2 | Submit batch then header; complete flow. | Header submission succeeds; data sent to server. |

### 4. Gouging (US-J3)

| Step | Action | Expected |
|------|--------|----------|
| 4.1 | Open Gouging; select incentive (S800 / Opcode 25). | Only applicable incentives shown. |
| 4.2 | Submit batch and header. | Header and full flow succeed. |

### 5. Sucker Pruning (US-J4)

| Step | Action | Expected |
|------|--------|----------|
| 5.1 | Enter Cycle no. and Seq no. | Fields accept values. |
| 5.2 | Select incentive (RP13 only). | Only RP13 shown. |
| 5.3 | Submit. | Cycle and Sqno saved in header on server. |

### 6. Chemical Mixing (US-J5)

| Step | Action | Expected |
|------|--------|----------|
| 6.1 | Select operation; submit. | Operation code on server matches selection. |

### 7. Weedspray (US-J6)

| Step | Action | Expected |
|------|--------|----------|
| 7.1 | Create report per product decision (one accomplishment per report or time per accomplishment). | Data consistent with rule. |

### 8. CPMS (US-J7)

| Step | Action | Expected |
|------|--------|----------|
| 8.1 | Enter decimal in material quantity. | Decimal accepted and saved. |

### 9. Fertilizer (US-J8)

| Step | Action | Expected |
|------|--------|----------|
| 9.1 | Check form: incentive present, Operations field absent. | Incentive (Opcode 25) available; Operations removed. |
| 9.2 | Submit. | Incentive saved. |

### 10. General Services Group (US-J9)

| Step | Action | Expected |
|------|--------|----------|
| 10.1 | Add timekeeping rows; set costcenter and incentive per employee. | Costcenter and incentive per row; filters same as Individual. |

### 11. PD Eradication (US-J10)

| Step | Action | Expected |
|------|--------|----------|
| 11.1 | Select incentive; add accomplishments; submit. | Incentive filter applied; accomplishments saved on server. |

### 12. Planting/Replanting (US-J11)

| Step | Action | Expected |
|------|--------|----------|
| 12.1 | Select header costcenter (two options). | Selection saved in header. |

### 13. Popcount (US-J12)

| Step | Action | Expected |
|------|--------|----------|
| 13.1 | Select layout; submit. | HAS = layout actual area; no incentive field on form. |

---

## QA sign-off: Options per form (baseline)

**Purpose**: Verify each form’s dropdown options match [baseline-expected-results.md](baseline-expected-results.md) and [dar-sync-queries-baseline.md](../../dar-sync-queries-baseline.md). Complete after sync; use same backend/DB as baseline where possible.

| # | Form | Dropdown | Expected (baseline) | Pass | Notes |
|---|------|----------|---------------------|------|--------|
| 1 | **Any (costcenter)** | Costcenter | Options from tblActivity Active=1 (e.g. baseline ~317). No restricted list. | ☐ | |
| 2 | **General Services** | Incentive | Full list: Doctype ≠ 03/01 (e.g. 5 options per baseline). Not filtered by cost center. | ☐ | |
| 3 | **Gouging** | Incentive | Opcode 25 only: **1** option (S800 Chemical Handler). | ☐ | |
| 4 | **Sucker Pruning** | Incentive | Icode RP13 only (1 option if DB has RP13). | ☐ | |
| 5 | **Fertilizer** | Incentive | Opcode 25 only: **1** option (S800). Optional field present. | ☐ | |
| 6 | **PD Eradication** | Incentive | Opcode 25 only: **1** option (S800). | ☐ | |
| 7 | **CPMS** | Incentive | Opcode 25 only: **1** option (S800). | ☐ | |
| 8 | **Chemical Mixing** | Incentive | Opcode 25 only: **1** option (S800). | ☐ | |
| 9 | **Weedspray** | Incentive | Opcode 30 (e.g. P14 WEEDSPRAY). | ☐ | |
| 10 | **Popcount** | Incentive | **No incentive dropdown** on form (field removed). | ☐ | |
| 11 | **Popcount** (card) | — | Card summary does not show incentive. | ☐ | |

**QA Lead / QA Senior sign-off (options per form)**  
- [ ] All rows above verified; any failure logged and retested after fix.  
- [ ] Sign-off table below completed.

---

## Regression (smoke)

- [ ] Fruit care: still works (no regression).
- [ ] Other operations not in scope: quick smoke test.

---

## Sign-off

| Role | Name | Date | Notes |
|------|------|------|--------|
| Stakeholder (UAT) | | | |
| QA Lead | | | |
| QA (options per form) | | | *Section "QA sign-off: Options per form" completed.* |
| PM | | | |

*Release should not be signed off until UAT is done or explicitly deferred by PM.*

*Options-per-form verification*: QA must complete the "QA sign-off: Options per form (baseline)" table and check the sign-off box before marking QA complete for this sprint.
