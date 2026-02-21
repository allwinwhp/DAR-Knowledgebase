# Team Review — GenServ Incentive Dropdown (Full List)

**Date**: 2026-02-22  
**Issue**: GenServ Incentive dropdown was not showing the full list as in [baseline-api-responses.json](baseline-api-responses.json) (5 items for `GET /api/incentive`).

---

## Root cause (Frontend)

In **main_dar_app** `lib/app/feature/others/gen_serv/view/form/widgets/gen_serv_main_details.dart`:

- The incentive list was **conditional on cost center selection**:
  - If **Cost Center** was selected, `mainDetails.opCode` was set (from the activity’s Oprtncde).
  - The dropdown then used `IncentiveRepository.getIncentivesByOpcode(mainDetails.opCode!)`, which filters to incentives with that Opcode only (e.g. Opcode 25 → 1 item: S800).
- So the **full list (5 items)** only appeared when **no cost center** was selected. After selecting a cost center, the dropdown showed 1–2 items instead of the baseline 5.

---

## Alignment with Jay feedback

- **US-J1**: General Services costcenter/incentive filter — costcenter from tblActivity Active=1; **incentive Doctype != '03','01'** (full list).
- **J-G-3**: General Services Group — “tblincentives (Doctype != '03','01')”; same as Individual.

So for **General Services** (Individual and Group), the incentive dropdown should show the **full Doctype-filtered list**, not filtered by the selected cost center’s opCode.

---

## Fix applied

- **File**: `main_dar_app/lib/app/feature/others/gen_serv/view/form/widgets/gen_serv_main_details.dart`
- **Change**: Incentive dropdown now **always** uses `IncentiveRepository.getIncentives()` (full list from local storage, which is synced via `GET /api/incentive` with no params → 5 items per baseline).
- **Removed**: The branch that used `getIncentivesByOpcode(mainDetails.opCode!)` when a cost center was selected.

---

## What the team should verify

1. **Sync**: User has run **Sync** at least once (e.g. from Home) so local incentives = `GET /api/incentive` response (5 items per baseline).
2. **GenServ form**: Open General Services → tap **Incentive** dropdown → confirm **5 options** (N/A, WEEDSPRAY/P14, Chemical Handler/S800, ENNG SKILLED WORKERS/S801, TOOLKEEPER/S804), matching [baseline-api-responses.json](baseline-api-responses.json).
3. **Regression**: Selecting a cost center and then opening the incentive dropdown still shows all 5 (no longer filtered by opCode).

---

## Owner

Frontend (per delivery-orchestrator); peer review by Dev-Senior / QA per governance.
