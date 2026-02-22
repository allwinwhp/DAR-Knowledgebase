# DAR — User Guide

**Purpose**: End-user guide for the DAR (Daily Accomplishment Report) system. Explains each module, workflows, form usage, and edge cases from the perspective of a farm supervisor. Reflects the **current implemented system**.

**Audience**: Farm supervisors, operations staff, stakeholders.  
**Repos**: DAR_Middleware (backend), main_dar_app (Flutter app). **Branch**: MDAG-339.

**Version**: 1.0 | **Date**: 2026-02-22

---

## 1. Introduction

### 1.1 What Is DAR?

**DAR** is an **input interface for farm supervisors** to record daily work done in the field. It supports:

- **Individual (solo)** reports — one operator per report
- **Gang/Group** reports — multiple operators per group

Reports go through operations such as:

| Category | Examples |
|----------|----------|
| **Fruit care (MPS)** | Bagging, Bud Capping, Deflowering, Propping, Hand Tubing, Leaf Pruning, Bud Bunch Spray, Bud Injection |
| **Plant care** | Weedspray, Fertilizer, CPMS, Chemical Mixing |
| **PDCM** | PD Survey, PD Eradication |
| **PPM** | Gouging, Sucker Pruning, Popcount, Planting/Replanting |
| **Others** | General Services (individual and group) |

### 1.2 Who Uses DAR?

- **Farm supervisor** — Primary user: records daily accomplishments in the field.
- **Operations / back office** — Syncs reference data, runs reports.
- **System** — API layer between app and DAR database.

---

## 2. Getting Started

### 2.1 Server Selection (Before Login)

1. On the **pre-login screen**, select a **server** from the dropdown.
2. Options typically include: MDAG-UAT (Cavite-Ngrok), Local (localhost), MDAG-UAT (Davao), MDAG-PROD (Davao).
3. The selected server is used as the API base URL for your entire session.

### 2.2 Login

1. Open the app → **Login** screen.
2. Enter **Username** and **Password**.
3. Tap **Login**.
4. On success, you see the **Home** screen.

**Edge cases**:
- Wrong credentials → Error message; no navigation.
- Offline → Login fails; retry when online.

### 2.3 Sign Up (Registration)

1. On the login screen, tap **Sign up**.
2. Select an **Employee** from the dropdown (must exist in the system).
3. Enter **Username** and **Password**.
4. Tap **Register**.
5. Username must be unique across all users.

---

## 3. Home Screen

### 3.1 Overview

The Home screen is your main hub. You can:

- **Filter** by Individual vs Gang/Group
- **Filter** by status (e.g. completed, incomplete, pending)
- **Search** reports
- **Sync** reference data (locations, parcels, materials, etc.)
- **Save / Download** operators to/from the server
- **Create** new reports

### 3.2 Report Type

| Type | Button | Description |
|------|--------|-------------|
| **Individual** | Create Individual Report | One operator per report. |
| **Gang/Group** | Create Gang Group | Multiple operators per group. |

Select the type before creating a new report.

---

## 4. Creating an Individual Report

### 4.1 Flow Overview

1. **Home** → **Create** (Individual)
2. **Operators** — Set individual operators
3. **Document** — Pick date and doctype (operation)
4. **Form** — Fill timekeeping, main details, accomplishments, materials
5. **Submit** — Send to server

### 4.2 Operators Screen

1. Go to **Operators** (from Create Individual).
2. Add or edit **individual operators** (list of operators for solo reports).
3. Tap **Save** to store locally and optionally sync to server via **Save to server**.

**Edge cases**:
- No operators → Can still create report; operator can be set in form or timekeeping.
- Sync failure → Operators saved locally; retry later.

### 4.3 Document Screen

1. Choose **date** (transaction date).
2. Choose **doctype** (operation type): Survey, MPS, Bagging, Gouging, Weedspray, Fertilizer, Chemical Mixing, Gen Serv, etc.
3. Create **report** → Opens the operation form.

### 4.4 Form — Individual Operations

Each operation has its own form. Common sections:

- **Timekeeping**: AM in/out, PM in/out (per operator)
- **Main details**: Farm, cost center, date, incentive (if applicable)
- **Accomplishments**: Parcel, layout, quantity
- **Materials**: Issued, used, returned (when the operation requires materials)

#### Individual Operations List

