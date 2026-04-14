# Sync Caching — Test Matrix

**Feature**: API-level caching (backend) + parallel sync & batch Hive writes (frontend)
**Repos**: DAR_Middleware (backend), main_dar_app (frontend) — Branch `MDAG-339`
**Author**: QA Lead
**Date**: 2026-04-15

---

## 1. Test Strategy

### Objective

Verify that:
1. **Backend**: In-memory caching serves correct, fresh data for static/semi-static endpoints; volatile endpoints bypass or use very short TTL; error responses are never cached; `X-Cache` headers are present.
2. **Frontend**: Parallelized sync groups complete without race conditions; batch Hive writes maintain data integrity; overall sync time improves measurably.

### Endpoint Categories

| Category | Count | TTL | Cache? |
|----------|-------|-----|--------|
| Static master data | 14 | 6 h | Yes |
| Semi-static | 4 | 15 m | Yes |
| Bulk | 6 | 15 m | Yes |
| Volatile | 4 | 60 s | Short TTL |
| Queue | 4 | — | Never |

### Test Levels

| Level | Scope | Owner |
|-------|-------|-------|
| Unit | Middleware logic, TTL config | Dev |
| Integration | Cache HIT/MISS per endpoint, header verification | QA Mid + Dev |
| E2E | Full sync flow, benchmarks | QA Senior |
| UAT | Supervisor persona experience | PO / PM |

### Entry / Exit Criteria

| Gate | Criteria |
|------|----------|
| Entry | Cache middleware merged; parallel sync merged; all endpoints reachable |
| Exit — QA | All Critical/High test cases pass; no P1 defects; ≥30% sync improvement |
| Exit — UAT | Stakeholder sign-off; all UAT scenarios executed |

---

## 2. Test Cases

### 2.1 Backend (25 test cases)

| TC-ID | Scenario | Expected | Priority | Traces To |
|-------|----------|----------|----------|-----------|
| TC-SC01 | First GET to static endpoint → MISS | 200; X-Cache: MISS; body matches DB | Critical | TS-SC01 |
| TC-SC02 | Second GET (within TTL) → HIT | 200; X-Cache: HIT; identical body | Critical | TS-SC01 |
| TC-SC03 | Cache expires after TTL | X-Cache: MISS on next request | Critical | TS-SC02 |
| TC-SC04 | Different query params → separate keys | Each returns correct filtered data | Critical | TS-SC03 |
| TC-SC05 | Same params different order → same key | Second request returns HIT | High | TS-SC03 |
| TC-SC06 | Cached response identical JSON schema | No extra fields or wrapper | Critical | TS-SC03 |
| TC-SC07 | X-Cache on all 14 static endpoints | MISS then HIT on all | High | TS-SC03 |
| TC-SC08 | X-Cache on all 4 semi-static endpoints | MISS then HIT on all | High | TS-SC03 |
| TC-SC09 | X-Cache on all 6 bulk endpoints | MISS then HIT on all | High | TS-SC03 |
| TC-SC10 | Volatile endpoints bypass/short TTL | HIT only within ≤60 s | Critical | TS-SC02 |
| TC-SC11 | Batch numbers fresh after DB change | New data visible after TTL | Critical | TS-SC02 |
| TC-SC12 | Queue endpoints never cached | No X-Cache header | High | TS-SC03 |
| TC-SC13 | 500 error not cached | Fresh data on recovery | Critical | TS-SC01 |
| TC-SC14 | 404/empty not cached | New data visible after insert | High | TS-SC01 |
| TC-SC15 | DB timeout not cached | Fresh data on recovery | High | TS-SC01 |
| TC-SC16 | Static TTL values in config (6–24 h) | Config matches spec | High | TS-SC02 |
| TC-SC17 | Semi-static TTL values (5–30 m) | Config matches spec | High | TS-SC02 |
| TC-SC18 | Bulk TTL values (10–30 m) | Config matches spec | Medium | TS-SC02 |
| TC-SC19 | maxKeys respected | Eviction on overflow | Medium | TS-SC01 |
| TC-SC20 | checkperiod sweeps expired entries | Memory freed | Medium | TS-SC01 |
| TC-SC21 | Cache flush clears all entries | Subsequent requests all MISS | Medium | TS-SC04 |
| TC-SC22 | 5 concurrent same-key requests | At most 1 MISS; no corruption | High | TS-SC01 |
| TC-SC23 | Content-Type preserved on HIT | application/json | Medium | TS-SC03 |
| TC-SC24 | Status code 200 on HIT | Not 304 or 204 | Medium | TS-SC03 |
| TC-SC25 | Large payload (>5000 rows) cached | No truncation; bodies identical | High | TS-SC01 |

### 2.2 Frontend (15 test cases)

