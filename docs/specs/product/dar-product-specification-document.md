# DAR — Product Specification Document (PSD)

**Purpose**: Formal Product Specification Document (PSD) grounded in the current implementation. Single source of truth for forms, fields, validation, and behavioral specifications. Aligned with [product-master-foundations](../../skills/product-master-foundations/SKILL.md) and [dar-system-context](../../project/dar-system-context.md).

**Repos**: DAR_Middleware (Node.js/Express), main_dar_app (Flutter). **Branch**: MDAG-339.

**Version**: 1.0 | **Date**: 2026-02-22

---

## 1. Document Scope

This PSD profiles every form, every field, behavioral specifications, and change traceability. It reflects the **current implemented system**, including deviations from earlier design and newly introduced requirements (Jay Feedback II, Sprint Jay Feedback).

---

## 2. Form Profiles

### 2.1 Authentication Forms

#### 2.1.1 Login Form

| Attribute | Value |
|-----------|-------|
| **Form Name** | Login Form |
| **Purpose** | Authenticate farm supervisor for DAR input interface. |
| **Location** | Flutter: `lib/app/screen/login/widgets/login_form/login_form.dart` |
| **Backend Endpoint** | POST `/user/login` |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| username | string | Yes | — | Non-empty | User identifier | Must exist in tblaccounts | tblaccounts.Username |
| password | string | Yes | — | Non-empty | Password | bcryptjs hash compared | tblaccounts (hashed) |

**Submission**:
- POST body: `{ Username, Password }`
- Response: `{ Empno, Compcde, Paytype, Empnem, userId?, username, ... }` from tblaccounts + tblemployee + tblusers
- Error: Invalid credentials → 401 or error message; no navigation to Home

---

#### 2.1.2 Sign Up Form

| Attribute | Value |
|-----------|-------|
| **Form Name** | Sign Up Form |
| **Purpose** | Register employee as DAR user. |
| **Location** | Flutter: `lib/app/screen/sign_up/widgets/sign_up_form.dart` |
| **Backend Endpoint** | POST `/user/register` |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Empno | string | Yes | — | Must exist in tblemployee | Employee number | Employee must exist | tblemployee.Empno |
| Paytype | string | Yes | — | Auto from employee | Pay type | Used for tblaccounts | tblemployee |
| Username | string | Yes | — | Unique across tblaccounts | Login ID | Unique constraint | tblaccounts.Username |
| Password | string | Yes | — | Non-empty; hashed | Credentials | bcryptjs hash before insert | tblaccounts |

**Submission**:
- POST body: `{ Empno, Paytype, Username, Password }`
- Response: 201 `{ message, Empno, Paytype }`
- Backend links tblaccounts to tblusers via userid where matching

---

#### 2.1.3 Server Selection (Pre-Login)

| Attribute | Value |
|-----------|-------|
| **Form Name** | Server Selection |
| **Purpose** | Select API base URL before login (TS-SERVER, Jay Feedback II). |
| **Location** | Flutter: Config.serverOptions |
| **Behavior** | Dropdown; selected value set via `Config.setBaseUrl(url)` and used for entire session |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| label | string | Yes | — | — | Display label (e.g., MDAG-UAT) | Shown in dropdown | Config.serverOptions |
| baseUrl | string | Yes | — | Valid URL | API base URL | Used as Config.baseURL | Config.serverOptions |

---

### 2.2 Report / Operator Forms

#### 2.2.1 Operators (Individual List)

| Attribute | Value |
|-----------|-------|
| **Form Name** | Individual Operators |
| **Purpose** | Manage list of individual operators for solo reports. |
| **Backend** | POST `/report/save-report`, POST `/report/get-report` |

**Payload**:
- Save: `{ supervisorId, individual_operators[] }` — each: `compCde`, `payType`, `empNo`, `empNem`
- Get: `{ supervisorId }` → `{ individual_operators[] }`

**Behavior**: Replaces all existing rows for supervisor on save; scoped by supervisorId.

---

#### 2.2.2 Operators (Group List)

| Attribute | Value |
|-----------|-------|
| **Form Name** | Group Operators |
| **Purpose** | Manage groups and assign operators to each group. |
| **Backend** | POST `/report/save-report`, POST `/report/get-report` |

**Payload**:
- Save: `{ supervisorId, group_operators[] }` — each: `groupId`, `operators[]` with compCde, payType, empNo, empNem
- Get: `{ supervisorId }` → `{ group_operators[] }`

---

