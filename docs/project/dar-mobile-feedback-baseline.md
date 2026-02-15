# DAR Mobile — Feedback Baseline (Team Review)

**Purpose**: Consolidated feedback from DAR mobile findings and pilot testing. The **whole team** should review and sign off on their sections. Use this to drive UAT test cases and sprint backlog items.

**Related**: [dar-uat-checklist.md](../development/dar-uat-checklist.md) | [dar-user-journey-and-form-specifications.md](dar-user-journey-and-form-specifications.md) | [dar-deployment-mop.md](../development/dar-deployment-mop.md)

---

## Team Review Instructions

| Role | Sections to review | Action |
|------|--------------------|--------|
| **Backend / Architect** | A, D (register/send flow) | Confirm feasibility; add to backlog; estimate |
| **Frontend (Flutter)** | B, C, E, F, G, H | Confirm feasibility; map to screens; estimate |
| **QA Lead** | All | Derive UAT test cases; add to dar-uat-checklist.md |
| **BA / PO** | All | Prioritize; confirm business rules |
| **PM / SM** | All | Track completion; resolve conflicts |

**Review deadline**: *(fill)*  
**Sign-off**: Each role checks off their section when reviewed.

---

## A. Back End

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| A1 | Each mobile user must have their own range of `batchno` and `documentno`. `tblaccounts` and `tblusers` must be related. Using one `userid` (e.g. 93) for all users causes all reports of same doctype/day to save in one batch. | Backend | TC-O01, TC-S03 | ☐ Reviewed |
| A2 | Each user gets `batchno` from `tblbatchno`. Each user has assigned batchno. | Backend | TC-S03 | ☐ Reviewed |
| A3 | Each user gets `documentno` from `tbldocno`. Each user has assigned documentno. | Backend | TC-S03 | ☐ Reviewed |
| A4 | Total accomplishment per report must be saved in `tbldarhdr.Qty1`. When no accomplishment, set to zero (not null). | Backend | TC-I08, TC-G05 | ☐ Reviewed |
| A5 | Total accomplishment of **all** reports in batch saved in `tbldarbatch.Accomp`. Currently only first document/report. When no accomplishments, set to zero. | Backend | TC-I08, TC-G05 | ☐ Reviewed |
| A6 | Additional tables needed for sync data. | Backend / Architect | TC-O03 | ☐ Reviewed |

---

## B. UI — General

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| B1 | **Employee lookup**: Filter only active — `WHERE Delcd = 'A'`. | Frontend | TC-I01, TC-G02 | ☐ Reviewed |
| B2 | **Layout lookup**: Filter only active — `WHERE Active = 'A'`. | Frontend | TC-I06 | ☐ Reviewed |
| B3 | **Numeric fields** (accomplishments, missout): Set value = 0 when null or empty. | Frontend | TC-I06, TC-OP* | ☐ Reviewed |
| B4 | **Regular hours**: Currently fixed 6:00AM–11:30 and 12:00PM–2:30. Need flexibility — some operations start at 5:00AM. | Frontend | TC-I04 | ☐ Reviewed |
| B5 | **Materials report**: Automatically compute `Returned` = Issued − Used. | Frontend | TC-I07, TC-OP* | ☐ Reviewed |
| B6 | **General services**: Request group entry. Mobile DAR Genserv form is individual only. | Frontend | TC-G01, TC-G03 | ☐ Reviewed |
| B7 | Accomplishments not recorded in `tbldarhdr.Qty1` and `tbldarbatch.Accomp` (same as Reghours/OT hours flow). | Backend + Frontend | TC-I08 | ☐ Reviewed |
| B8 | **Field saving**: Fields with 0 value not saved when non-Parcel X and non-Layout X selected; form appears blank after save. | Frontend | TC-I06 | ☐ Reviewed |
| B9 | **Sync dialog**: Add text indicators for `tblhectarageMPS` and `tblhectarage` sync. | Frontend | TC-O03 | ☐ Reviewed |

---

