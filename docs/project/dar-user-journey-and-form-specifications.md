# DAR — User Journey & Form Specifications

**Purpose**: Unlocked user journey and form specifications for the DAR system. Use with [dar-system-context.md](dar-system-context.md). Backend = Node.js (DAR_Middleware), Frontend = Flutter (main_dar_app). Reports are **individual/solo** or **gang/group**.

---

## 1. Report Types

| Type | Backend / API | Flutter | Description |
|------|----------------|--------|-------------|
| **Individual / Solo** | `reportType: 'individual'`, `groupId: null` | `IndividualReport`, Hive box `individual_reports` | One operator per report; supervisor assigns individual operators. |
| **Gang / Group** | `reportType: 'group'`, `groupId` required | `GroupReport`, Hive box `group_reports` | Multiple operators per group; supervisor assigns operators to a group (e.g. gang). |

- **Operator assignment** is stored in `tblsupervisor_operators` (middleware) and synced via `/report/save-report` and `/report/get-report`.
- **DAR form submission** (batch → hdr → dtls → accomp → materials) is the same flow for both; the report type affects how operators are chosen and displayed (individual list vs groups).

---

## 2. User Journey (High Level)

### 2.1 Login & entry

1. **Splash** → **Login** (or Sign up).
2. **Login**: POST `/user/login` with `Username`, `Password`. Response: `Empno`, `Compcde`, `Paytype`, `Empnem`, `userId`, `username`, etc.
3. **Home**: User sees create/send options and **report type filter**: **Individual** vs **Gang / Group** (from `AppConstants.individualButtonTitle` / `gangGroupButtonTitle`).

### 2.2 Create report (individual)

1. Home → **Create** (Individual) → **Operators** (`/operator`).
2. Supervisor sets **individual operators** (list of operators for solo reports).
3. Navigate to **Document** (or similar) to pick date/doctype and create a **report** (IndividualReport) in Hive.
4. **Form** flow: open form by operation (e.g. Survey, MPS, Fertilizer). Report holds: `supervisor`, `operator`, `timekeeping`, `doctype`, `commonBaseForm` (operation-specific form).
5. Fill **timekeeping** (AM in/out, PM in/out, etc.), **main details** (farm, cost center, dates), **accomplishments**, **materials** (if required).
6. **Submit**: Flutter calls middleware in order — see §3 (Form submission sequence).

### 2.3 Create report (gang/group)

1. Home → **Create** (Gang / Group) → **Group** (`/group`).
2. Supervisor creates **groups** and assigns **operators** to each group (`groupId` + list of operators).
3. Report type is **GroupReport**; structure parallels IndividualReport but with group/operator set.
4. Form flow and submit sequence are analogous; backend still receives batch/hdr/dtls/accomp/materials per operation.

### 2.4 Operators & report sync

1. **Operators** screen: manage **individual** list or **group** list.
2. **Save to server**: POST `/report/save-report` with `supervisorId`, `individual_operators[]` and/or `group_operators[]` (each group has `groupId`, `operators[]`).
3. **Load from server**: POST `/report/get-report` with `supervisorId`; response: `individual_operators`, `group_operators`.

### 2.5 Review, log, send

1. **Review** (`/review`): review reports before final send.
2. **Log** (`/log`): view/log sent or draft reports (IndividualReportService / GroupReportService, log services).
3. **Send**: for each report, run the full form submission sequence to middleware; on success, update report status (e.g. completed) and optionally remove from local queue or log.

---

## 3. Form Submission Sequence (Backend Contract)

Every DAR form submission follows this order. Flutter must call in sequence; IDs from each step are used in the next.

| Step | Action | Endpoint (example MPS/Survey) | Key request body | Response |
|------|--------|-------------------------------|------------------|----------|
| 1 | Get or create batch | POST `/sendDARForm/utilities/getExistingBatchId` | `Entrydate`, `Doctype`, `userid` | `Batch_id` or null |
| 2 | Create batch (if none) | POST `/sendDARForm/batch/tblDARbatch` | `Compcde`, `Yer`, `Weekno`, `Doctype`, `Transdate`, `Entrydate`, `userid`, … | `id.Batch_id` |
| 3 | Create header | POST `/sendDARForm/<operation>/tblDARhdr` | `Batch_id`, `TransNo`, `Paytype`, `PHCode`, `FarmCode`, `Username`, `Entrydate`, … | `id.Hdr_id` |
| 4 | Create details (per operator/timekeeping row) | POST `/sendDARForm/<operation>/tblDARdtls` | `Hdr_id`, `Batch_id`, `Empno`, `CstCtr`, `AMin`, `AMout`, `PMin`, `PMout`, … | `id.Dtl_id` (or similar) |
| 5 | Create accomplishments (per row) | POST `/sendDARForm/<operation>/tblDARaccomp` | `Hdr_id`, `Batch_id`, `ParCode`, `LayoutCode`, `OpCode`, `Empno`, `HAS`, `QTY`, … | — |
| 6 | Create materials (if operation has materials) | POST `/sendDARForm/<operation>/tblDARmaterials` | `Hdr_id`, `Batch_id`, material fields | — |

