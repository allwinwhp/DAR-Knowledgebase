# Sprint-Jay-Feedback — Ready for Review

**Date**: 2026-02-22  
**Status**: Baseline complete; partial backend; remaining BE/FE tasks documented for follow-up.

---

## What was done this run

1. **Baseline for dropdown testing (requested)**
   - **baseline-api-responses.json** — Captured from **localhost:3000**: activity (317 rows), incentive (no params: 5), incentive?opcode=25 (1), incentive?icode=RP13 (0).
   - **baseline-expected-results.md** — Table of expected results per Jay feedback SQL for comparing dropdown contents and API responses during testing.

2. **Backend**
   - **BE-1**: Already in place (activity Active=1; incentive Doctype filter + opcode/icode).
   - **BE-4 (partial)**: pseudostem-cgms/tblDARhdr.js now accepts **Cycle** and **Sqno** in the request body and passes them to the insert. *(Ensure tblDARhdr has Cycle, Sqno columns or add migration.)*

3. **Sprint package**
   - Deliverables updated to include baseline artifacts.
   - execution-summary.md updated with this run.

---

## What to review

| Item | Where | Action |
|------|--------|--------|
| Baseline payloads | baseline-api-responses.json | Confirm counts/samples match your DB; re-run against localhost:3000 if needed. |
| Expected results table | baseline-expected-results.md | Use for UAT: compare app dropdowns and API responses to the table. |
| Cycle/Sqno backend | DAR_Middleware sendDARForm/pseudostem-cgms/tblDARhdr.js | Confirm Cycle, Sqno columns exist in tblDARhdr; add migration if not. |
| Remaining BE/FE | tasks-by-agent.md, execution-summary.md | BE-2, BE-3, BE-5–BE-8 and FE-1–FE-9 left for next iteration. |

---

## Remaining to-dos (for next sprint or follow-up)

- **BE-2** PD Survey: fix header submission; material max 5 validation.
- **BE-3** Gouging: fix header submission (incentive filter already in API).
- **BE-5** Chemical Mixing: operation code mapping.
- **BE-6** Gen Services Group: costcenter/incentive per timekeeping row.
- **BE-7** PD Eradication: accomplishment submission.
- **BE-8** Popcount: HAS from layout; no incentive.
- **FE-1–FE-9** Form changes (Survey, Gouging, Sucker Pruning Cycle/Sqno fields, Weedspray, CPMS decimal, Fertilizer, Gen Services Group, Planting/Replanting, Popcount).

---

## Quick verification

- Backend on localhost:3000: `GET /api/activity` → 317 rows; `GET /api/incentive` → 5 rows; `GET /api/incentive?opcode=25` → 1 row (S800).
- Use baseline-expected-results.md and uat-checklist.md for manual testing and sign-off.

**Sprint team**: Please review the baseline artifacts and Cycle/Sqno backend change; then proceed with UAT and remaining tasks as planned.