### 2.3 DAR Form Entities (Backend)

#### 2.3.1 Batch (tblDARbatch)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Group DAR documents by date, doctype, and user. |
| **Source** | DAR_Middleware `sendDARForm/batch/tblDARbatch.js`, `getExistingBatchId.js` |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Compcde | string | Yes | — | — | Company code | From user session | tblaccounts |
| Yer | string | Yes | — | — | Year | Transaction year | Derived |
| Weekno | string | Yes | — | — | Week number | ISO week | Derived |
| Doctype | string | Yes | — | — | Document type | Operation code (e.g. Survey, MPS) | tbl_OperationConfig |
| Transdate | string (date) | Yes | — | — | Transaction date | Date of work | User input |
| Entrydate | string (date) | Yes | — | — | Entry date | Date of data entry | User input |
| userid | string | Yes | — | — | User ID | From tblusers | tblusers.userid |
| TtlDocs | number | No | null | — | Total documents | Batch total | Computed |
| TtlRegHrs | number | No | null | — | Total regular hours | Batch total | Computed |
| TtlOTHrs | number | No | null | — | Total OT hours | Batch total | Computed |
| Accomp | number | No | null | — | Accomplishment | Batch total | Computed |
| TtlDocsEntered, TtlRegHrsEntered, TtlOTHrsEntered, AccompEntered | number | No | null | — | Entered totals | Running totals | Computed |
| Batchno | number | Server | — | — | Batch number | From tblbatchno per user | tblbatchno |
| Daycode | string | Server | '01'/'02' | tblholiday | Holiday/Sunday code | '01'=weekday, '02'=Sunday | tblholiday |

**Behavior**:
- `getExistingBatchId`: POST `{ Entrydate, Doctype, userid }` → returns existing `Batch_id` or null
- Batch creation: If existing for (Entrydate, Doctype, userid), return existing Batch_id (idempotent)
- Otherwise: Increment tblbatchno, resolve Daycode, insert tblDARbatch

---

#### 2.3.2 Header (tblDARhdr)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Document header per report; holds farm, cost center, incentive, etc. |
| **Source** | `sendDARForm/<operation>/tblDARhdr.js` (per operation) |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Batch_id | number | Yes | — | FK tblDARbatch | Batch reference | Must exist | tblDARbatch.Batch_id |
| TransNo | string | Yes | — | — | Transaction number | From tbldocno per user | tbldocno |
| Paytype | string | Yes | — | — | Pay type | From operator/report | Report |
| PHCode | string | Yes | — | — | Pay header code | — | Report |
| FarmCode | string | Yes | — | — | Farm / location | From location sync | tblLocation |
| Entrydate | string (date) | Yes | — | — | Entry date | Same as batch | Batch |
| Username | string | No | — | — | Username | From session | tblaccounts |
| TtlRegHrs, TtlRegOT | number | No | — | — | Totals | From dtls | calculateTotalsHdr |
| TtlNoRecs | number | No | — | — | Detail count | From dtls | calculateTotalsHdr |
| Incentive1 | string | No | — | — | Incentive code | From incentive sync | tblincentives |
| Qty1 | number | No | — | — | Accomplishment total | From accomp | autoCalculate |
| Missout | number | No | — | — | Miss-out | Bagging | User |
| CstCtr | string | No | — | — | Cost center | From activity sync | tblActivity |
| TLCode | string | No | — | — | Team lead | — | User |
| Ptype | string | No | — | — | Pay type variant | — | User |
| Cycle, Sqno | string/number | No | — | — | Cycle/Seq (Sucker Pruning) | J-PPM-2 | tblDARhdr |
| DocumentNo | number | Server | — | — | Document number | Per user from tbldocno | tbldocno |

**Dependencies**: Incentive dropdown filters per operation (see §4 Sync/Dropdown Rules). Cost center from tblActivity Active=1.

---

