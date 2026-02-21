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

## Test cases (QA Lead to expand)

| TC-id | Scenario | Expected |
|-------|----------|----------|
| TC-J1-1 | Request costcenter list for General Services | From tblActivity Active=1 |
| TC-J1-2 | Request incentive list (excl. Harvest/Fruit care) | Doctype != '03','01' |
| TC-J2-1 | PD Survey: 5 materials, submit | Header and full flow succeed |
| TC-J3-1 | Gouging: incentive S800, submit | Header and full flow succeed |
| TC-J4-1 | Sucker Pruning: enter Cycle, Sqno, submit | Saved in tblDarHdr |
| TC-SRV-1 | Select server from dropdown, then login | Login uses selected base URL |
| TC-SRV-2 | After login, sync or send report | Same base URL used |
| *(expand)* | | |

---

## Regression

- Re-run relevant scenarios from [dar-uat-checklist.md](../../../development/dar-uat-checklist.md) for affected operations (Survey, Gouging, Sucker Pruning, Chemical Mixing, Weedspray, CPMS, Fertilizer, Gen Services Group, Planting/Replanting, Popcount, ERAD).
