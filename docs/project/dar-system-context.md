# DAR System — Project Context & Team Alignment

**Purpose**: Single source of truth for the DAR (Daily Accomplishment Report) system. Use this when running design sprints, delivery orchestration, or onboarding. **Override** any event-marketplace / Stagepass context with this DAR context.

**Branch**: `MDAG-339` (not yet merged to `DEV`). All current work for DAR Middleware and main_dar_app is on this branch.

**Stack**: Backend = **Node.js** (DAR_Middleware). Frontend = **Flutter** (main_dar_app).

**Report types**: DAR supports two types of reports — **gang/group** (crew-based) and **single/solo** (individual) reports.

---

## 1. What Is DAR?

**DAR** is an **input interface for farm supervisors** to track work done within the day. Reports go deep into farm operations and can be **gang/group** or **single/solo**. Operations include:

| Domain | Examples |
|--------|----------|
| **Fruit care** | Fruit care sync, variety, materials |
| **Bagging** | Bag-week (tbl_BagWeek, tbl_BagWeek2), week-bagged calculations |
| **Gouging** | Erad-ppm, pseudostem-cgms, erad layout, remaining cases |
| **Harvest** | Harvest headers, details, accomplishments, materials |
| **Fertilization** | Fertilization, fertilizer types/materials/cost center |
| **Chemical mixing** | Chem-mixer, mixing operations, chemical join |
| **Survey** | Survey parcels, survey DAR (hdr/dtls/accomp/materials) |
| **Other operations** | Utility, utility2, genserv, genserv3, popcount, sigatoka, MPS, PH |

**Personas** (override design-sprint default):

- **Farm supervisor** — Primary user: logs daily accomplishments by operation type, batch, and location.
- **Operations / back office** — Sync reference data (locations, parcels, materials, layouts), run reports.
- **System / integration** — DAR Middleware is the API layer between the field app and the DAR database.

---

## 2. Repositories & Architecture

| Repo | Role | Branch |
|------|------|--------|
| **DAR_Middleware** | Node/Express API; MSSQL (Knex); DAR form submission, sync endpoints, user/auth, reports | `MDAG-339` |
| **main_dar_app** | Flutter frontend — input interface consumed by farm supervisors | `MDAG-339` |

- **Database**: MSSQL, database name `DAR` (see `db/knex_config.js`).
- **Core DAR tables**: `tblDARbatch`, `tblDARhdr`, `tblDARdtls`, `tblDARaccomp`, `tblDARmaterials` — repeated per operation type (mps, harvest, fertilization, utility, survey, etc.).
- **Sync/reference**: Employees, locations, parcels, layouts, hectarage, materials, variety, disease, farm, operation config, mixing operations, fruit-care, etc.

When the **delivery-orchestrator** or **design-sprint-facilitator** runs for DAR:

- **Frontend codebase** = **main_dar_app** (not stagepass-v0-styling).
- **Backend / API** = **DAR_Middleware**.
- **Branch strategy**: Work on `MDAG-339` until merged to `DEV`; then follow branch-per-sprint from `DEV`/`main` as per governance.

---

## 3. DAR Middleware — High-Level Capabilities

- **`/sendDARForm/*`** — Submit and manage DAR data: batch, hdr, dtls, accomp, materials per operation (mps, harvest, fertilization, utility, utility2, survey, erad-ppm, chem-mixer, bag-week, ph, pseudostem-cgms, sigatoka, popcount, genserv, genserv3).
- **`/user/*`** — Login, register, validate submission, assign user ID.
- **`/report/*`** — Reporting routes.
- **`/api/*`** — Controller: data sync (users, operation-config, mixing-operations, location, parcel, material, variety, fruit-care, fertilizer, chem-mixing, etc.), hectarage, parcel, activity, erad, schema (DAR tables).

Use this document and the codebase as the **source of truth** for DISCOVER/DESIGN/DECIDE when facilitating design sprints for DAR.

---

## 4. Alignment Checklist for the Team

- [ ] **Context**: We are working on **DAR** (farm supervisor daily accomplishment tracking), not the events marketplace.
- [ ] **Repos**: **DAR_Middleware** + **main_dar_app** on branch **MDAG-339**.
- [ ] **Personas**: Farm supervisor (primary), operations, system/integration.
- [ ] **Operations**: Fruit care, bagging, gouging, harvest, fertilization, chemical mixing, survey, utility, MPS, erad, sigatoka, etc.
- [ ] **Orchestrator / sprint execution**: When running for DAR, use DAR repos and this context; ignore Stagepass-specific decisions (e.g. stagepass-v0-styling, Validator App).
- [ ] **Design sprint**: Use **design-sprint-facilitator** with **DAR product context** (this doc and dar-discovery-summary.md).

---

## 5. References

- **Discovery summary** (DISCOVER → DESIGN → DECIDE): [dar-discovery-summary.md](dar-discovery-summary.md)
- **User journey & form specifications**: [dar-user-journey-and-form-specifications.md](dar-user-journey-and-form-specifications.md)
- **Backend squad review** (DAR Middleware): [dar-backend-squad-review.md](dar-backend-squad-review.md)
- **Frontend squad review** (main_dar_app Flutter): [dar-frontend-squad-review.md](dar-frontend-squad-review.md)
- **Deployment MOP** (pre-implementation, implementation, post-implementation): [../development/dar-deployment-mop.md](../development/dar-deployment-mop.md)
- **UAT Test Case Checklist** (client-side, supervisor persona): [../development/dar-uat-checklist.md](../development/dar-uat-checklist.md)
- **Feedback Baseline** (DAR mobile findings — team review): [dar-mobile-feedback-baseline.md](dar-mobile-feedback-baseline.md)
- **Jay Feedback Intake** (Feb 21, 2026 — design-sprint intake): [jay-feedback-intake.md](jay-feedback-intake.md)
- **Sprint: Jay Feedback** (sprint plan, scope, code deep-dive, server selection): [sprints/Sprint-Jay-Feedback/sprint-plan.md](sprints/Sprint-Jay-Feedback/sprint-plan.md)
- **Delivery orchestrator** (when running for DAR): [../../agents/delivery-orchestrator.md](../../agents/delivery-orchestrator.md) — see § “Project context: DAR”.
- **Design sprint facilitator**: [../../skills/design-sprint-facilitator/SKILL.md](../../skills/design-sprint-facilitator/SKILL.md) — Product context overridden by DAR when working on DAR.
