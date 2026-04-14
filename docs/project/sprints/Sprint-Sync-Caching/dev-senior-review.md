# Dev Senior Review — Sprint-Sync-Caching

**Reviewer**: Dev Senior
**Sprint**: Sprint-Sync-Caching
**Branch**: MDAG-339

---

## 1. Implementation Scope Assessment

**Verdict: Appropriately sized for a 5-day sprint.**

| Area | New files | Modified files | Complexity |
|------|-----------|----------------|------------|
| Backend cache layer | 3 core + 1 admin | ~24 endpoint files (one-line addition each) + `package.json` | Low-Medium |
| Frontend parallel sync | 0 | 1 (`home_sync.dart`) | Medium |
| Frontend batch Hive writes | 0 | 6 repository files | Low |
| Frontend progress dialog | 0 | 1 (`sync_dialog_service.dart`) | Medium |

The 24 endpoint modifications are mechanical (identical one-liner per file). Actual design work is in 3 backend files and 8 frontend files.

---

## 2. Capacity Allocation

| Task | Assigned | Est. effort |
|------|----------|-------------|
| Cache middleware + TTL config + client | Backend squad | 1 day |
| Apply cacheGet to 24 endpoints | Dev-junior | 0.5 day |
| Cache admin endpoints | Dev-junior | 0.5 day |
| Cache middleware tests | Dev-mid | 0.5 day |
| Parallel sync refactor | Frontend squad | 1.5 days |
| Progress dialog redesign | Frontend squad | 0.5 day |
| Batch Hive writes (6 repos) | Dev-mid | 0.5 day |
| Integration testing | Dev-mid + Frontend | 0.5 day |

**Total**: ~5.5 developer-days. Fits in 5-day sprint with buffer.

---

## 3. Technical Risks

### Risk 1 (High): Deployment Target Clarity

A `vercel.json` is present in DAR_Middleware. If deployed on Vercel serverless, `node-cache` will have near-zero hit rate (cold starts). The queue reaper's `setInterval` also wouldn't work on serverless.

**Action**: Confirm deployment target before Day 1. If persistent host (likely, given queue reaper behavior): proceed. If Vercel: switch to `Cache-Control` headers or Vercel KV.

### Risk 2 (Medium): Memory Pressure

`node-cache` stores full JSON payloads in Node.js heap. With `maxKeys: 200` and typical dataset sizes (100s-1000s rows), estimated cache memory is ≤50 MB.

**Mitigation**: Monitor via `/cache/stats`. Reduce TTLs for bulk endpoints if memory spikes.

### Risk 3 (Medium): Stale Data

Semi-static endpoints (employees, materials) could serve outdated data for up to 15 minutes.

**Mitigation**: Acceptable for field use. Admin `/cache/flush` for urgent updates.

### Risk 4 (Medium): Hive addAll Atomicity

`addAll()` is not truly atomic — same risk as current `add()` loop. The refactor makes it faster (smaller window for interruption).

### Risk 5 (High): Parallel Sync Error Handling

`Future.wait` throws on first error by default. Must use `eagerError: false` or per-call wrapping.

**Mitigation**: `SyncOutcome` wrapper pattern in FE-SC09.

### Risk 6 (Medium): Progress Dialog Coupling

Current `SyncDialogService` has hardcoded indices 0–19. Must redesign for group-based progress.

---

## 4. Code Review Expectations

### Backend PRs

- Cache key uses `req.originalUrl` (not `req.path`) to avoid key collisions between routers
- `syncBatchNumbers`, `syncDocNumbers`, `syncAccounts` have short TTL (60 s) — verify in `cacheTtl.js`
- `cacheGet` comes after auth middleware (none currently, but future-proof)
- `useClones: false` → handlers must not mutate response objects
- Only 2xx responses cached

### Frontend PRs

- `Future.wait` uses `eagerError: false` or wraps individual futures
- `addAll()` receives correctly-typed lists, not `List<dynamic>`
- Progress indices updated correctly for group model
- `mounted` checks after async awaits
- `FruitCareSyncResult.hasErrors` logic preserved

---

## 5. Merge Strategy

**Backend first, then frontend.** Both can be developed in parallel.

```
Day 1-2:  Backend cache layer → PR to MDAG-339
Day 2-3:  Frontend parallel sync + batch writes → PR to MDAG-339
Day 3-4:  Integration testing on combined branch
Day 4-5:  Bug fixes, review cycles, polish
```

---

## 6. Estimated Timeline

| Day | Backend | Frontend | Dev-Mid | Dev-Junior |
|-----|---------|----------|---------|------------|
| 1 | cacheClient, cacheTtl, cacheGet | home_sync parallel refactor | Cache middleware tests | — |
| 2 | Review + finalize TTLs | SyncDialogService redesign | addAll refactors (6 repos) | Wire 24 endpoints |
| 3 | Backend PR review + merge | Frontend PR | Frontend PR (addAll) | Cache admin |
| 4 | Integration support | Integration on device | Integration testing | Smoke test all endpoints |
| 5 | Bug fixes + final review | Bug fixes + final review | Bug fixes | — |

---

## 7. Definition of Done

- [ ] `cacheClient.js` exports configured `node-cache` with `maxKeys`, `checkperiod`, `useClones` documented
- [ ] `cacheTtl.js` has correct TTL for all tiers; volatile endpoints are ≤60 s
- [ ] `cacheGet.js` handles hit/miss/bypass correctly
- [ ] All 24 data_sync GET endpoints have `cacheGet` middleware applied
- [ ] Jest tests cover: hit, miss, TTL expiry, excluded route, key uniqueness
- [ ] `home_sync.dart` uses `Future.wait` with 5 parallel groups
- [ ] Error isolation: one failed endpoint doesn't abort the group
- [ ] `SyncDialogService` shows group-based progress
- [ ] 6 repositories use `addAll()` instead of `add()` loops
- [ ] Partial-failure behavior (snackbar with error count) still works
- [ ] Full sync tested on physical device — no regressions
- [ ] No new linter warnings in either repo
- [ ] Deployment target confirmed and documented

---

## Pre-Sprint Action Item

**Confirm production deployment target** before Day 1. If it's a persistent host (VM with PM2/systemd): proceed with `node-cache`. If serverless (Vercel): pivot to `Cache-Control` headers or Vercel KV.
