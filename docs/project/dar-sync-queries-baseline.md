# DAR — Sync Queries Baseline (Single Source of Truth)

**Purpose**: One place for all backend sync queries. Use this to check DB vs requirements and to validate frontend dropdown/list content.  
**Repos**: DAR_Middleware (backend), main_dar_app (Flutter).  
**Aligned with**: [delivery-orchestrator](../../agents/delivery-orchestrator.md), [jay-feedback-intake](jay-feedback-intake.md), [Sprint-Jay-Feedback baseline](sprints/Sprint-Jay-Feedback/baseline-expected-results.md).

---

## How to use

1. **Backend**: Ensure each route’s query matches the SQL/equivalent in this doc; document any params.
2. **Frontend**: When building or fixing a dropdown/list, sync from the listed endpoint and compare count/keys to this baseline (or to a captured payload).
3. **QA / Team**: Run baseline capture (e.g. from localhost:3000), compare to this doc and to [baseline-api-responses.json](sprints/Sprint-Jay-Feedback/baseline-api-responses.json); use [baseline-expected-results.md](sprints/Sprint-Jay-Feedback/baseline-expected-results.md) for UAT.

---

## 1. Activity (Costcenter)

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/activity` |
| **Backend file** | DAR_Middleware `controller/activity/getActivity.js` |
| **SQL / logic** | `SELECT * FROM tblActivity WHERE Active = 1`; optional `?doctype=`, `?oprtncde=` |
| **Query params** | `doctype` (e.g. 10), `oprtncde` (e.g. 25) |
| **Frontend** | Costcenter dropdown (GenServ, Chemical Mixing, etc.); Config: `syncCostCenterEndpoint = '/api/activity'` |
| **Jay / requirements** | J-OTH-1, US-J1: costcenter from tblActivity Active=1 for all applicable operations. |

---

## 2. Incentive

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/incentive` |
| **Backend file** | DAR_Middleware `controller/data_sync/getIncentive.js` |
| **SQL / logic** | Base: `SELECT * FROM tblincentives WHERE Doctype NOT IN ('03','01')`. Optional: `AND Opcode = ?opcode` or `AND Icode = ?icode`. |
| **Query params** | `opcode` (e.g. 25, 30), `icode` (e.g. RP13) |
| **Frontend** | Incentive dropdown (GenServ, Gouging, Sucker Pruning, Fertilizer, ERAD, CPMS, Chemical Mixing, Weedspray, etc.); Config: `syncIncentiveEndpoint = '/api/incentive'` |
| **Jay / requirements** | J-OTH-2, US-J1, J-G-3: Doctype != 03, 01. Gouging/ERAD/Fertilizer/CPMS: Opcode 25. Sucker Pruning: Icode RP13. GenServ: full list. |

---

## 3. Location

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/location` |
| **Backend file** | `controller/data_sync/getLocation.js` |
| **SQL / logic** | `SELECT * FROM tblLocation` |
| **Query params** | — |
| **Frontend** | Farm/location dropdown; Config: `syncLocationEndpoint = '/api/location'` |

---

## 4. Disease

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/disease` |
| **Backend file** | `controller/data_sync/getDisease.js` |
| **SQL / logic** | `SELECT * FROM tblDisease` |
| **Frontend** | Disease type dropdown; Config: `syncDiseaseTypeEndpoint = '/api/disease'` |

---

## 5. Employees

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/employee-list` |
| **Backend file** | `controller/data_sync/getEmployeeList.js` |
| **SQL / logic** | `SELECT * FROM tblemployee` (or filtered per implementation) |
| **Frontend** | Employee list; Config: `syncEmployeesEndpoint = '/api/employee-list'` |

---

## 6. Holidays

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/holidays` |
| **Backend file** | `controller/data_sync/getHoliday.js` |
| **SQL / logic** | Per file (holiday table) |
| **Frontend** | Config: `syncHolidaysEndpoint = '/api/holidays'` |

---

