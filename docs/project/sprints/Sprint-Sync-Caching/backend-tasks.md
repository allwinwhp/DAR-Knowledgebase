# Backend Tasks — Sprint-Sync-Caching

**Squad**: Backend
**Repo**: DAR_Middleware (branch `MDAG-339`)
**Goal**: Implement cache-aside middleware using `node-cache` to eliminate redundant MSSQL hits during mobile sync.

---

## Summary

| ID | Description | Size | Depends On |
|----|-------------|------|------------|
| BE-SC01 | Install `node-cache` dependency | S | — |
| BE-SC02 | Create `services/cacheClient.js` — singleton cache instance | S | BE-SC01 |
| BE-SC03 | Create `config/cacheTtl.js` — TTL map by route pattern | S | — |
| BE-SC04 | Create `middleware/cacheGet.js` — cache-aside Express middleware | M | BE-SC02, BE-SC03 |
| BE-SC05 | Add `cacheGet` to 14 static master-data endpoints (TTL 6 h) | M | BE-SC04 |
| BE-SC06 | Add `cacheGet` to 4 semi-static endpoints (TTL 15 min) | S | BE-SC04 |
| BE-SC07 | Add `cacheGet` to 6 bulk sync endpoints (TTL 15 min) | S | BE-SC04 |
| BE-SC08 | Add `cacheGet` to 4 volatile endpoints (TTL 60 s) | S | BE-SC04 |
| BE-SC09 | Register cache admin routes in `controller/routes/index.js` | S | BE-SC10 |
| BE-SC10 | Create `controller/cache/cacheAdmin.js` — stats + flush endpoints | S | BE-SC02 |
| BE-SC11 | Unit tests for `cacheGet` middleware | M | BE-SC04 |
| BE-SC12 | Integration smoke test — verify `X-Cache` headers | M | BE-SC05–SC08 |

---

## BE-SC01 — Install `node-cache`

```bash
npm install node-cache
```

---

## BE-SC02 — Create `services/cacheClient.js`

```js
const NodeCache = require('node-cache');
const logger = require('../logger');

const cache = new NodeCache({
  stdTTL: 900,
  checkperiod: 120,
  useClones: false,
  maxKeys: 200,
});

cache.on('expired', (key) => {
  logger.info(`[CACHE] Key expired: ${key}`);
});

const logStats = () => {
  const { hits, misses, keys } = cache.getStats();
  logger.info(`[CACHE STATS] keys=${keys} hits=${hits} misses=${misses} hitRate=${hits + misses > 0 ? ((hits / (hits + misses)) * 100).toFixed(1) : 0}%`);
};

setInterval(logStats, 5 * 60 * 1000).unref();

module.exports = cache;
```

---

## BE-SC03 — Create `config/cacheTtl.js`

```js
const TTL = {
  '/api/holidays':            21600,
  '/api/location':            21600,
  '/api/disease':             21600,
  '/api/farm':                21600,
  '/api/layout':              21600,
  '/api/parcel':              21600,
  '/api/material':            21600,
  '/api/variety':             21600,
  '/api/fertilizer-type':     21600,
  '/api/operation-config':    21600,
  '/api/mixing-operations':   21600,
  '/api/chem-mixing':         21600,
  '/api/hectarage-area':      21600,
  '/api/hectarage-mps':       21600,
  '/api/employee-list':          900,
  '/api/fertilizer-materials':   900,
  '/api/fertilizer-cost-center': 900,
  '/api/incentive':              900,
  '/api/fruit-care-sync':        900,
  '/api/other-operations-sync':  900,
  '/api/accounts':     60,
  '/api/batch-numbers': 60,
  '/api/doc-numbers':  60,
  '/api/users':        60,
};

const sortedPrefixes = Object.keys(TTL).sort((a, b) => b.length - a.length);

function getTtl(url) {
  for (const prefix of sortedPrefixes) {
    if (url.startsWith(prefix)) return TTL[prefix];
  }
  return null;
}

module.exports = { TTL, getTtl };
```

---

## BE-SC04 — Create `middleware/cacheGet.js`

```js
const cache = require('../services/cacheClient');
const { getTtl } = require('../config/cacheTtl');
const logger = require('../logger');

function cacheGet(req, res, next) {
  const ttl = getTtl(req.originalUrl);
  if (ttl === null) return next();

  const key = req.originalUrl;
  const cached = cache.get(key);

  if (cached !== undefined) {
    logger.info(`[CACHE HIT] ${key}`);
    res.set('X-Cache', 'HIT');
    return res.status(200).json(cached);
  }

  logger.info(`[CACHE MISS] ${key}`);
  const originalJson = res.json.bind(res);

  res.json = (body) => {
    if (res.statusCode >= 200 && res.statusCode < 300) {
      cache.set(key, body, ttl);
    }
    res.set('X-Cache', 'MISS');
    return originalJson(body);
  };

  next();
}

module.exports = cacheGet;
```

---

## BE-SC05 — Static Master Data Endpoints (14 files)

**Change pattern** for each file:

1. Add require: `const cacheGet = require('../../middleware/cacheGet');`
2. Add middleware: `router.get('/', cacheGet, async (req, res) => {`

