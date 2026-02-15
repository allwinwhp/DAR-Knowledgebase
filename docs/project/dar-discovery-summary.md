# DAR System — Discovery Summary (Design Sprint Style)

This document is the **discovery output** for the DAR system using the **design-sprint-facilitator** flow: **DISCOVER → DESIGN → DECIDE**. It reflects the current state of **DAR Middleware** and the intended role of **main_dar_app** (frontend), both on branch **MDAG-339**.

---

## DISCOVER

### Business problem

- Farm operations need **daily accomplishment tracking** by supervisors.
- Work must be recorded **per operation type** (fruit care, bagging, gouging, harvest, fertilization, chemical mixing, survey, utility, etc.) with batch, header, detail, accomplishment, and material data.
- Data must sync with reference masters (locations, parcels, materials, layouts, employees, operation config) and remain consistent for reporting and operations.

### Target users and personas

| Persona | Description |
|---------|-------------|
| **Farm supervisor** | Primary user; records daily accomplishments in the field via the input interface (main_dar_app). |
| **Operations / back office** | Consumes sync APIs and reports; ensures reference data and DAR data integrity. |
| **System** | Integrations and API consumers (DAR Middleware). |

### Constraints and context

- **Database**: MSSQL, database `DAR`; connection via Knex (see `db/knex_config.js`).
- **Branch**: `MDAG-339` for both DAR Middleware and main_dar_app; not yet merged to DEV.
- **Existing implementation**: DAR Middleware is implemented with per-operation modules (mps, harvest, fertilization, utility, utility2, survey, erad-ppm, chem-mixer, bag-week, ph, pseudostem-cgms, sigatoka, popcount, genserv, genserv3); sync and controller APIs are in place; user login/register and accomplishment auto-calculation are done (see IMPLEMENTATION_STATUS.md in DAR_Middleware).

---

## DESIGN

### Core user journeys

1. **Supervisor logs daily work**  
   Open main_dar_app → sync reference data (operation config, locations, parcels, etc.) → create/select batch → enter header and details → record accomplishments (and materials where applicable) → submit to DAR Middleware. Totals (e.g. Qty1, batch accomp) are calculated on the backend.

2. **Sync reference data**  
   main_dar_app calls `/api/*` endpoints (users, operation-config, mixing-operations, location, parcel, material, variety, fruit-care, fertilizer, layout, employee-list, etc.) to keep local/reference data current.

3. **Submit DAR forms**  
   main_dar_app POSTs to `/sendDARForm/<operation>/tblDARhdr`, `tblDARdtls`, `tblDARaccomp`, `tblDARmaterials` as appropriate; utilities handle batch number, document number, and totals (calculateTotalsHdr, calculateTotalsBatch, autoCalculate).

### System boundaries (DAR Middleware)

| Component | Responsibility |
|-----------|----------------|
| **sendDARForm routes** | CRUD for batch, hdr, dtls, accomp, materials per operation type. |
| **Utilities** | latestBatchNo, latestDocumentNo, getExistingBatchId, calculateTotalsHdr, calculateTotalsBatch, calculateWeekBagged(2), autoCalculate. |
| **user routes** | Login, register, validate submission, assign user ID; integration with tblusers/tblaccounts. |
| **controller / api** | Data sync (reference data), hectarage, parcel, activity, erad, schema (DAR tables). |
| **report routes** | Reporting. |

### Integration points

| System | Purpose | Protocol |
|--------|---------|----------|
| **main_dar_app** | Input UI; form submission and sync | REST (JSON) to DAR Middleware |
| **MSSQL (DAR)** | Persistence for DAR and reference data | Knex / mssql driver |

---

## DECIDE

### MVP scope (already in place in DAR Middleware)

- DAR form submission per operation type (hdr, dtls, accomp, materials).
- Batch and document number handling; totals and auto-calculation.
- User registration and login; link to tblusers/tblaccounts.
- Sync endpoints for reference data (users, operation-config, mixing-operations, location, parcel, material, variety, fruit-care, fertilizer, etc.).
- Schema endpoint for DAR tables.

### Repos and branch

- **DAR_Middleware**: backend API; branch **MDAG-339**.
- **main_dar_app**: frontend input interface; branch **MDAG-339** (discover this repo separately if needed).

### Backlog / next steps (suggested)

- Merge **MDAG-339** to **DEV** after QA and UAT.
- Align team and agents on DAR context using [dar-system-context.md](dar-system-context.md).
- Run design sprints and sprint execution with **DAR product context** and **DAR repos**; use delivery-orchestrator with DAR project context when executing sprints for DAR.

---

## Traceability

| Design-sprint phase | Output |
|---------------------|--------|
| DISCOVER | Business problem, personas, constraints (this section). |
| DESIGN | User journeys, system boundaries, integration points (DAR Middleware + main_dar_app). |
| DECIDE | MVP scope, repos, branch, alignment and next steps. |

**Reference**: [dar-system-context.md](dar-system-context.md) — use for all DAR-related design and delivery alignment.
