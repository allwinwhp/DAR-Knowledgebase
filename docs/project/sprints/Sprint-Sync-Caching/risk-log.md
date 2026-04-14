# Risk Log — Sprint-Sync-Caching

---

| # | Risk | Severity | Impact | Mitigation | Owner | Status |
|---|------|----------|--------|------------|-------|--------|
| 1 | **Deployment target unclear** — `vercel.json` present but queue reaper uses `setInterval` (serverless-incompatible). If deployed on Vercel, `node-cache` will have near-zero hit rate. | High | Cache ineffective; no performance gain | Confirm deployment target before Day 1. If Vercel: use `Cache-Control` headers or Vercel KV instead. If persistent host: proceed as planned. | Dev-Senior / DevOps | Open |
| 2 | **Stale data from cached endpoints** — Supervisor may not see newly added employees, locations, or materials until TTL expires | Medium | Supervisor works with outdated reference data for up to 15 min (semi-static) or 6 h (static) | TTL values chosen conservatively; admin `/cache/flush` endpoint for urgent updates; document purge procedure for IT staff | PO / Backend | Mitigated |
| 3 | **Memory pressure from large cached payloads** — Bulk fruit-care and other-operations endpoints may return large datasets | Medium | Node.js process OOM if too many large payloads cached | `maxKeys: 200` safety bound; monitor via `/cache/stats`; set `checkperiod` for proactive eviction | Backend | Mitigated |
| 4 | **`node-cache` eviction behavior** — Lazy TTL check, no LRU eviction | Low | Stale entries consume memory | Set `checkperiod` to half the shortest TTL; `useClones: false` for performance | Backend | Mitigated |
| 5 | **Parallel sync error propagation** — `Future.wait` throws on first error by default | High | One failed endpoint kills entire sync group | Use `eagerError: false`; wrap each future in try/catch returning `SyncOutcome` | Frontend | Mitigated |
| 6 | **Hive `addAll()` atomicity** — Not truly atomic; app killed mid-write after `clear()` = partial data | Medium | Partial reference data in local store | Same risk exists today with `add()` loops; `addAll()` is faster so the window is smaller. Full fix (temp box swap) deferred. | Frontend | Accepted |
| 7 | **Progress dialog UX with parallel groups** — Current dialog tightly coupled to 20 sequential steps | Medium | Dialog shows inconsistent state during parallel execution | Redesign as group-based progress (5 groups); FE-SC08 addresses this | Frontend | Mitigated |
| 8 | **Cache key collision** — Two endpoint files both define `router.get('/')` but are mounted at different paths | Low | Wrong data returned from cache | Cache key uses `req.originalUrl` (full path including mount point), not `req.path`. Verified safe. | Backend | Mitigated |
| 9 | **Cold cache after server restart** — First sync hits all DB queries | Low | First supervisor to sync after restart gets slow sync | Acceptable; cache warms within one cycle. Cache warming deferred to future sprint. | DevOps | Accepted |