| # | File | Route |
|---|------|-------|
| 1 | `controller/data_sync/getHoliday.js` | `/api/holidays` |
| 2 | `controller/data_sync/getLocation.js` | `/api/location` |
| 3 | `controller/data_sync/getDisease.js` | `/api/disease` |
| 4 | `controller/data_sync/getFarm.js` | `/api/farm` |
| 5 | `controller/data_sync/getLayout.js` | `/api/layout` |
| 6 | `controller/data_sync/getParcel.js` | `/api/parcel` |
| 7 | `controller/data_sync/getMaterial.js` | `/api/material` |
| 8 | `controller/data_sync/getVariety.js` | `/api/variety` |
| 9 | `controller/data_sync/getFertType.js` | `/api/fertilizer-type` |
| 10 | `controller/data_sync/getOperationConfig.js` | `/api/operation-config` |
| 11 | `controller/data_sync/getMixingOperations.js` | `/api/mixing-operations` |
| 12 | `controller/data_sync/getChemMixing.js` | `/api/chem-mixing` |
| 13 | `controller/data_sync/getHectarageArea.js` | `/api/hectarage-area` |
| 14 | `controller/data_sync/getHectarageMPS.js` | `/api/hectarage-mps` |

---

## BE-SC06 — Semi-Static Endpoints (4 files)

Same change pattern as BE-SC05.

| # | File | Route |
|---|------|-------|
| 1 | `controller/data_sync/getEmployeeList.js` | `/api/employee-list` |
| 2 | `controller/data_sync/getFertilizerMaterials.js` | `/api/fertilizer-materials` |
| 3 | `controller/data_sync/getFertilizerCostCenter.js` | `/api/fertilizer-cost-center` |
| 4 | `controller/data_sync/getIncentive.js` | `/api/incentive` |

---

## BE-SC07 — Bulk Sync Endpoints (2 files, 6 routes)

### `getFruitCareData.js`

Add `cacheGet` to 3 `/all` routes:

```js
router.get('/areas/all', cacheGet, async (req, res) => { ... });
router.get('/parcels/all', cacheGet, async (req, res) => { ... });
router.get('/layouts/all', cacheGet, async (req, res) => { ... });
```

### `getOtherOperationsData.js`

Add `cacheGet` to 3 `/all` routes:

```js
router.get('/parcels/all', cacheGet, async (req, res) => { ... });
router.get('/layouts/all', cacheGet, async (req, res) => { ... });
router.get('/survey-layouts/all', cacheGet, async (req, res) => { ... });
```

---

## BE-SC08 — Volatile Endpoints (4 files)

Same change pattern as BE-SC05.

| # | File | Route | TTL |
|---|------|-------|-----|
| 1 | `controller/data_sync/syncAccounts.js` | `/api/accounts` | 60 s |
| 2 | `controller/data_sync/syncBatchNumbers.js` | `/api/batch-numbers` | 60 s |
| 3 | `controller/data_sync/syncDocNumbers.js` | `/api/doc-numbers` | 60 s |
| 4 | `controller/data_sync/getUsers.js` | `/api/users` | 60 s |

---

## BE-SC10 — Cache Admin Endpoints

```js
const express = require('express');
const router = express.Router();
const cache = require('../../services/cacheClient');
const logger = require('../../logger');

router.get('/stats', (req, res) => {
  const stats = cache.getStats();
  const keys = cache.keys();
  const hitRate = stats.hits + stats.misses > 0
    ? ((stats.hits / (stats.hits + stats.misses)) * 100).toFixed(1)
    : '0.0';
  res.json({
    keys: keys.length,
    hits: stats.hits,
    misses: stats.misses,
    hitRate: `${hitRate}%`,
    cachedRoutes: keys,
  });
});

router.post('/flush', (req, res) => {
  const { prefix } = req.query;
  if (prefix) {
    const keys = cache.keys().filter((k) => k.startsWith(prefix));
    keys.forEach((k) => cache.del(k));
    logger.info(`[CACHE] Flushed ${keys.length} keys matching prefix "${prefix}"`);
    res.json({ flushed: keys.length, prefix });
  } else {
    cache.flushAll();
    logger.info('[CACHE] Flushed ALL keys');
    res.json({ flushed: 'all' });
  }
});

module.exports = router;
```

---

## BE-SC09 — Register Admin Routes

Add to `controller/routes/index.js` before `module.exports`:

```js
controllerRouter.use("/cache", require("../cache/cacheAdmin"));
```

---

## BE-SC11 — Unit Tests

File: `__tests__/middleware/cacheGet.test.js`

Cover: cache hit, cache miss, non-cached route bypass, error response exclusion, query-string key isolation.

---

## BE-SC12 — Integration Smoke Test

File: `__tests__/integration/cacheSmoke.test.js`

Using `supertest`: verify MISS→HIT cycle, flush clears keys, stats endpoint returns valid JSON.

---

## Execution Order

```
BE-SC01  (npm install)
   ↓
BE-SC02 + BE-SC03  (parallel)
   ↓
BE-SC04  (middleware)
   ↓
BE-SC05 + BE-SC06 + BE-SC07 + BE-SC08  (parallel)
   ↓
BE-SC10 + BE-SC09  (admin routes)
   ↓
BE-SC11 + BE-SC12  (tests)
```