- **&lt;operation&gt;** is one of: `mps`, `survey`, `fertilization`, `utility`, `utility2`, `harvest`, `erad-ppm`, `chem-mixer`, `ph`, `pseudostem-cgms`, `sigatoka`, `popcount`, `genserv`, `genserv3`, `bag-week` (and any other mounted under `sendDARForm`).
- **Doctype** (batch level) identifies the document/operation type (e.g. numeric or string codes; see seeds and `tbl_OperationConfig`).
- After accomplishments (and materials) are inserted, backend **auto-calculates** header `Qty1` and batch `Accomp`/`AccompEntered` via `autoCalculate.js` (and related utilities).

---

## 4. Form Specifications (Backend)

### 4.1 Batch (tblDARbatch)

- **Required**: `Compcde`, `Yer`, `Weekno`, `Doctype`, `Transdate`, `Entrydate`, `userid`.
- **Optional**: `TtlDocs`, `TtlRegHrs`, `TtlOTHrs`, `Accomp`, `TtlDocsEntered`, `TtlRegHrsEntered`, `TtlOTHrsEntered`, `AccompEntered`.
- **Backend behavior**: If a batch already exists for `(Entrydate, Doctype, userid)`, returns existing `Batch_id`; otherwise increments `tblbatchno` per user, resolves `Daycode` (holiday/Sunday/default), inserts into `tblDARbatch`.

### 4.2 Header (tblDARhdr)

- **Required**: `Batch_id`, `TransNo`, `Paytype`, `PHCode`, `FarmCode`, `Username`, `Entrydate`.
- **Optional**: `TtlRegHrs`, `TtlRegOT`, `TtlNoRecs`, `Incentive1`, `Qty1`, `Missout`, `CstCtr`, `TLCode`, `Ptype`, `Weather`, `Drift`.
- **Backend behavior**: Resolves `DocumentNo` from `tbldocno` (per userid from batch); inserts into operation-specific `tblDARhdr` (e.g. mps, survey).

### 4.3 Details (tblDARdtls)

- **Required** (typical): `Hdr_id`, `Batch_id`, `AMin`, `AMout`, `PMin`, `PMout`, `Empno`, `CstCtr`.
- **Optional**: `No`, `Type`; backend may compute `RegHrs`, `RegOTHrs` from times (`compHrs`).
- **Backend behavior**: `SeqNo` auto-incremented per (Hdr_id, Batch_id).

### 4.4 Accomplishments (tblDARaccomp)

- **Required** (operation-dependent): `Hdr_id`, `Batch_id`, `ParCode`, `LayoutCode`, `OpCode`, `Empno`. Often `QTY` or `HAS` (and operation-specific fields).
- **Optional**: `HAS`, `QTY`, `QTY9`, `QTY10`, `ColCode`, `AgeWeek`, `Variety`, `DiseaseType`, `DetectionStage`, `PlantAgeCls`, `Weekbagged`, `AreaCode`.
- **Backend behavior**: Validates max accomplishments via `tbl_OperationConfig` (Doctype + OpCode); after insert, triggers `updateHdrQty1` and `updateBatchAccomp`. Bagging operations may set QTY9/QTY10 = 0.

### 4.5 Materials (tblDARmaterials)

- **When present**: Chem-mixer, fertilization, MPS, survey, pseudostem-cgms, erad-ppm, etc. (see `validateOperation.requiresMaterials` and operation-specific routes).
- **Backend behavior**: Validates max materials per header/batch; Chemical Mixing (Doctype 08) has OpCode per material row.

### 4.6 Report (operator assignment)

- **Save**: POST `/report/save-report` — body: `supervisorId`, `individual_operators[]` (each: `compCde`, `payType`, `empNo`, `empNem`), `group_operators[]` (each: `groupId`, `operators[]` with same fields). Replaces all existing rows for that supervisor.
- **Get**: POST `/report/get-report` — body: `supervisorId`. Response: `individual_operators[]`, `group_operators[]` (each group: `groupId`, `operators[]`).

---

## 5. Flutter Form Model (Alignment)

- **IndividualReport**: `id`, `status`, `supervisor`, `operator`, `timekeeping`, `doctype`, `commonBaseForm` (operation-specific form, e.g. SurveyForm, BudCappingForm).
- **GroupReport**: analogous with group/operator set.
- **CommonBaseForm** implementations map to middleware operations: Survey, MPS (bagging, bud capping, bud injection, deflowering, hand tubing, leaf pruning, propping, bud bunch spray), Weedspray, Fertilizer, Chemical Mixing, CPMS, Gouging, Sucker Pruning, Gen Serv, Erad, etc.
- **Timekeeping**: AM in/out, PM in/out → maps to dtls (and compHrs for RegHrs/RegOTHrs).
- **Doctype** in Flutter aligns with backend `Doctype` (batch) and operation routes (e.g. `AppDoctypes.survey` → survey endpoints).

---

## 6. References

- **Backend review**: [dar-backend-squad-review.md](dar-backend-squad-review.md)
- **Frontend review**: [dar-frontend-squad-review.md](dar-frontend-squad-review.md)
- **System context**: [dar-system-context.md](dar-system-context.md)