#### 2.3.3 Details (tblDARdtls)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Timekeeping rows per operator (AM/PM in/out). |
| **Source** | `sendDARForm/<operation>/tblDARdtls.js` |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Hdr_id | number | Yes | — | FK tblDARhdr | Header reference | Must exist | tblDARhdr.Hdr_id |
| Batch_id | number | Yes | — | FK tblDARbatch | Batch reference | Must exist | tblDARbatch |
| AMin | string (time) | Yes | — | — | AM in | Time format | User |
| AMout | string (time) | Yes | — | — | AM out | Time format | User |
| PMin | string (time) | Yes | — | — | PM in | Time format | User |
| PMout | string (time) | Yes | — | — | PM out | Time format | User |
| Empno | string | No | — | — | Employee number | From operator | Report |
| CstCtr | string | No | — | — | Cost center (per employee) | J-G-3 Gen Serv Group | tblActivity |
| No | number | No | — | — | Row number | — | — |
| Type | string | No | — | — | Row type | — | — |
| RegHrs, RegOTHrs | number | No | Computed | compHrs | Regular/OT hours | From AM/PM times | compHrs |
| SeqNo | number | Server | max+1 | — | Sequence | Auto-increment | — |

**Dependencies**: J-G-3 — Gen Serv Group: cost center and incentive per employee in timekeeping.

**Note**: ERAD uses `genserv3/tblDARdtls`; ERAD has no dedicated dtls route.

---

#### 2.3.4 Accomplishments (tblDARaccomp)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Record work done per parcel/layout/operation. |
| **Source** | `sendDARForm/<operation>/tblDARaccomp.js`; validateOperation.js |

**Common Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Hdr_id | number | Yes | — | FK | Header reference | Must exist | tblDARhdr |
| Batch_id | number | Yes | — | FK | Batch reference | Must exist | tblDARbatch |
| ParCode | string | Yes | — | — | Parcel code | From parcel sync | tblparcel |
| LayoutCode | string | Yes | — | — | Layout code | From layout sync | tblhectarage |
| OpCode | string | Yes | — | tbl_OperationConfig | Operation code | Valid for doctype | tbl_OperationConfig |
| Empno | string | Yes | — | — | Employee | From operator | Report |
| HAS | number | No | 0 | — | Hectarage/area | Layout actual area (Popcount J-G-6) | tblhectarage |
| QTY | number | No | 0 | — | Quantity | Work quantity | User |
| QTY9, QTY10 | number | No | 0 | Bagging: forced 0 | Extra quantities | Bagging: 0 | User / Bagging rule |
| SeqNo | number | Server | max+1 | — | Sequence | Auto | — |

**Operation-Specific** (survey, erad-ppm): ColCode, AgeWeek, Variety, DiseaseType, DetectionStage, PlantAgeCls, Weekbagged, AreaCode, NoCSurvey, NoCErad, Treated, New, Adjacent, etc.

**Validation**: Max accomplishments from tbl_OperationConfig (MaxAccomplishments). 400 when limit reached.

**Side Effects**: On insert → updateHdrQty1(Hdr_id), updateBatchAccomp(Batch_id).

---

#### 2.3.5 Materials (tblDARmaterials)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Record materials issued, used, returned per report. |
| **Source** | `sendDARForm/<operation>/tblDARmaterials.js` |

**Fields**:

| Field Name | Data Type | Required | Default | Validation | Description | Expected Behavior | Source |
|------------|-----------|----------|---------|------------|-------------|-------------------|--------|
| Hdr_id | number | Yes | — | FK | Header reference | Must exist | tblDARhdr |
| Batch_id | number | Yes | — | FK | Batch reference | Must exist | tblDARbatch |
| ItemCode | string | Yes | — | — | Material code | From material sync | tbl_materials |
| Issued | number | Yes | — | — | Issued quantity | User input | User |
| Used | number | Yes | — | — | Used quantity | User input; decimal for CPMS (J-G-1) | User |
| Returned | number | Server | Issued - Used | — | Returned | Computed | — |
| FertType | string | No | — | — | Fertilizer type | Fertilizer op | tbl_FertilizerType |
| SlipNo | string | No | — | — | Slip number | Harvest | User |
| SeqNo | number | Server | max+1 | — | Sequence | Auto | — |

**Validation**: Max materials from tbl_OperationConfig (MaxMaterials). 409 on constraint violations.

---

### 2.4 Operation-Specific Forms (Flutter)

#### 2.4.1 Individual Operation Forms