## C. Fruit Care Operations

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| C1 | **Areacode** in Header; **Parcel** in details/accomplishments. One area can have 2 parcela. | Frontend | TC-OP02 | ☐ Reviewed |
| C2 | **Area filter query** (use provided query): `tbl_hectarageMPS`, `tbl_hectarageArea`, `tbloperation` — filter `M.Active=1`, `A.Active=1`, `Parcel != 'X'`. | Backend | TC-I05 | ☐ Reviewed |
| C3 | **Parcel filter query** (use provided query): Same base, add `Areacode` for cascade. | Backend | TC-I05 | ☐ Reviewed |
| C4 | **Layout filter query** (use provided query): Same base, add `Parcel` for cascade. | Backend | TC-I05 | ☐ Reviewed |
| C5 | **Hand Tubing**: Cannot enter materials. Investigate. | Frontend | TC-OP02 | ☐ Reviewed |
| C6 | **Bagging**: Qty calculation must total 9H + 10H for all accomplishments. Bud/Bunch field = 0 in Bagging (not total of 9H+10H). | Frontend | TC-OP02 | ☐ Reviewed |
| C7 | **Leaf Pruning**: Qty calculation must total HAS for all accomplishments. | Frontend | TC-OP02 | ☐ Reviewed |
| C8 | **Materials expansion** (Bagging, Bud Capping, Bud and Bunch Spray, Hand Tubing, Propping): Expand to 10 materials. | Frontend | TC-I07 | ☐ Reviewed |

---

## D. Other Operations — Parcel/Layout Filters

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| D1 | **Parcel** (Weedspray, Fertilizer, CPMS, etc.): `SELECT Parcel FROM tblparcel WHERE CompCode=@Compcde AND FarmCode=@Farmcode AND Parcel != 'S'`. | Backend | TC-OP03 | ☐ Reviewed |
| D2 | **Layout** (same ops): `SELECT Layoutcode, Layout, ActArea FROM tblhectarage WHERE CompCode=@Compcde AND FarmCode=@Farmcode AND Parcel=@Parcel AND Active=1`. | Backend | TC-OP03 | ☐ Reviewed |
| D3 | **Survey & Erad** use parcel `'S'` automatically. Layout: `Parcel = 'S' AND Active = 1`. | Backend | TC-OP01 | ☐ Reviewed |

---

## E. Register, Sync & User Assignment (Feb 6 Feedback)

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| E1 | **1:1 constraint**: One user in `tblusers` ↔ one account in `tblaccounts`. Enforce in DB and app. | Backend | TC-L02 | ☐ Reviewed |
| E2 | **Register screen** — Two tabs: (1) Account Creation, (2) Connect User ID. | Frontend | TC-L02 | ☐ Reviewed |
| E3 | **Tables to sync**: `tblemployee`, `tblusers`, `tblaccounts`, `tblbatchno`, `tbldocno`. | Backend | TC-O03 | ☐ Reviewed |
| E4 | **Account creation**: Empno, Paytype (dropdown), Username, Password. Save to `tblaccounts`. | Frontend | TC-L02 | ☐ Reviewed |
| E5 | **Connect User ID tab**: List accounts; admin selects employee; assigns User ID from `tblusers`. Validate: User ID not already assigned; account not already assigned (allow reassignment). | Frontend + Backend | TC-L02 | ☐ Reviewed |
| E6 | **Send to Server validation (STEP 1)**: If `userId` NULL/empty in `tblaccounts` → modal: "Cannot submit. User ID not assigned. Contact IT Team." Block submission. | Frontend | TC-S03 | ☐ Reviewed |
| E7 | **Send to Server validation (STEP 2)**: If no `batchno` or `docno` for `userId` in `tblbatchno`/`tbldocno` → modal: "Missing batch or document numbers. Contact IT Team." Block submission. | Frontend | TC-S03 | ☐ Reviewed |
| E8 | **Send to Server flow (STEP 3)**: Retrieve batchno, docno; increment both; UPDATE tables; build payload; submit. | Backend | TC-S03 | ☐ Reviewed |

---

## F. Gang/Group & Employee Recognition

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| F1 | **Gang/CPMS**: Status always "incomplete" unless Time/Shift is clicked or updated. | Frontend | TC-G04 | ☐ Reviewed |
| F2 | **Group report — Employee Not Found**: 3 employees in gang; only 2 sent to DB. Investigate. | Backend + Frontend | TC-G05 | ☐ Reviewed |
| F3 | **Half-day time entries** not recognized (possible empty string issue). | Frontend | TC-I04, TC-G04 | ☐ Reviewed |

---

## G. Operation-Specific Rules

