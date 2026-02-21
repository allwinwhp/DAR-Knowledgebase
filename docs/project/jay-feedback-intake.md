# Jay Feedback Intake — DAR Findings II (Feb 21, 2026)

**Purpose**: Design-sprint-style intake of [Jay's feedback](jay%20feedback/feedback.md). DISCOVER → DESIGN → DECIDE. Use as input for sprint planning and backlog. Aligned with [dar-system-context.md](dar-system-context.md) and [dar-mobile-feedback-baseline.md](dar-mobile-feedback-baseline.md).

**Source**: DAR FINDINGS II — 02212026

---

## DISCOVER — Summary of feedback

### Business problem (BA)

- **Fruit care**: All operations sending to server; no blocking issues.
- **Individual – Others / PDCM / PPM / Plant Care**: Filter and submission issues (costcenter, incentive, header failures, missing fields).
- **Gang/Group**: Filter issues, timekeeping structure (costcenter/incentive per employee), decimal support, missing HAS, header costcenter selection.

### Target users and constraints

| Persona | Impact |
|---------|--------|
| Farm supervisor | Correct filters, one-accomplishment-per-report for Weedspray, decimal for CPMS, HAS from layout. |
| Operations | Data integrity: Survey/Gouging/ERAD submission fixes; Cycle/Sqno; operation code mapping. |

### High-level risks

- Survey and Gouging **do not send** (batch saved, header fails).
- ERAD accomplishments **not saved** on server.
- Chemical mixing **operation code** mismatch (tbl_MixingOprtn / costcenter mapping).

---

## DESIGN — Findings mapped to backlog

### A. Individual

| ID | Area | Finding | Proposed change | Owner |
|----|------|---------|-----------------|--------|
| J-OTH-1 | General Services | Costcenter filter limited to 4 (chemical mixers). | Use `SELECT Cstctr, Cstctrdesc FROM tblActivity WHERE Active = 1` for all operations. | Backend |
| J-OTH-2 | General Services | Incentive filter. | Use `SELECT * FROM tblincentives WHERE Doctype != '03' AND Doctype != '01'`. | Backend |
| J-PD-1 | PD Survey | Material max 1; should be 5. Batch saved, header fails — can't send. | Survey: material max = 5; fix header submission (investigate PD Survey header endpoint). | Backend + Frontend |
| J-PPM-1 | Gouging | Incentive: show S800 only (chemical handler). Can't send — batch saved, header fails. | Incentive: `SELECT * FROM tblincentives WHERE Opcode = '25'`; fix Gouging header submission. | Backend + Frontend |
| J-PPM-2 | Sucker Pruning | Incentive: RP13 only. Cycle no. and Seq no. missing in app. | Incentive: `SELECT * FROM tblincentives WHERE Icode = 'RP13'`; add Cycle no. and Seq no. to mobile; save to tblDarHdr.Cycle, tblDarHdr.Sqno. | Backend + Frontend |
| J-PC-1 | Chemical Mixing | Operation code sent differs from mobile; tbl_MixingOprtn / costcenter mapping. | Fix operation code mapping (backend and/or lookup) so saved OpCode matches selection. | Backend |
| J-PC-2 | Weedspray | DAR has one time per accomplishment; mobile sends one time per report. | Option: one accomplishment per report for Weedspray, or send time per accomplishment (design decision). | Backend + Frontend |

### B. Gang/Group

| ID | Area | Finding | Proposed change | Owner |
|----|------|---------|-----------------|--------|
| J-G-1 | CPMS | Material quantity no decimal. | Enable decimal place for CPMS material quantity. | Frontend |
| J-G-2 | Fertilizer | No incentive in app; remove Operations field. | Add incentive (query: Opcode = '25' for S800); remove Operations field from UI. | Frontend + Backend |
| J-G-3 | General Services Group | Costcenter/incentive filter same as Individual; costcenter and incentive per employee in timekeeping. | Use tblActivity (Active=1) and tblincentives (Doctype != '03','01'); move costcenter and incentive code to **timekeeping** (per employee). | Frontend + Backend |
| J-G-4 | PD Eradication | Incentive filter; accomplishments not sent to server. | Incentive: Opcode = '25'; fix ERAD accomplishment submission (backend/contract). | Backend + Frontend |
| J-G-5 | Planting/Replanting | Two costcenters; add selection in header. | Add costcenter/operation dropdown in header: 22190-LP-Planting and Replanting, 14090–Planting and Replanting (hardcode or config). | Frontend |
| J-G-6 | Popcount | Remove incentive field; HAS has no value — should be layout actual area. | Remove incentive from Popcount; set HAS = actual area of selected layout when sending. | Frontend + Backend |

---

## DECIDE — User stories and technical stories

### Epic: Jay Feedback II — Filters, submission, and data integrity

| Story ID | Title | Acceptance criteria | Priority |
|----------|--------|---------------------|----------|
| US-J1 | General Services costcenter/incentive filter | Costcenter from tblActivity Active=1; incentive Doctype != '03','01'. Available to all applicable operations. | P1 |
| US-J2 | PD Survey send to server | Material max 5; header submission succeeds; full flow sends to server. | P1 |
| US-J3 | Gouging send to server | Incentive filter Opcode '25'; header submission succeeds. | P1 |
| US-J4 | Sucker Pruning Cycle/Sqno and incentive | Cycle no. and Seq no. in mobile and saved to tblDarHdr; incentive Icode 'RP13'. | P1 |
| US-J5 | Chemical Mixing operation code | Operation code saved matches mobile selection; fix tbl_MixingOprtn/costcenter mapping. | P1 |
| US-J6 | Weedspray timekeeping | One accomplishment per report, or time per accomplishment (product decision). | P2 |
| US-J7 | CPMS decimal | Material quantity accepts decimal. | P1 |
| US-J8 | Fertilizer incentive and remove Operations | Incentive (Opcode '25'); remove Operations field. | P1 |
| US-J9 | General Services Group timekeeping | Costcenter and incentive in timekeeping per employee; filters as above. | P1 |
| US-J10 | PD Eradication incentive and accomplishments | Incentive filter; ERAD accomplishments saved to server. | P1 |
| US-J11 | Planting/Replanting header costcenter | Costcenter selection in header (two options). | P1 |
| US-J12 | Popcount HAS and incentive | HAS = layout actual area; remove incentive field. | P1 |

### Technical story (new feature)

| Story ID | Title | Acceptance criteria | Priority |
|----------|--------|---------------------|----------|
| TS-SERVER | Server selection before login | On pre-login screen, user selects server from dropdown; value set as app base URL and used for entire session. | P1 |

---

## Traceability

| Design phase | Output |
|--------------|--------|
| DISCOVER | Business problem, users, risks (above). |
| DESIGN | Findings table (A, B) and proposed changes. |
| DECIDE | US-J1–US-J12, TS-SERVER; priority. |

**Next**: Sprint plan carves these into tasks; see [sprints/Sprint-Jay-Feedback/sprint-plan.md](sprints/Sprint-Jay-Feedback/sprint-plan.md).