| Operation | Main Details | Accomplishments | Materials | Notes |
|-----------|--------------|-----------------|-----------|-------|
| Bagging | Farm, area, missout, date | Up to 10 rows | Up to 10 | QTY9/QTY10 set to 0 by system |
| Bud Capping | Farm, area, date | Up to 10 | Up to 10 | |
| Deflowering/Defingering | Farm, area, date | Up to 10 | — | No materials |
| Propping | Farm, area, date | Up to 10 | Up to 10 | |
| Hand Tubing | Farm, area, date | Up to 10 | Up to 10 | |
| Leaf Pruning FOR | Farm, area, date | Up to 10 | — | |
| Bud Bunch Spray | Farm, area, date | Up to 10 | Up to 10 | |
| Bud Injection | Farm, area, date | Up to 10 | Up to 3 | Max 3 materials |
| Chemical Mixing | Main details + mixing rows | — | Up to 10 | Operation code per mixing row |
| Weedspray | Farm, area, date, incentive | Up to 5 | Up to 10 (Chemical only) | Manual: no materials |
| Survey | Survey main details | Up to 20 (incl. bag week) | Up to 5 | |
| Sucker Pruning | Main details, incentive | Up to 5 | 1 | Cycle no. and Seq no. required |
| Gouging | Main details, incentive | Up to 5 | Yes | Incentive: S800 only |
| Gen Serv | Farm, cost center, incentive, date, op code | — | — | Main details only |

### 4.5 Form Fields — Individual

| Field | Purpose | Required | Notes |
|-------|---------|----------|-------|
| transDate | Transaction date | Yes | Date of work |
| farmCode | Farm / location | Yes | From sync |
| areaCode | Area (bagging, etc.) | Yes (where shown) | From sync |
| costCenter / costCenterCode | Cost center | Yes (where shown) | From tblActivity Active=1 |
| incentiveCode | Incentive | Yes (where shown) | Filter depends on operation (see PSD) |
| missout | Miss-out (bagging) | No | Bagging only |
| parCode | Parcel | Yes (accomplishment) | From parcel sync |
| layoutCode | Layout | Yes (accomplishment) | From layout sync |
| qty | Quantity | Yes (accomplishment) | Work quantity |
| itemCode, issued, used, returned | Material | Yes (where applicable) | Materials section |

---

## 5. Creating a Gang/Group Report

### 5.1 Flow Overview

1. **Home** → **Create** (Gang/Group)
2. **Group** — Create groups and assign operators
3. **Members** — Choose group members
4. **Document** — Pick date and doctype
5. **Form** — Fill timekeeping, main details, accomplishments, materials
6. **Submit** — Send to server

### 5.2 Group Screen

1. Go to **Group** (from Create Gang).
2. Create **groups** and assign **operators** to each group.
3. Tap **Save** to store; optionally **Save to server**.

**Edge cases**:
- No groups → Create at least one group before creating a report.
- Sync failure → Groups saved locally; retry later.

### 5.3 Form — Group Operations

| Operation | Main Details | Accomplishments | Materials | Notes |
|-----------|--------------|-----------------|-----------|-------|
| CPMS | Farm, cost center, incentive, date | Up to 15 | Up to 5 (decimal qty) | Material quantity accepts decimals |
| Fertilizer | Farm, cost center, incentive, date | Up to 20 | Up to 5 | Incentive: S800 (Opcode 25) |
| PD Eradication | Main details, incentive | Up to 20 (incl. bag week) | Up to 15 | Incentive: S800 |
| Popcount | Farm, cost center, date | Up to 10 | — | No incentive; HAS = layout actual area |
| Planting/Replanting | Farm, cost center, date | Up to 10 | Up to 10 | Cost center: 22190-LP or 14090 |
| Gen Serv Group | Farm, cost center, incentive, date, op code | — | — | Cost center/incentive per employee in timekeeping |

### 5.4 Timekeeping — Group (Gen Serv Group)

For **General Services Group**, **cost center** and **incentive** are entered **per employee** in the timekeeping section (not only in main details).

---

## 6. Form Usage — Step by Step

### 6.1 Timekeeping

1. Enter **AM in**, **AM out**, **PM in**, **PM out** for each operator or row.
2. Regular and overtime hours are calculated by the system from these times.
3. For **Gen Serv Group**, also set cost center and incentive per employee.

### 6.2 Main Details

1. Select **farm** (location) from dropdown.
2. Select **cost center** (where applicable) — from tblActivity Active=1.
3. Select **incentive** (where applicable) — filter depends on operation (see PSD or baseline).
4. Enter **date** if not pre-filled.

### 6.3 Accomplishments

1. Add rows: **parcel** (ParCode), **layout** (LayoutCode), **quantity** (QTY).
2. Operation-specific fields (e.g. variety, disease, HAS) where shown.
3. Max rows per operation — app enforces limits; server rejects excess.

### 6.4 Materials

1. Add rows: **item**, **issued**, **used** (and **returned** — computed when applicable).
2. Max rows per operation — app enforces limits; server rejects excess.
3. **CPMS**: Material quantity accepts decimal values.

### 6.5 Save vs Submit