### Weedspray

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G1 | Max accomplishments = 5. | Backend | TC-OP* | ☐ Reviewed |
| G2 | Weed control Manual: no materials — always shows incomplete. OK (expected). Weed control Chemical: has materials. | Frontend | TC-OP* | ☐ Reviewed |
| G3 | Qty header: must total HAS for accomplishments. | Frontend | TC-OP* | ☐ Reviewed |
| G4 | Materials: expand to 10. | Frontend | TC-OP* | ☐ Reviewed |
| G5 | Incentive field: filter from `tblincentive`; display code (not name). Optional: Spray Equipment, Drifts, Weather. | Frontend | TC-OP* | ☐ Reviewed |

### Chemical Mixing

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G6 | Max Mixing Details/Materials = 10. | Backend | TC-OP04 | ☐ Reviewed |
| G7 | Operation lookup: `tbl_MixingOprtn`; Chemical: `tbl_ChemMixing`. Filter operation by selected tblactivity. | Backend | TC-OP04 | ☐ Reviewed |
| G8 | No accomplishment; set `tbldarbatch.Accomp=0`, `AccompEntered=0` (not null). | Backend | TC-OP04 | ☐ Reviewed |
| G9 | Add Cost Center field in details/header. Query per tblactivity. | Frontend | TC-OP04 | ☐ Reviewed |
| G10 | Incentive field: filter from `tblincentive`; display code. | Frontend | TC-OP04 | ☐ Reviewed |
| G11 | Batch total hours not calculated in Mixing Batch. | Backend | TC-OP04 | ☐ Reviewed |

### Gouging

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G12 | Add "Hills" accomplishment for gouging. | Frontend | TC-OP* | ☐ Reviewed |

### Sucker Pruning

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G13 | Max accomplishments = 5. | Backend | TC-OP* | ☐ Reviewed |
| G14 | Add materials max = 1; optional — do not include in incomplete validation. | Frontend | TC-OP* | ☐ Reviewed |

### Survey

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G15 | Max accomplishments = 20. | Backend | TC-OP01 | ☐ Reviewed |
| G16 | Filter parcel where `Parcel = 'S'`. | Backend | TC-OP01 | ☐ Reviewed |
| G17 | Add `AreaType` in accomplishment: "N" (Normal), "SO" (Squared Off). | Frontend | TC-OP01 | ☐ Reviewed |
| G18 | Missing fields: RS and SE. | Frontend | TC-OP01 | ☐ Reviewed |
| G19 | Numeric fields (New, Adjacent, Recurrence, Early, Late, Unshot, Shot): 0 when null/empty. | Frontend | TC-OP01 | ☐ Reviewed |
| G20 | Bagweek: Survey uses `tbl_Bagweek`; Erad uses `tbl_BagWeek2` (was interchanged). | Backend | TC-OP01 | ☐ Reviewed |

### Erad

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G21 | Accomplishment max = 20; materials max = 15. | Backend | TC-OP* | ☐ Reviewed |
| G22 | Bagweek in `tbl_BagWeek2`; filter layout `Active='A' AND Parcel='S'`. Cases Eradicated and Treatment are different. | Backend | TC-OP* | ☐ Reviewed |
| G23 | Eradication must connect to Survey: calculate remaining cases (Surveyed not yet eradicated) when sending to server. | Backend | TC-OP* | ☐ Reviewed |

### PPM

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G24 | Add: Replanting (Fallow Area) BT, BBM, BSV; Haul Fertilizer; Haul TC; Mortality. | Frontend | TC-OP* | ☐ Reviewed |

### CPMS / Pseudo

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G25 | Accomplishment max = 15 (max reached 13 in records). | Backend | TC-OP* | ☐ Reviewed |
| G26 | Materials max = 5. | Backend | TC-OP* | ☐ Reviewed |
| G27 | Qty header: total HAS. Batch total hours not calculated. | Frontend | TC-OP* | ☐ Reviewed |

### Fertilization

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G28 | Accomplishment max = 20; materials max = 5. | Backend | TC-OP03 | ☐ Reviewed |
| G29 | Cost center lookup: `tblActivity WHERE Doctype = 10`. Some cost centers don't display. | Backend | TC-OP03 | ☐ Reviewed |
| G30 | Materials/fertilizers lookup: check query — some don't display. | Backend | TC-OP03 | ☐ Reviewed |
| G31 | Incentive field: filter; display code. Qty header: total accomplishments. | Frontend | TC-OP03 | ☐ Reviewed |