| Form | Operation | Max Accomplishments | Max Materials | Main Details Fields | Notes |
|------|-----------|---------------------|---------------|---------------------|-------|
| BaggingForm | Bagging | 10 | 10 | farmCode, areaCode, missout, transDate | QTY9/QTY10=0 on backend |
| BudCappingForm | Bud Capping | 10 | 10 | farmCode, areaCode, transDate | |
| DefloweringDefingeringForm | Deflowering | 10 | — | farmCode, areaCode, transDate | No materials |
| ProppingForm | Propping | 10 | 10 | farmCode, areaCode, transDate | |
| HandTubingForm | Hand Tubing | 10 | 10 | farmCode, areaCode, transDate | |
| LeafPruningFORForm | Leaf Pruning FOR | 10 | — | farmCode, areaCode, transDate | |
| BudBunchSprayForm | Bud and Bunch Spray | 10 | 10 | farmCode, areaCode, transDate | |
| BudInjectionForm | Bud Injection | 10 | 3 | farmCode, areaCode, transDate | |
| ChemicalMixingForm | Chemical Mixing | — | 10 | MainDetails, MixingDetails | OpCode in materials (J-PC-1) |
| WeedsprayForm | Weed Spray | 5 | 10 | MainDetails | Manual: no materials; Chemical: materials |
| SurveyForm | PD Survey | 20 | 5 | SurveyMainDetails | J-PD-1: material max 5; header fix |
| SuckerPruningForm | Sucker Pruning | 5 | 1 | MainDetails | Cycle/Sqno (J-PPM-2); Icode RP13 |
| GougingForm | Gouging | 5 | commonMaterial | MainDetails | Incentive Opcode 25 (J-PPM-1) |
| GenServForm | General Services | — | — | farmCode, costCenter, incentiveCode, transDate, opCode | Costcenter tblActivity; Incentive Doctype != 03,01 |

#### 2.4.2 Group Operation Forms

| Form | Operation | Max Accomplishments | Max Materials | Main Details | Notes |
|------|-----------|---------------------|---------------|--------------|-------|
| CPMSForm | CPMS | 15 | 5 | MainDetails | Material quantity decimal (J-G-1) |
| FertilizerForm | Fertilizer | 20 | 5 | farmCode, costCenterCode, transDate, incentiveCode | Add incentive Opcode 25 (J-G-2); remove Operations |
| EradForm | PD Eradication | 20 | 15 | EradMainDetails | Incentive Opcode 25; accomplishments fix (J-G-4) |
| PopCountForm | Popcount | 10 | — | MainDetails | Remove incentive (J-G-6); HAS=layout actual area |
| PlantingReplantingForm | Planting/Replanting | 10 | 10 | MainDetails | Add costcenter dropdown (J-G-5) |
| GenServForm (Group) | Gen Serv Group | — | — | MainDetails | Costcenter/incentive per employee (J-G-3) |

---

## 3. Behavioral Specifications

### 3.1 Form Submission Sequence

1. **getExistingBatchId**: POST `{ Entrydate, Doctype, userid }` → Batch_id or null
2. **Create batch** (if none): POST `/sendDARForm/batch/tblDARbatch` → Batch_id
3. **Create header**: POST `/<operation>/tblDARhdr` → Hdr_id
4. **Create details**: POST `/<operation>/tblDARdtls` (per operator/timekeeping row)
5. **Create accomplishments**: POST `/<operation>/tblDARaccomp` (per row)
6. **Create materials** (if applicable): POST `/<operation>/tblDARmaterials`
7. **calculateTotalsHdr**: PUT with `{ Hdr_id }`
8. **calculateTotalsBatch**: PUT with `{ Batch_id }`

Operations: mps, survey, fertilization, utility, utility2, harvest, erad-ppm, popcount, chem-mixer, pseudostem-cgms, sigatoka, genserv, genserv3, ph, bag-week.

### 3.2 Error Handling

| Scenario | Response | Behavior |
|----------|----------|----------|
| Missing required field | 400 | Error message; no insert |
| Batch_id not found | 404 | Error message |
| Max accomplishments exceeded | 400 | validateMaxAccomplishments message |
| Max materials exceeded | 400 | validateMaxMaterials message |
| Constraint violation | 409 | DB error |
| Server/DB error | 500 | error, details (generic in prod) |

### 3.3 State Changes & Side Effects

- **tblDARaccomp insert**: updateHdrQty1(Hdr_id), updateBatchAccomp(Batch_id)
- **tblDARhdr insert**: Updates tbldocno for userid
- **tblDARbatch insert**: Updates tblbatchno for userid; checks tblholiday
- **save-report**: DELETE + INSERT for supervisor operators

### 3.4 Role-Based Access

- No auth middleware on sendDARForm or report endpoints (per backend review). Recommendation: Add JWT/session and enforce on state-changing endpoints.

---

## 4. Sync / Dropdown Rules (Source: dar-sync-queries-baseline, Jay Feedback)