## 7. Variety

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/variety` (and `/api/variety/varcode-varietyy` if used) |
| **Backend file** | `controller/data_sync/getVariety.js`, `controller/variety/getVariety.js` |
| **SQL / logic** | Variety table select |
| **Frontend** | Config: `syncVarietiesEndpoint = '/api/variety'` |

---

## 8. Materials (join by cost center)

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/material/join-materials-cstctr` |
| **Backend file** | `controller/material/joinMaterialCstctr.js` |
| **SQL / logic** | Join tbl_materials + tbl_materialsacvty; `WHERE Ma.DTag = 1` |
| **Frontend** | Material dropdown (by cost center); Config: `syncMaterialsEndpoint = '/api/material/join-materials-cstctr'` |

---

## 9. Chem mixing

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/chem-mixing` |
| **Backend file** | `controller/data_sync/getChemMixing.js` |
| **SQL / logic** | `SELECT * FROM tbl_ChemMixing` |
| **Frontend** | Config: `syncChemMixingsEndpoint = '/api/chem-mixing'` |

---

## 10. Mixing operations

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/mixing-operations` |
| **Backend file** | `controller/data_sync/getMixingOperations.js` |
| **SQL / logic** | `SELECT Opcode, Opdesc FROM tbl_MixingOprtn`; optional `?cstctr=` (LIKE pattern) |
| **Query params** | `cstctr` |
| **Frontend** | Chemical Mixing operation dropdown; Config: `syncMixingOperationsEndpoint = '/api/mixing-operations'` |
| **Jay / requirements** | US-J5: operation code mapping (tbl_MixingOprtn / costcenter). |

---

## 11. Fertilizer type

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/fertilizer-type` |
| **Backend file** | `controller/data_sync/getFertType.js` |
| **SQL / logic** | `SELECT * FROM tbl_FertilizerType` |
| **Frontend** | Config: `syncFertTypesEndpoint = '/api/fertilizer-type'` |

---

## 12. Fertilizer (join material)

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/fertilizer/join-material` |
| **Backend file** | `controller/fertilizer/joinMaterial.js` |
| **SQL / logic** | Join tbl_materials + tbl_materialsacvty (fertilizer context) |
| **Frontend** | Config: `syncFertilizersEndpoint = '/api/fertilizer/join-material'` |

---

## 13. Fertilizer cost center

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/fertilizer-cost-center` |
| **Backend file** | `controller/data_sync/getFertilizerCostCenter.js` |
| **SQL / logic** | `SELECT * FROM tblActivity WHERE Active = 1 AND Doctype = 10` |
| **Frontend** | Fertilizer cost center dropdown; Config: `syncFertilizerCostCenterEndpoint = '/api/fertilizer-cost-center'` |

---

## 14. Fertilizer materials

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/fertilizer-materials` |
| **Backend file** | `controller/data_sync/getFertilizerMaterials.js` |
| **SQL / logic** | Join tbl_materials + fertilizer type (FertType) |
| **Frontend** | Config: `syncFertilizerMaterialsEndpoint = '/api/fertilizer-materials'` |

---

## 15. Fruit care (areas / parcels / layouts)

| Item | Value |
|------|--------|
| **Endpoints** | `GET /api/fruit-care-sync/areas`, `/parcels`, `/layouts`; `/areas/all`, `/parcels/all`, `/layouts/all` |
| **Backend file** | `controller/data_sync/getFruitCareData.js` |
| **SQL / logic** | tbl_hectarageMPS (areas, parcels, layouts); filters by farm etc. |
| **Frontend** | Fruit care sync; Config: syncFruitCareAreasEndpoint, syncFruitCareParcelsEndpoint, etc. |

---

## 16. Other operations (parcels / layouts / survey-layouts)

| Item | Value |
|------|--------|
| **Endpoints** | `GET /api/other-operations-sync/parcels`, `/layouts`, `/parcels/all`, `/layouts/all`, `/survey-layouts`, `/survey-layouts/all` |
| **Backend file** | `controller/data_sync/getOtherOperationsData.js` |
| **SQL / logic** | Parcels: tblparcel, Parcel != 'S'. Layouts: tblhectarage, Active=1. Survey: Parcel = 'S'. Optional farmCode, parcel. |
| **Query params** | farmCode, parcel |
| **Frontend** | Weedspray, Fertilizer, CPMS, Survey, etc.; Config: syncOtherOperationsParcelsEndpoint, syncSurveyLayoutsEndpoint, etc. |

---

