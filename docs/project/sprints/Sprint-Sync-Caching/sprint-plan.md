# Sprint Plan — Sync Caching

**Sprint**: Sprint-Sync-Caching
**Goal**: Eliminate redundant MSSQL queries during mobile sync by implementing API-level in-memory caching on DAR_Middleware for static/semi-static data_sync endpoints, plus frontend optimizations (parallel sync calls, batch Hive writes) to dramatically reduce sync wall-clock time.
**Timeline**: 5 working days (single sprint)
**Branch**: `feature/sync-caching` from `dev` (DAR_Middleware) | `feature/sync-caching` from `MDAG-339` (main_dar_app)

---

## 1. Sprint goal

- **Business**: Farm supervisors experience significantly faster data sync (target: ≥50% reduction on repeat sync, ≥40% from parallelism), reducing field downtime and improving adoption.
- **Feature**: In-memory cache-aside middleware (`node-cache`) with tiered TTLs applied to 22+ data_sync endpoints; frontend parallel sync groups with `Future.wait`; batch Hive writes replacing row-by-row inserts.
- **Quality**: 50 test cases covering cache behavior, parallel sync, batch writes, and regression. UAT scenarios for farm supervisor persona.

---

## 2. Scope summary

| Category | Items |
|----------|--------|
| **Epic** | Epic-SC — Sync Caching |
| **User stories** | US-SC01–US-SC04 (faster repeat sync, parallel download, faster writes, data freshness) |
| **Technical stories** | TS-SC01–TS-SC07 (cache middleware, TTL config, route application, cache admin, parallel sync, batch Hive, progress dialog) |
| **Out of scope** | ETag/304, Redis, delta sync, query optimization, submission queue changes |

Full scope: [scope.md](scope.md)

---

## 3. Sprint structure — 5 days

| Phase | Days | Activities |
|-------|------|-----------|
| **Phase 1: Core Implementation** | Days 1–2 | Cache middleware + TTL config (backend); batch Hive writes (frontend); progress dialog redesign (frontend) |
| **Phase 2: Integration** | Days 2–3 | Apply cache to all endpoints (backend); parallel sync groups (frontend); cache admin endpoints |
| **Phase 3: Testing** | Days 3–4 | Unit tests, integration tests, benchmark measurements |
| **Phase 4: Review & QA** | Days 4–5 | Code review, regression testing, UAT, bug fixes |

---

## 4. Execution alignment

Per [delivery-orchestrator](../../../agents/delivery-orchestrator.md) (DAR context):

- **Frontend**: main_dar_app (Flutter)
- **Backend**: DAR_Middleware (Node/Express, MSSQL, Knex)
- **QA first**: Test strategy and test cases before dev implementation
- **No DB changes**: No migrations needed — cache is in-process memory
- **Backend and frontend are independent**: Can be developed and deployed in parallel

---

## 5. Team capacity allocation

| Task | Assigned to | Est. effort |
|------|-------------|-------------|
| Cache middleware + TTL config + cacheClient | Backend squad | 1 day |
| Apply cacheGet to 22+ endpoint files | Dev-junior | 0.5 day |
| Cache admin endpoints | Dev-junior | 0.5 day |
| Cache middleware unit tests | Dev-mid | 0.5 day |
| Parallel sync refactor (home_sync.dart) | Frontend squad | 1.5 days |
| Progress dialog redesign | Frontend squad | 0.5 day |
| Batch Hive writes (6 repositories) | Dev-mid | 0.5 day |
| Integration testing | Dev-mid + Frontend squad | 0.5 day |
| Code review + bug fixes | Dev-senior | 1 day |

---

## 6. Key risks

| Risk | Mitigation |
|------|------------|
| Deployment target unclear (Vercel vs persistent host) | Confirm before Day 1; `node-cache` works best on persistent host |
| Stale data from cached endpoints | Tiered TTLs; admin flush endpoint; documented procedures |
| Memory pressure from large cached payloads | `maxKeys: 200`; monitor via `/cache/stats` |
| Parallel sync error propagation | `eagerError: false`; per-call try/catch |

---

## 7. Deliverables (mandatory)

| # | Artifact | Owner | Status |
|---|----------|--------|--------|
| 1 | [sprint-plan.md](sprint-plan.md) | PM | Done |
| 2 | [scope.md](scope.md) | PO / Business | Done |
| 3 | [technical-design.md](technical-design.md) | Architect | Done |
| 4 | [backend-tasks.md](backend-tasks.md) | Backend squad | Done |
| 5 | [frontend-tasks.md](frontend-tasks.md) | Frontend squad | Done |
| 6 | [test-matrix.md](test-matrix.md) | QA Lead | Done |
| 7 | [tasks-by-agent.md](tasks-by-agent.md) | SM | Done |
| 8 | [deliverables.md](deliverables.md) | Dev squads | Done |
| 9 | [cicd-impact.md](cicd-impact.md) | DevOps | Done |
| 10 | [deployment-plan.md](deployment-plan.md) | DevOps | Done |
| 11 | [risk-log.md](risk-log.md) | SM / PM | Done |
| 12 | [dev-senior-review.md](dev-senior-review.md) | Dev Senior | Done |

---

## 8. Key references

- [Technical design](technical-design.md)
- [Backend tasks](backend-tasks.md) — BE-SC01–BE-SC12
- [Frontend tasks](frontend-tasks.md) — FE-SC01–FE-SC10
- [dar-system-context](../../project/dar-system-context.md)
- [delivery-orchestrator](../../../agents/delivery-orchestrator.md)