### Popcount

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G32 | Add: Replant Follow Area – BW; Replant Follow Area – PD; Missing Hills Follow Area – BW; Missing Hills Follow Area – PD. | Frontend | TC-OP* | ☐ Reviewed |

### Bud Inject

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| G33 | Max materials = 3. | Backend | TC-OP* | ☐ Reviewed |

---

## H. Time Calculation & Batch Hours

| # | Finding | Owner | UAT TC | Status |
|---|---------|-------|--------|--------|
| H1 | **Time calculation fix** (per Cilan adds). | Frontend | TC-I04 | ☐ Reviewed |
| H2 | **Utility Batch**: Total hours not calculated. | Backend | TC-OP* | ☐ Reviewed |
| H3 | **PD Pseudo Batch**: Total hours not calculated. | Backend | TC-OP* | ☐ Reviewed |

---

## Send to Server — Validation Flow (Reference)

```
User: Click "Send to Server"
  |
  v
VALIDATION 1: userId in tblaccounts?
  NO → Modal: "Contact IT Team - No User ID" → STOP
  YES
  |
  v
VALIDATION 2: batchno in tblbatchno for userId?
  NO → Modal: "Contact IT Team - Missing Batch/Doc Numbers" → STOP
  YES
  |
  v
VALIDATION 3: docno in tbldocno for userId?
  NO → Modal: "Contact IT Team - Missing Batch/Doc Numbers" → STOP
  YES
  |
  v
PROCESS: Get batchno, docno → increment → UPDATE tblbatchno, tbldocno
  |
  v
PROCESS: Build payload (username, userid, batchno, docno, ...)
  |
  v
ACTION: Submit to Server → SUCCESS
```

---

## Queries Reference (Fruit Care — Area/Parcel/Layout)

**Area filter**:
```sql
SELECT DISTINCT A.Areacode, A.Area, A.AreaTotal
FROM dbo.tbl_hectarageMPS AS M
INNER JOIN dbo.tbl_hectarageArea AS A ON M.Area_id = A.Area_id
INNER JOIN dbo.tbloperation AS O ON A.Opcode = O.Opcode
WHERE M.Active = 1 AND A.Active = 1 AND Parcel != 'X'
  AND A.FarmCode = @Farmcode AND A.Opcode = @Operationcode
```

**Parcel filter** (add `A.Areacode = @Areacode`):
```sql
SELECT DISTINCT A.Areacode, A.Area, A.FarmCode, A.Opcode, M.Parcel
FROM dbo.tbl_hectarageMPS AS M
INNER JOIN dbo.tbl_hectarageArea AS A ON M.Area_id = A.Area_id
INNER JOIN dbo.tbloperation AS O ON A.Opcode = O.Opcode
WHERE M.Active = 1 AND A.Active = 1 AND Parcel != 'X'
  AND A.FarmCode = @Farmcode AND A.Opcode = @Operationcode
  AND A.Areacode = @Areacode
```

**Layout filter** (add `M.Parcel = @Parcel`):
```sql
SELECT A.Areacode, A.Area, A.FarmCode, A.Opcode, A.AreaTotal, O.Opdesc,
       M.Sqno, M.Parcel, M.Layoutcode, M.Layout, M.Oxbow, M.ActArea
FROM dbo.tbl_hectarageMPS AS M
INNER JOIN dbo.tbl_hectarageArea AS A ON M.Area_id = A.Area_id
INNER JOIN dbo.tbloperation AS O ON A.Opcode = O.Opcode
WHERE M.Active = 1 AND A.Active = 1 AND Parcel != 'X'
  AND A.FarmCode = @Farmcode AND A.Opcode = @Operationcode
  AND A.Areacode = @Areacode AND M.Parcel = @Parcel
```

---

## Team Sign-Off

| Role | Reviewed by | Date | Notes |
|------|-------------|------|-------|
| Backend / Architect | | | |
| Frontend (Flutter) | | | |
| QA Lead | | | |
| BA / PO | | | |
| PM / SM | | | |
| **Business Unit** | | | |

---

## Document control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | (fill) | QA Lead | Consolidated DAR Mobile Findings, Feb 2 Feedback, Feb 5 Cilan adds, Feb 6 Register/Send flow. |
