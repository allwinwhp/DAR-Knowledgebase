# Sprint-Sync-Caching — Scope

**[PO]** — Product Owner sign-off on scope, prioritization, and acceptance criteria.

**Problem**: The DAR sync process is extremely slow. `_syncData()` in `home_sync.dart` calls 20 sync endpoints **sequentially** — each one does a `SELECT *` from MSSQL, returns JSON to the Flutter app, which then clears the local Hive box and writes row-by-row. Many of these tables are static or semi-static master data (holidays, locations, disease types, varieties, materials, chem mixing, etc.) that rarely change but are re-fetched in full every single sync. Combined with single-threaded Hive writes and no API-level caching, the middleware hits the database hundreds of times per sync across all concurrent users.

**Solution**: Server-side in-memory caching on the DAR_Middleware backend for static and semi-static data_sync endpoints (tiered TTLs), plus frontend optimizations: parallel sync calls via `Future.wait()` and batch Hive writes via `addAll()`. No changes to database schema or existing query logic — the cache sits in front of Knex, not behind it.

**Branches**: `feature/sync-caching` from `dev` (DAR_Middleware) | `feature/sync-caching` from `MDAG-339` (main_dar_app)

---

## Epics

| Epic | Description |
|------|-------------|
| **Epic-SC** | Sync Caching — In-memory API response caching with tiered TTLs for DAR data_sync endpoints, eliminating redundant MSSQL queries for static/semi-static master data. Frontend parallel sync execution and batch Hive writes to reduce wall-clock sync time. |

---

## User stories

| ID | Title | Acceptance criteria | Deps |
|----|-------|---------------------|------|
| US-SC01 | Repeat sync is faster because static data is cached | 1. When a supervisor syncs and the server has already served the same static endpoint within its TTL window, the API returns the cached response without hitting MSSQL. 2. The supervisor experiences noticeably faster sync times on repeat syncs (target: ≥50% reduction for static-heavy syncs). 3. The data returned is identical in structure and content to an uncached response — the supervisor sees no data differences. | TS-SC01, TS-SC02, TS-SC03 |
| US-SC02 | Multiple data tables download simultaneously | 1. When a supervisor initiates sync, independent data endpoints are fetched in parallel groups rather than one-at-a-time. 2. The progress dialog updates as each group completes. 3. Total sync wall time is reduced compared to sequential execution (target: ≥40% reduction from parallelism alone). 4. If any single endpoint in a group fails, the error is reported and remaining endpoints in the group still complete. | TS-SC05, TS-SC07 |
| US-SC03 | Large dataset sync writes are faster | 1. FruitCare and OtherOperations data write to local storage in bulk rather than row-by-row. 2. The supervisor sees faster completion of sync steps for fruit care and other operations. 3. All data is correctly persisted — no data loss compared to the previous row-by-row approach. | TS-SC06 |
| US-SC04 | Data stays current within acceptable freshness windows | 1. Static master data refreshes from MSSQL at least every 6 hours. 2. Semi-static data refreshes at least every 15 minutes. 3. Bulk data refreshes at least every 15 minutes. 4. Volatile data (accounts, batch numbers, doc numbers, users) is always fetched fresh (no cache or TTL ≤ 60 s). 5. An admin can force-purge the cache if an urgent data update is made outside normal TTL windows. | TS-SC02, TS-SC04 |

---

## Technical stories

