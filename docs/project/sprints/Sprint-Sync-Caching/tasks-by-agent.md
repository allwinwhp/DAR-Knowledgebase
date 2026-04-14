# Tasks by Agent — Sprint-Sync-Caching

---

## Backend Squad

| ID | Task | Size | Days |
|----|------|------|------|
| BE-SC01 | Install `node-cache` dependency | S | 1 |
| BE-SC02 | Create `services/cacheClient.js` — singleton cache instance | S | 1 |
| BE-SC03 | Create `config/cacheTtl.js` — TTL map by route pattern | S | 1 |
| BE-SC04 | Create `middleware/cacheGet.js` — cache-aside Express middleware | M | 1 |

## Dev-Junior

| ID | Task | Size | Days |
|----|------|------|------|
| BE-SC05 | Add `cacheGet` to 14 static master-data endpoints (TTL 6 h) | M | 2 |
| BE-SC06 | Add `cacheGet` to 4 semi-static endpoints (TTL 15 min) | S | 2 |
| BE-SC07 | Add `cacheGet` to 6 bulk sync endpoints (TTL 15 min) | S | 2 |
| BE-SC08 | Add `cacheGet` to 4 volatile endpoints (TTL 60 s) | S | 2 |
| BE-SC10 | Create `controller/cache/cacheAdmin.js` — stats + flush | S | 2 |
| BE-SC09 | Register cache admin routes in `controller/routes/index.js` | S | 2 |

## Dev-Mid

| ID | Task | Size | Days |
|----|------|------|------|
| BE-SC11 | Unit tests for `cacheGet` middleware | M | 2 |
| BE-SC12 | Integration smoke test — verify `X-Cache` headers | M | 3 |
| FE-SC02 | Batch Hive writes — FruitCareRepository | M | 2 |
| FE-SC03 | Batch Hive writes — OtherOperationsRepository | M | 2 |
| FE-SC04 | Batch Hive writes — UserRepository | S | 2 |
| FE-SC05 | Batch Hive writes — AccountRepository | S | 2 |
| FE-SC06 | Batch Hive writes — BatchNoRepository | S | 2 |
| FE-SC07 | Batch Hive writes — DocNoRepository | S | 2 |

## Frontend Squad

| ID | Task | Size | Days |
|----|------|------|------|
| FE-SC08 | Redesign SyncDialogService for group-based progress | M | 1–2 |
| FE-SC09 | Add error-resilient `Future.wait` wrapper (`_syncGroup`) | S | 2 |
| FE-SC01 | Parallelize sync calls in `_syncData()` using `Future.wait` groups | L | 2–3 |

## Dev-Senior

| ID | Task | Size | Days |
|----|------|------|------|
| — | Code review: backend cache middleware PR | — | 3 |
| — | Code review: frontend parallel sync PR | — | 3–4 |
| — | Technical risk assessment and merge strategy | — | 1 |

## QA Lead

| ID | Task | Size | Days |
|----|------|------|------|
| — | Test strategy and test matrix (50 test cases) | M | 1 |
| — | UAT scenario definition (16 scenarios) | S | 1 |
| — | Regression checklist (22 checks) | S | 1 |

## QA Mid / QA Junior

| ID | Task | Size | Days |
|----|------|------|------|
| FE-SC10 | Manual QA — measure sync time before/after | S | 4–5 |
| — | Execute backend cache test cases (TC-SC01–SC25) | M | 3–4 |
| — | Execute frontend test cases (TC-SC30–SC44) | M | 4 |
| — | Execute E2E test cases (TC-SC50–SC59) | M | 4–5 |
| — | Execute UAT scenarios (UAT-SC01–SC16) | M | 5 |
| — | Regression testing | M | 4–5 |

## DevOps

| ID | Task | Size | Days |
|----|------|------|------|
| — | CI/CD impact assessment | S | 1 |
| — | Deployment plan | S | 1 |
| — | Post-deployment monitoring checklist | S | 5 |

## PM

| ID | Task | Size | Days |
|----|------|------|------|
| — | Sprint plan and goal framing | S | 1 |
| — | Sprint review and sign-off | S | 5 |

## PO

| ID | Task | Size | Days |
|----|------|------|------|
| — | Scope validation and acceptance criteria | S | 1 |

## Architect

| ID | Task | Size | Days |
|----|------|------|------|
| — | Technical design (cache-aside pattern, TTL table, sequence diagrams) | M | 1 |

---

## Timeline

```
Day 1:  Backend core (BE-SC01–04) + Frontend batch writes (FE-SC02–07) + Dialog redesign (FE-SC08)
Day 2:  Wire endpoints (BE-SC05–08) + Cache admin (BE-SC09–10) + Parallel sync (FE-SC01, FE-SC09) + Tests (BE-SC11)
Day 3:  Integration tests (BE-SC12) + Backend PR review + Frontend PR
Day 4:  Integration testing on device + QA execution + Bug fixes
Day 5:  UAT + Regression + Sprint review + Sign-off
```
