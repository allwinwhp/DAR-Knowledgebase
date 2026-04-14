# Deliverables — Sprint-Sync-Caching

---

## Backend Deliverables (DAR_Middleware)

| # | Deliverable | File(s) | Owner |
|---|------------|---------|-------|
| 1 | Cache client singleton | `services/cacheClient.js` | Backend |
| 2 | TTL configuration map | `config/cacheTtl.js` | Backend |
| 3 | Cache-aside middleware | `middleware/cacheGet.js` | Backend |
| 4 | Cache admin endpoints | `controller/cache/cacheAdmin.js` | Backend |
| 5 | Route registration for cache admin | `controller/routes/index.js` (modified) | Backend |
| 6 | Cache middleware applied to 14 static endpoints | `controller/data_sync/*.js` (14 files modified) | Dev-Junior |
| 7 | Cache middleware applied to 4 semi-static endpoints | `controller/data_sync/*.js` (4 files modified) | Dev-Junior |
| 8 | Cache middleware applied to 6 bulk endpoints | `getFruitCareData.js`, `getOtherOperationsData.js` (modified) | Dev-Junior |
| 9 | Cache middleware applied to 4 volatile endpoints | `controller/data_sync/*.js` (4 files modified) | Dev-Junior |
| 10 | Unit tests for cache middleware | `__tests__/middleware/cacheGet.test.js` | Dev-Mid |
| 11 | Integration smoke test | `__tests__/integration/cacheSmoke.test.js` | Dev-Mid |
| 12 | `node-cache` dependency | `package.json` (modified) | Backend |

## Frontend Deliverables (main_dar_app)

| # | Deliverable | File(s) | Owner |
|---|------------|---------|-------|
| 13 | Parallel sync with `Future.wait` groups | `home_sync.dart` (modified) | Frontend |
| 14 | Error-resilient sync group helper | `home_sync.dart` (modified) | Frontend |
| 15 | Group-based progress dialog | `sync_dialog_service.dart` (modified) | Frontend |
| 16 | Batch Hive writes — FruitCareRepository | `fruit_care_repository.dart` (modified) | Dev-Mid |
| 17 | Batch Hive writes — OtherOperationsRepository | `other_operations_repository.dart` (modified) | Dev-Mid |
| 18 | Batch Hive writes — UserRepository | `user_repository.dart` (modified) | Dev-Mid |
| 19 | Batch Hive writes — AccountRepository | `account_repository.dart` (modified) | Dev-Mid |
| 20 | Batch Hive writes — BatchNoRepository | `batch_no_repository.dart` (modified) | Dev-Mid |
| 21 | Batch Hive writes — DocNoRepository | `doc_no_repository.dart` (modified) | Dev-Mid |

## Documentation Deliverables

| # | Deliverable | File | Owner |
|---|------------|------|-------|
| 22 | Sprint plan | `sprint-plan.md` | PM |
| 23 | Scope | `scope.md` | PO |
| 24 | Technical design | `technical-design.md` | Architect |
| 25 | Backend tasks | `backend-tasks.md` | Backend |
| 26 | Frontend tasks | `frontend-tasks.md` | Frontend |
| 27 | Test matrix | `test-matrix.md` | QA Lead |
| 28 | Tasks by agent | `tasks-by-agent.md` | SM |
| 29 | CI/CD impact | `cicd-impact.md` | DevOps |
| 30 | Deployment plan | `deployment-plan.md` | DevOps |
| 31 | Risk log | `risk-log.md` | SM / PM |
| 32 | Dev Senior review | `dev-senior-review.md` | Dev Senior |

---

## Summary

| Category | Count |
|----------|-------|
| New backend files | 5 (3 core + 1 admin + 1 dependency) |
| Modified backend files | 24 endpoint files + 1 route index |
| Modified frontend files | 8 files |
| Test files | 2 new |
| Documentation | 11 artifacts |
| **Total deliverables** | 32 |
