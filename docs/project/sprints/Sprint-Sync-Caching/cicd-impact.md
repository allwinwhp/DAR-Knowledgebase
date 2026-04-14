# CI/CD Impact Assessment — Sync Caching

**[Senior DevOps]** | Sprint-Sync-Caching | 2026-04-15

---

## 1. Impact on the Existing Express App

The caching sprint adds a transparent middleware that wraps existing data_sync GET endpoints. No routes are added or removed. The cache is in-process memory (no Redis, no external service) — **zero infrastructure change**.

### New Files

```
middleware/cacheGet.js       — Express middleware
services/cacheClient.js      — node-cache singleton
config/cacheTtl.js           — TTL configuration per endpoint
```

### No Database Changes

No migrations, no schema changes, no seed data. The highest-risk deployment step is eliminated.

---

## 2. New Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `node-cache` | latest (`^5.x`) | In-process key-value cache with TTL |

```bash
npm install node-cache
```

Frontend: No new dependencies.

---

## 3. Branch Strategy

| Repo | Feature Branch | Base Branch | Merge Target |
|------|---------------|-------------|--------------|
| DAR_Middleware | `feature/sync-caching` | `dev` | `dev` |
| main_dar_app | `feature/sync-caching` | `MDAG-339` | `MDAG-339` |

Backend merges and deploys first. Tag `sync-caching-v1.0` after verified deployment.

---

## 4. Environment Variables

No required env vars. Optional TTL overrides documented in `config/cacheTtl.js`.

---

## 5. Pre-Deployment Checklist

### Backend (10 gates)

| # | Gate | Required |
|---|------|----------|
| 1 | `npm install` completes without errors | Yes |
| 2 | Server starts with cache middleware loaded | Yes |
| 3 | Cached endpoints return `X-Cache: MISS` on first call | Yes |
| 4 | Cached endpoints return `X-Cache: HIT` on second call | Yes |
| 5 | Cache expires after TTL | Yes |
| 6 | Non-cached endpoints unaffected | Yes |
| 7 | Health endpoint returns 200 | Yes |
| 8 | `/sendDARForm/*` still works | Yes |
| 9 | `/queue/*` endpoints still work | Yes |
| 10 | Memory stable under repeated sync | Yes |

### Frontend (5 gates)

| # | Gate | Required |
|---|------|----------|
| 1 | App builds without errors | Yes |
| 2 | Parallel sync faster than baseline | Yes |
| 3 | All sync data arrives correctly | Yes |
| 4 | Batch Hive writes don't corrupt storage | Yes |
| 5 | Existing app flows unaffected | Yes |

---

## 6. Rollback Plan

**Risk level: Low.** No database changes. Cache evaporates on restart.

### Backend

```bash
pm2 stop dar-middleware
git checkout <previous-commit-sha>
npm install
pm2 start dar-middleware
```

Time: ~2 minutes. No data cleanup.

### Frontend

Redistribute previous APK. Sequential sync still works against the backend.