## 17. Operation config

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/operation-config` |
| **Backend file** | `controller/data_sync/getOperationConfig.js` |
| **SQL / logic** | `SELECT * FROM tbl_OperationConfig` |
| **Frontend** | Operation configuration. |

---

## 18. Users / accounts / batch-numbers / doc-numbers

| Item | Value |
|------|--------|
| **Endpoints** | `GET /api/users`, `/api/accounts`, `/api/batch-numbers`, `/api/doc-numbers` |
| **Backend files** | getUsers.js, syncAccounts.js, syncBatchNumbers.js, syncDocNumbers.js |
| **SQL / logic** | tblusers, tblaccounts, tblbatchno, tbldocno — each select * or equivalent |
| **Frontend** | User/account/batch/doc sync. |

---

## 19. Hectarage / parcel / ERAD layout (specialized)

| Item | Value |
|------|--------|
| **Endpoints** | `/api/hectarage-area`, `/hectarage-mps`, `/hectarage/erad-layout`, `/parcel/farmcode-parcel`, `/parcel/survey-parcel`, etc. |
| **Backend files** | getHectarageArea.js, getOtherOperationsData (survey-layouts), getEradLayout.js, getFarmcodeParcel.js, getSurveyParcel.js |
| **SQL / logic** | See Swagger or source; tblhectarage, tblparcel, tbl_hectarageMPS — filters by Parcel, FarmCode, etc. |
| **Frontend** | Parcel/layout dropdowns for Survey, ERAD, Other Ops. |

---

## 20. ERAD remaining cases

| Item | Value |
|------|--------|
| **Endpoint** | `GET /api/erad/remaining-cases` |
| **Backend file** | `controller/erad/getRemainingCases.js` |
| **SQL / logic** | tblDARaccomp — SurveyID, NoCSurvey, NoCErad |
| **Frontend** | ERAD flow. |

---

## Frontend changes vs baseline (options per form)

Per [baseline-expected-results.md](sprints/Sprint-Jay-Feedback/baseline-expected-results.md) and [jay-feedback-intake.md](jay-feedback-intake.md), the following frontend behavior is required. **Delivery-orchestrator** context: frontend = **main_dar_app**.

| Form | Dropdown | Required API / behavior | Frontend change |
|------|----------|-------------------------|-----------------|
| Costcenter (all) | Costcenter | `GET /api/activity` | Already uses activity sync. |
| General Services | Incentive | `GET /api/incentive` (full list) | **Done**: GenServ uses `getIncentives()`. |
| Gouging | Incentive | `GET /api/incentive?opcode=25` (1 row: S800) | **Change**: Use `getIncentivesByOpcode('25')` (was `'09'`). |
| Sucker Pruning | Incentive | `GET /api/incentive?icode=RP13` | Already uses `getIncentivesByIcode('RP13')`. |
| Fertilizer | Incentive | `GET /api/incentive?opcode=25` | **Add**: Incentive dropdown with opcode 25 (J-G-2, US-J8). |
| PD Eradication | Incentive | `GET /api/incentive?opcode=25` | **Change**: Use `getIncentivesByOpcode('25')` (was `'09'`). |
| CPMS | Incentive | `GET /api/incentive?opcode=25` | Already uses `'25'`. |
| Chemical Mixing | Incentive | `GET /api/incentive?opcode=25` | Already uses `'25'`. |
| Weedspray | Incentive | `GET /api/incentive?opcode=30` | Already uses `'30'`. |
| Popcount | Incentive | **Remove field** (Jay J-G-6, US-J12) | **Remove**: Remove incentive dropdown from Popcount form; send no incentive. |
| Planting/Replanting | Incentive | Confirm with PO (baseline does not specify) | Currently uses opcode `'09'`; leave as-is unless PO says otherwise. |

**Summary of required changes**: (1) Gouging → opcode 25. (2) ERAD → opcode 25. (3) Popcount → remove incentive field. (4) Fertilizer → add incentive dropdown (opcode 25).

---

## Maintenance

- **Backend (backend-squad / dev-mid)**: When adding or changing a sync route, add/update the row here and the SQL.
- **QA / Juniors**: Capture payloads (e.g. from localhost:3000) into baseline JSON; compare to this doc and to frontend.
- **Frontend (frontend-squad / dev-mid)**: When adding a dropdown that uses sync data, reference this doc and the baseline expected results.

**Full course / team scaling**: See [dar-sync-baseline-team-assignment.md](dar-sync-baseline-team-assignment.md).
