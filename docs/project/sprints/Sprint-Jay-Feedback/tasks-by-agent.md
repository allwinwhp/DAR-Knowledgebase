# Sprint-Jay-Feedback — Tasks by Agent

---

## Backend (DAR_Middleware)

| Task | Story | Description |
|------|--------|-------------|
| BE-1 | US-J1 | Costcenter API: use `tblActivity WHERE Active = 1`. Incentive API: `tblincentives WHERE Doctype != '03' AND Doctype != '01'` where applicable. |
| BE-2 | US-J2 | PD Survey: fix header submission (investigate failure); ensure material max 5 in validation. |
| BE-3 | US-J3 | Gouging: fix header submission; incentive filter Opcode '25'. |
| BE-4 | US-J4 | Sucker Pruning: accept and persist Cycle, Sqno in tblDarHdr. Incentive Icode 'RP13'. |
| BE-5 | US-J5 | Chemical Mixing: fix operation code mapping (tbl_MixingOprtn / costcenter). |
| BE-6 | US-J9 | General Services Group: support costcenter and incentive per timekeeping row. |
| BE-7 | US-J10 | PD Eradication: fix accomplishment submission; incentive Opcode '25'. |
| BE-8 | US-J12 | Popcount: HAS from layout actual area when sending; no incentive. |

---

## Frontend (main_dar_app)

| Task | Story | Description |
|------|--------|-------------|
| FE-1 | US-J2 | PD Survey: material max 5; ensure header payload correct. |
| FE-2 | US-J3 | Gouging: incentive dropdown (Opcode 25). |
| FE-3 | US-J4 | Sucker Pruning: add Cycle no., Seq no. fields; map to hdr. |
| FE-4 | US-J6 | Weedspray: implement one-accomplishment-per-report or per-accomplishment time (per decision). |
| FE-5 | US-J7 | CPMS: allow decimal for material quantity. |
| FE-6 | US-J8 | Fertilizer: add incentive (Opcode 25); remove Operations field. |
| FE-7 | US-J9 | General Services Group: costcenter and incentive per employee in timekeeping. |
| FE-8 | US-J11 | Planting/Replanting: costcenter/operation dropdown in header (two options). |
| FE-9 | US-J12 | Popcount: set HAS from selected layout ActArea; remove incentive field. |
| FE-10 | TS-SERVER | Server selection: pre-login screen with server dropdown; persist base URL for session. |

---

## QA Lead

| Task | Description |
|------|-------------|
| QA-1 | Test strategy and test cases for US-J1–US-J12 and TS-SERVER (before dev implementation). |
| QA-2 | test-matrix.md with TC-ids and traceability. |
| QA-3 | uat-checklist.md updates for Jay scenarios and server selection. |

---

## Architect / Dev-Senior

| Task | Description |
|------|-------------|
| A-1 | code-deep-dive.md: where base URL is set, where login/splash live, API client usage. |
| A-2 | Validate backend contract for new/updated fields (Cycle, Sqno, costcenter per row, HAS). |

---

## Design Authority

| Task | Description |
|------|-------------|
| D-1 | server-selection-feature.md: UI placement, dropdown list source, session persistence. |

---

## DevOps

| Task | Description |
|------|-------------|
| O-1 | cicd-impact.md; deployment-plan.md for sprint branch and UAT. |

---

## SM

| Task | Description |
|------|-------------|
| SM-1 | risk-log.md; carryover.md; Definition of Done check. |