| ID | Title | Acceptance criteria |
|----|-------|---------------------|
| TS-SC01 | Create in-memory caching middleware for Express | 1. Install `node-cache` as a production dependency. 2. Create a reusable Express middleware in `middleware/cacheGet.js`. 3. On cache **hit**: return cached JSON with HTTP 200 — handler never invoked. 4. On cache **miss**: intercept response, store in cache with configured TTL. 5. Cache key = full request URL including query string. 6. Cache instance is a module-level singleton. |
| TS-SC02 | TTL tier configuration and environment overrides | 1. Define TTL config in `config/cacheTtl.js` with tiers: **STATIC** (6 h), **SEMI_STATIC** (15 m), **BULK** (15 m). 2. Each tier overridable via environment variables. 3. **VOLATILE** tier with TTL ≤60 s. 4. Configuration loaded at startup and logged. |
| TS-SC03 | Apply caching middleware to data_sync routes | 1. **Static tier** applied to 12+ endpoints. 2. **Semi-static tier** applied to 4 endpoints. 3. **Bulk tier** applied to 6 `/all` endpoints. 4. **No cache** on accounts, batch-numbers, doc-numbers, queue endpoints. 5. `X-Cache: HIT\|MISS` header on every cached response. |
| TS-SC04 | Cache admin endpoints (status + purge) | 1. `GET /api/cache/stats` returns hit/miss statistics. 2. `POST /api/cache/flush` clears the entire cache. 3. `POST /api/cache/flush?prefix=` clears keys by prefix. |
| TS-SC05 | Refactor `_syncData()` for parallel sync groups | 1. Execute sync calls in 5 parallel groups using `Future.wait()`. 2. Groups execute sequentially; calls within each group run in parallel. 3. If any call in a group fails, others continue. 4. Failures collected and shown in summary. |
| TS-SC06 | Batch Hive writes in repositories | 1. Replace per-row `await box.add(item)` loops with `await box.addAll(items)` in FruitCareRepository, OtherOperationsRepository, UserRepository, AccountRepository, BatchNoRepository, DocNoRepository. 2. `box.clear()` preserved before writes. 3. Data integrity verified. |
| TS-SC07 | Update progress dialog for parallel sync UX | 1. Display 5 group-level phases instead of 20 sequential steps. 2. Completed groups show checkmarks; in-progress shows spinner. 3. Progress bar advances proportionally per group. |

---

## Dependencies

```
TS-SC01 (cache middleware)
  └── TS-SC02 (TTL configuration)
       └── TS-SC03 (apply to routes)
            └── US-SC01 (faster repeat sync)
            └── US-SC04 (data freshness)
       └── TS-SC04 (cache admin API)
            └── US-SC04 (admin force-purge)

TS-SC05 (parallel sync groups) ── independent of backend caching
  └── TS-SC07 (progress dialog UX)
       └── US-SC02 (parallel download UX)

TS-SC06 (batch Hive writes) ── independent of all other stories
  └── US-SC03 (faster large writes)
```

**Critical path**: TS-SC01 → TS-SC02 → TS-SC03 (backend caching).

**Parallel work**:
- TS-SC04 can be developed alongside TS-SC03.
- TS-SC05 + TS-SC07 (frontend parallel sync) are independent of backend — can start Day 1.
- TS-SC06 (batch Hive writes) has zero dependencies — can start Day 1.

---

## Out of scope

- **ETag / If-None-Match conditional responses** — Flutter HTTP client does not honor HTTP cache headers by default.
- **Redis or distributed cache** — DAR runs on a single middleware instance.
- **Client-side HTTP caching** — Would require Dio or a caching HTTP adapter.
- **Incremental / delta sync** — Larger architectural change; deferred.
- **Database query optimization** — Caching reduces query frequency, not query cost.
- **Submission queue changes** — Already delivered in Sprint-Submission-Queue.
- **Cache warming on server startup** — Cache populates lazily.
- **Authentication on cache admin endpoints** — DAR middleware has no auth layer currently.

---

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Stale cache serves outdated master data | Supervisor doesn't see new employees for up to 15 minutes | Admin can force-purge via `/cache/flush`. Document purge procedure. |
| In-memory cache increases Node.js memory usage | Process could approach memory limits | Monitor via `/cache/stats`. Set `maxKeys` bound. |
| Parallel sync overwhelms middleware connections | Connection pool saturation | Groups execute sequentially (5 groups, 4-6 calls each). Backend caching means most are HITs. |
| `Future.wait()` error handling | One failure masks others | Use `eagerError: false`; individually try/catch each future. |
| `addAll()` Hive performance with large datasets | Could block isolate | Hive batches disk writes internally. Chunk if needed. |
| Cache key collisions for parameterized endpoints | Wrong data returned | Key is full URL including query string. Verify with tests. |
| Server restart clears cache — first sync is slow | Cold cache after deployment | Acceptable: cache warms within one sync cycle. |

---

*[PO] Scope approved. All user stories are from the farm supervisor persona. Acceptance criteria are testable and measurable. TTL tiers are clearly defined with environment overrides. Backend and frontend work streams are independent. No scope creep beyond caching and sync optimization.*