| Form | Dropdown | API | Filter / Expected |
|------|----------|-----|-------------------|
| All | Costcenter | GET /api/activity | tblActivity Active=1 |
| General Services | Incentive | GET /api/incentive | Doctype != '03','01' |
| Gouging | Incentive | GET /api/incentive?opcode=25 | 1 row S800 |
| Sucker Pruning | Incentive | GET /api/incentive?icode=RP13 | RP13 only |
| Fertilizer | Incentive | GET /api/incentive?opcode=25 | S800 |
| PD Eradication | Incentive | GET /api/incentive?opcode=25 | S800 |
| CPMS | Incentive | GET /api/incentive?opcode=25 | S800 |
| Chemical Mixing | Incentive | GET /api/incentive?opcode=25 | S800 |
| Weedspray | Incentive | GET /api/incentive?opcode=30 | Opcode 30 |
| Popcount | Incentive | — | **Remove** (J-G-6) |
| Gen Serv (with op) | Incentive | GET /api/incentive?opcode=&lt;selected&gt; | Match selected op |
| Planting/Replanting | Costcenter | — | Add: 22190-LP, 14090 (J-G-5) |

---

## 5. Change Traceability

### 5.1 Newly Introduced Requirements (Jay Feedback II)

| ID | Change | Modified/Extended Behavior | Status |
|----|--------|----------------------------|--------|
| J-OTH-1 | Costcenter filter | tblActivity Active=1 for all ops | Backend |
| J-OTH-2 | Incentive filter | Doctype != 03, 01 | Backend |
| J-PD-1 | Survey material max 5 | Material max 5; header submission fix | Backend + Frontend |
| J-PPM-1 | Gouging incentive Opcode 25 | Fix Gouging header submission | Backend + Frontend |
| J-PPM-2 | Sucker Pruning Cycle/Sqno | Add Cycle no., Seq no. to mobile; save to tblDARhdr | Backend + Frontend |
| J-PC-1 | Chemical Mixing OpCode | Fix tbl_MixingOprtn/costcenter mapping | Backend |
| J-PC-2 | Weedspray timekeeping | One accomp per report or time per accomp | Product decision |
| J-G-1 | CPMS material decimal | Enable decimal for CPMS material quantity | Frontend |
| J-G-2 | Fertilizer incentive | Add incentive Opcode 25; remove Operations | Frontend + Backend |
| J-G-3 | Gen Serv Group timekeeping | Costcenter/incentive per employee | Frontend + Backend |
| J-G-4 | PD Eradication | Incentive Opcode 25; accomplishments saved | Backend + Frontend |
| J-G-5 | Planting/Replanting | Add costcenter dropdown | Frontend |
| J-G-6 | Popcount | Remove incentive; HAS=layout actual area | Frontend + Backend |
| TS-SERVER | Server selection | Pre-login dropdown for base URL | Implemented (config.dart) |

### 5.2 Deviations: Documentation vs Implementation

| Area | Documentation | Implementation | Action |
|------|---------------|----------------|--------|
| getExistingBatchId | Swagger shows GET | Actual: POST with body | Update Swagger |
| MPS endpoint case | — | Config uses `/MPS/`; middleware `/mps/` | Use lowercase in Config |
| ERAD dtls | — | Uses genserv3/tblDARdtls | Documented as intentional |
| ERAD materials | — | Uses survey/tblDARmaterials | Documented as intentional |

---

## 6. Source of Truth Summary

| Area | Source |
|------|--------|
| Batch/Header/Details/Accomp/Materials schema | DAR_Middleware route files; controller/schema/getDARSchema.js |
| Validation rules | validateOperation.js; tbl_OperationConfig (seeds) |
| Sync/dropdown filters | dar-sync-queries-baseline.md; jay-feedback-intake.md |
| Flutter form models | main_dar_app lib/app/data/hive/forms/, lib/app/feature/ |
| API contracts | Swagger (/api-docs); route handlers |

---

## 7. References

- [dar-system-context.md](../../project/dar-system-context.md)
- [dar-user-journey-and-form-specifications.md](../../project/dar-user-journey-and-form-specifications.md)
- [dar-sync-queries-baseline.md](../../project/dar-sync-queries-baseline.md)
- [dar-backend-squad-review.md](../../project/dar-backend-squad-review.md)
- [dar-frontend-squad-review.md](../../project/dar-frontend-squad-review.md)
- [jay-feedback-intake.md](../../project/jay-feedback-intake.md)
- [product-master-foundations](../../skills/product-master-foundations/SKILL.md)