- **Save**: Stores report locally in Hive. Status may update to complete/incomplete.
- **Submit**: Sends full sequence (batch → header → details → accomplishments → materials) to server. On success, report is marked sent and may move to Log.

---

## 7. Review, Log, and Send

### 7.1 Review Screen

- View reports before sending.
- Verify all required sections are filled.
- Use filters to find specific reports.

### 7.2 Log Screen

- View **sent** and **draft** reports.
- Status reflects whether the report was successfully submitted.

### 7.3 Send Flow

1. Select report(s) → **Send**.
2. App validates (e.g. userId, batchno, docno; server validation if configured).
3. Confirmation dialog may appear.
4. Submit runs full sequence to server.
5. Success → Report marked sent; may move to Log.

**Edge cases**:
- **Offline** → Send fails; report stays in queue; retry when online.
- **Partial failure** (e.g. batch created, header fails) → Backend may be inconsistent; report may not be fully saved. Retry or contact support.
- **Max accomplishments/materials** → Server returns 400; cannot add more rows.

---

## 8. Sync and Dropdown Behavior

### 8.1 What Syncs

- **Locations** (farm)
- **Parcels** (survey, other operations)
- **Layouts** (hectarage, survey, fruit care)
- **Materials** (by cost center)
- **Employees**
- **Cost centers** (activity)
- **Incentives** (by operation filter)
- **Fertilizer types, materials, cost centers**
- **Chemical mixing operations**
- **Varieties, disease types**
- **Holidays**

### 8.2 Incentive Filters (Summary)

| Operation | Filter | Expected |
|-----------|--------|----------|
| General Services | Doctype != 03, 01 | Full list |
| Gouging | Opcode 25 | S800 |
| Sucker Pruning | Icode RP13 | RP13 |
| Fertilizer | Opcode 25 | S800 |
| PD Eradication | Opcode 25 | S800 |
| CPMS | Opcode 25 | S800 |
| Chemical Mixing | Opcode 25 | S800 |
| Weedspray | Opcode 30 | Per DB |
| Popcount | — | **No incentive field** |
| Gen Serv (with op) | Opcode from selection | Match selection |

### 8.3 Cost Center

- Source: `tblActivity` where `Active = 1`.
- Used in main details and, for Gen Serv Group, in timekeeping per employee.

---

## 9. Edge Cases and System Constraints

### 9.1 Limits

| Item | Constraint |
|------|------------|
| Accomplishments | Per operation (e.g. Weedspray 5, Survey 20, Fertilizer 20) |
| Materials | Per operation (e.g. Weedspray 10, Survey 5, Fertilizer 5, Bud Injection 3) |
| Batch | One batch per user per date per doctype (idempotent) |
| Document number | Per user from tbldocno |

### 9.2 Validation

- **Required fields** must be filled before submit.
- **Timekeeping** must be complete (AM/PM times).
- **Max accomplishments/materials** enforced on server; 400 if exceeded.
- **Offline** — Local save works; submit requires connectivity.

### 9.3 Known Behaviors

- **Weedspray** — Manual: no materials required; Chemical: materials required. One accomplishment per report vs time per accomplishment — product decision (Jay J-PC-2).
- **Bagging** — QTY9/QTY10 forced to 0 by server.
- **Popcount** — Incentive field removed; HAS = layout actual area when sending.
- **ERAD** — Details use genserv3 route; materials use survey route (backend design).
- **Chemical Mixing** — Operation code per material row; tbl_MixingOprtn/costcenter mapping must match selection.

### 9.4 Server Selection

- Server choice affects all API calls for the session.
- Change requires new session (logout/login) or app restart to pick a different server.

---

## 10. Troubleshooting

| Issue | Likely Cause | Action |
|-------|--------------|--------|
| Login fails | Wrong credentials or offline | Check credentials; ensure network |
| Dropdown empty | Sync not done or API error | Sync from Home; check server selection |
| Header submission fails | Survey/Gouging/ERAD (known issues) | See Jay Feedback; ensure backend fixes applied |
| Max accomplishments | Limit reached | Cannot add more; submit with current rows |
| Max materials | Limit reached | Cannot add more; submit with current rows |
| Send fails | Offline or server error | Retry when online; check server URL |
| Wrong incentive list | Operation filter | Verify operation; use correct filter per PSD |

---

## 11. References

- [DAR Product Specification Document](dar-product-specification-document.md)
- [dar-user-journey-and-form-specifications](../../project/dar-user-journey-and-form-specifications.md)
- [dar-sync-queries-baseline](../../project/dar-sync-queries-baseline.md)
- [dar-uat-checklist](../../development/dar-uat-checklist.md)
- [dar-system-context](../../project/dar-system-context.md)