| TC-ID | Scenario | Expected | Priority | Traces To |
|-------|----------|----------|----------|-----------|
| TC-SC30 | All sync groups complete | Success message; last sync updated | Critical | US-SC01 |
| TC-SC31 | One group failure doesn't block others | Partial success reported | Critical | TS-SC05 |
| TC-SC32 | No Hive race condition in parallel | Each box correct; no exceptions | Critical | TS-SC05 |
| TC-SC33 | addAll writes complete dataset | Box count matches API response | Critical | TS-SC06 |
| TC-SC34 | addAll with empty response | No crash; box cleared | High | TS-SC06 |
| TC-SC35 | addAll with large dataset (>5000) | All records written; no ANR | High | TS-SC06 |
| TC-SC36 | Progress dialog shows group progress | Groups advance; checkmarks shown | Medium | TS-SC07 |
| TC-SC37 | Navigate away mid-sync | No crash; graceful handling | Medium | US-SC02 |
| TC-SC38 | Model type safety after addAll | All fields populated correctly | High | TS-SC06 |
| TC-SC39 | Sync time ≥30% faster (HIT) vs baseline | Logged timing comparison | High | US-SC01 |
| TC-SC40 | Sync time ≥20% faster (MISS) vs baseline | Parallel improvement measured | High | US-SC02 |
| TC-SC41 | Second sync (warm cache) near-instant | ≤50% of first sync time | High | US-SC01 |
| TC-SC42 | Volatile data fresh on each sync | Batch/doc numbers current | Critical | US-SC04 |
| TC-SC43 | Full offline → graceful error | Error message; Hive data retained | High | US-SC02 |
| TC-SC44 | Partial network failure | Succeeded groups persist; errors reported | Medium | US-SC02 |

### 2.3 E2E (10 test cases)

| TC-ID | Scenario | Expected | Priority | Traces To |
|-------|----------|----------|----------|-----------|
| TC-SC50 | Full sync flow end-to-end | All endpoints called; Hive populated; UI correct | Critical | US-SC01 |
| TC-SC51 | Second sync uses cache | X-Cache: HIT; faster completion | Critical | US-SC01 |
| TC-SC52 | Form dropdowns correct after cached sync | All data matches baseline | Critical | US-SC03 |
| TC-SC53 | DB change visible after TTL expiry | New data appears on re-sync | Critical | US-SC04 |
| TC-SC54 | Benchmark: before vs after timing | Documented improvement | High | US-SC02 |
| TC-SC55 | Multi-supervisor shared cache | Second device gets HITs | High | TS-SC01 |
| TC-SC56 | Server restart clears cache | Next sync all MISS | Medium | TS-SC01 |
| TC-SC57 | Query param isolation (incentive opcodes) | No cross-contamination | Critical | TS-SC03 |
| TC-SC58 | Fruit care multi-endpoint sync | All 3 boxes populated correctly | High | US-SC03 |
| TC-SC59 | Other operations multi-endpoint sync | All 3 boxes populated correctly | High | US-SC03 |

---

## 3. Coverage Summary

| Priority | Backend | Frontend | E2E | Total |
|----------|---------|----------|-----|-------|
| Critical | 9 | 6 | 5 | 20 |
| High | 10 | 6 | 4 | 20 |
| Medium | 6 | 3 | 1 | 10 |
| **Total** | **25** | **15** | **10** | **50** |

---

## 4. UAT Scenarios (16)

| UAT-ID | Scenario | Expected |
|--------|----------|----------|
| UAT-SC01 | First sync faster than before | Noticeable improvement |
| UAT-SC02 | Second sync feels near-instant | Cache warm |
| UAT-SC03 | Last sync date updates | Correct date shown |
| UAT-SC04 | Location dropdown correct | All locations present |
| UAT-SC05 | Employee list correct | All active employees |
| UAT-SC06 | Materials dropdown correct | Filtered by cost center |
| UAT-SC07 | Incentive dropdown correct (opcode filtering) | No cross-contamination |
| UAT-SC08 | Fruit care areas/parcels/layouts correct | All entries present |
| UAT-SC09 | Batch numbers fresh | Newly created batch visible |
| UAT-SC10 | New location appears after TTL | Eventually visible |
| UAT-SC11 | Sync after 24+ hours fresh | All TTLs expired; fresh data |
| UAT-SC12 | Sync with poor network | Completes; no crash |
| UAT-SC13 | Sync with no network | Clear error; data retained |
| UAT-SC14 | Two supervisors sync simultaneously | Both succeed independently |
| UAT-SC15 | Rapid double-tap Sync | Single sync; no crash |
| UAT-SC16 | Sync after report submission | Batch/doc numbers updated |

---

## 5. Regression Checklist (22 checks)

### Sync Regression (8)

- [ ] All 20 sync endpoints return 200 with valid JSON
- [ ] Hive box names unchanged
- [ ] Sync dialog shows progress and final status
- [ ] Last sync date saved correctly
- [ ] Sync error handling (SnackBar)
- [ ] FruitCareSyncResult.hasErrors still works
- [ ] OtherOperationsSyncResult.hasErrors still works
- [ ] Sequential fallback if needed

### DAR Form Regression (9)

- [ ] GenServ form — dropdowns populated, submit works
- [ ] Harvest form — full pipeline works
- [ ] Fertilization form — type, material, cost center correct
- [ ] Chemical Mixing form — mixing operations correct
- [ ] Survey form — survey parcels and layouts load
- [ ] Fruit Care form — areas, parcels, layouts populated
- [ ] ERAD/Gouging form — incentive opcode correct
- [ ] Gang/Group report — employee list, batch numbering
- [ ] Login/Auth — unaffected

### Backend Regression (5)

- [ ] POST/PUT/DELETE routes unaffected by cache
- [ ] Query param endpoints return correct filtered data
- [ ] Response schema unchanged from baseline
- [ ] Error handling (400, 404, 500) unaffected
- [ ] Server startup/shutdown clean

---

## 6. Defect Severity

| Severity | Definition |
|----------|-----------|
| P1 | Stale data causes incorrect submission; Hive corruption |
| P2 | Volatile endpoints cached too long; error cached; sync fails |
| P3 | TTL outside range; sync improvement below target |
| P4 | Cosmetic; logging gaps |
