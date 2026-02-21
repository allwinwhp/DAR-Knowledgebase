# Sprint-Jay-Feedback — Test Matrix

**Owner**: QA Lead. Test cases must be defined **before** dev implementation (TDD).

---

## Traceability

| Story | Test case IDs | Scope |
|-------|----------------|--------|
| US-J1 | TC-J1-1, TC-J1-2 | Costcenter filter tblActivity; incentive filter Doctype |
| US-J2 | TC-J2-1, TC-J2-2 | PD Survey material max 5; header sends |
| US-J3 | TC-J3-1, TC-J3-2 | Gouging incentive; header sends |
| US-J4 | TC-J4-1, TC-J4-2, TC-J4-3 | Cycle/Sqno in app and hdr; incentive RP13 |
| US-J5 | TC-J5-1 | Chemical Mixing operation code matches |
| US-J6 | TC-J6-1 | Weedspray one accomplishment or time per accomp |
| US-J7 | TC-J7-1 | CPMS material quantity decimal |
| US-J8 | TC-J8-1, TC-J8-2 | Fertilizer incentive; Operations removed |
| US-J9 | TC-J9-1, TC-J9-2 | Gen Services Group costcenter/incentive per row |
| US-J10 | TC-J10-1, TC-J10-2 | ERAD incentive; accomplishments saved |
| US-J11 | TC-J11-1 | Planting/Replanting header costcenter |
| US-J12 | TC-J12-1, TC-J12-2 | Popcount HAS from layout; no incentive |
| TS-SERVER | TC-SRV-1, TC-SRV-2, TC-SRV-3 | Server dropdown; base URL set; used for login and rest of session |

---

## Test cases (full list)

| TC-id | Scenario | Expected |
|-------|----------|----------|
| TC-J1-1 | GET /api/activity (costcenter list) | All rows from tblActivity WHERE Active=1 |
| TC-J1-2 | GET /api/incentive (no opcode/icode) | Rows from tblincentives WHERE Doctype NOT IN ('03','01') |
| TC-J2-1 | PD Survey: add up to 5 materials, submit | Header and full flow succeed; data on server |
| TC-J2-2 | PD Survey: submit with valid header payload | No header submission failure |
| TC-J3-1 | Gouging: select incentive (Opcode 25), submit | Header and full flow succeed |
| TC-J3-2 | Gouging: submit header after batch | Header creation succeeds |
| TC-J4-1 | Sucker Pruning: enter Cycle no., Seq no., submit | Values saved in tblDarHdr.Cycle, tblDarHdr.Sqno |
| TC-J4-2 | Sucker Pruning: incentive dropdown | Only Icode 'RP13' shown |
| TC-J4-3 | Sucker Pruning: full submit | Batch, hdr, dtls/accomp use correct incentive |
| TC-J5-1 | Chemical Mixing: select operation, submit | Saved operation code matches mobile selection |
| TC-J6-1 | Weedspray: one accomplishment per report OR time per accomplishment | Per product decision; data consistent |
| TC-J7-1 | CPMS: enter decimal material quantity | Accepts decimal; saves correctly |
| TC-J8-1 | Fertilizer: incentive dropdown (Opcode 25) | Shows correct incentives; submit succeeds |
| TC-J8-2 | Fertilizer: Operations field | Removed from UI |
| TC-J9-1 | General Services Group: timekeeping rows | Costcenter and incentive selectable per employee |
| TC-J9-2 | General Services Group: filters | Same tblActivity(Active=1) and incentive (Doctype excl.) |
| TC-J10-1 | PD Eradication: incentive filter | Opcode '25' only |
| TC-J10-2 | PD Eradication: submit accomplishments | Accomplishments saved to server |
| TC-J11-1 | Planting/Replanting: header costcenter | Two options selectable; saved in header |
| TC-J12-1 | Popcount: HAS | HAS = layout actual area (ActArea) when sending |
| TC-J12-2 | Popcount: incentive field | Removed from UI |
| TC-SRV-1 | Select server from dropdown, then login | Login request uses selected base URL |
| TC-SRV-2 | After login, sync or send report | Same base URL used for all API calls |
| TC-SRV-3 | Change server (if supported) or restart | Session uses one base URL until change/restart |

---

## Test Guide (Manual Review Phase)

Use this section for step-by-step validation during the manual review phase.

### Test scenarios (summary)

1. **Filters (US-J1)**  
   - Sync costcenter list: verify source is tblActivity Active=1.  
   - Sync incentive list (no opcode): verify no Doctype '03' or '01'.  
   - Sync incentive with opcode=25: verify only Opcode 25 and Doctype excluded.  
   - Sync incentive with icode=RP13: verify only Icode RP13 and Doctype excluded.

2. **Survey (US-J2)**  
   - Create PD Survey with up to 5 materials; submit batch then header.  
   - Expected: header submission succeeds; full flow sends to server.

3. **Gouging (US-J3)**  
   - Select incentive (S800 / Opcode 25); submit.  
   - Expected: header and full flow succeed.

4. **Sucker Pruning (US-J4)**  
   - Enter Cycle no. and Seq no.; select incentive RP13; submit.  
   - Expected: Cycle and Sqno in tblDarHdr; incentive correct.

5. **Chemical Mixing (US-J5)**  
   - Select operation; submit.  
   - Expected: operation code in DB matches selection.

6. **Weedspray (US-J6)**  
   - Per product decision: one accomplishment per report or time per accomplishment.  
   - Expected: data matches chosen rule.

7. **CPMS (US-J7)**  
   - Enter decimal in material quantity.  
   - Expected: accepts and saves decimal.

8. **Fertilizer (US-J8)**  
   - Verify incentive (Opcode 25) present and Operations field removed; submit.  
   - Expected: incentive saved; no Operations field.

9. **General Services Group (US-J9)**  
   - Add timekeeping rows; set costcenter and incentive per employee.  
   - Expected: filters same as Individual; data per row.

10. **PD Eradication (US-J10)**  
    - Select incentive; add accomplishments; submit.  
    - Expected: incentive filter; accomplishments saved.

11. **Planting/Replanting (US-J11)**  
    - Select header costcenter (two options).  
    - Expected: saved in header.

12. **Popcount (US-J12)**  
    - Select layout; submit.  
    - Expected: HAS = layout actual area; no incentive field.

13. **Server selection (TS-SERVER)**  
    - Select server from dropdown → login → sync or send report.  
    - Expected: same base URL used throughout session.

### Edge cases

- Survey: 0 materials vs 5 materials; header submit immediately after batch.  
- Incentive API: no query params vs opcode=25 vs icode=RP13.  
- Server selection: invalid URL handling; default server on first launch.  
- Decimal CPMS: very small/large decimals; locale.

### Regression checklist

- Re-run relevant scenarios from [dar-uat-checklist.md](../../../development/dar-uat-checklist.md) for: Survey, Gouging, Sucker Pruning, Chemical Mixing, Weedspray, CPMS, Fertilizer, Gen Services Group, Planting/Replanting, Popcount, ERAD.  
- Fruit care and other operations not in Jay scope: smoke test only.

---

## Regression

- Re-run relevant scenarios from [dar-uat-checklist.md](../../development/dar-uat-checklist.md) for affected operations (Survey, Gouging, Sucker Pruning, Chemical Mixing, Weedspray, CPMS, Fertilizer, Gen Services Group, Planting/Replanting, Popcount, ERAD).
