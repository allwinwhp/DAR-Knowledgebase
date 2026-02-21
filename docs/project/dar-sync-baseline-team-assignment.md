# DAR — Sync Baseline: Full-Course Team Assignment

**Purpose**: Scale the “sync-queries-in-one-place” baseline work across the whole team, with more juniors and mid-levels. Single source of truth: [dar-sync-queries-baseline.md](dar-sync-queries-baseline.md).

**Context**: Constant battle of DB vs requirements; this baseline is the reference to check backend responses and frontend dropdowns.

---

## Roles and responsibilities

| Role | Responsibility | Artifacts |
|------|----------------|-----------|
| **Backend (backend-squad / dev-senior / dev-mid)** | Own and maintain the sync-queries baseline doc; ensure each endpoint’s SQL/params match the doc. | Updates to `dar-sync-queries-baseline.md` |
| **Frontend (frontend-squad / dev-mid)** | Map each sync endpoint to screens/dropdowns; document in baseline or a frontend-mapping section. | Endpoint → screen/dropdown table |
| **Juniors (dev-junior, qa-junior)** | Capture sample payloads from localhost:3000 (or env); fill baseline JSON; run checklist vs doc. | `baseline-api-responses.json`, checklist results |
| **Mid (dev-mid, qa-mid)** | Validate payloads vs baseline doc; run UAT checklist for dropdown counts; update baseline when findings are confirmed. | Validation notes, baseline-expected-results updates |
| **QA (qa-senior / qa-mid)** | Own test matrix and UAT checklist; sign off when dropdowns match baseline. | test-matrix.md, uat-checklist.md |
| **Delivery-orchestrator / SM** | Track completion; ensure no scope drift; gate “baseline complete” before closing. | Sprint/audit notes |

---

## Workstreams (parallel)

### WS-1: Backend — Document and lock sync queries (Senior / Mid)

- **Owner**: backend-squad, dev-senior, dev-mid  
- **Tasks**:
  1. For every sync route in `controller/routes/index.js`, ensure a row exists in [dar-sync-queries-baseline.md](dar-sync-queries-baseline.md).
  2. For each row: confirm SQL (or equivalent) and query params in code match the doc; fix code or doc.
  3. Add any missing endpoints (e.g. Swagger vs actual routes).
- **Output**: Baseline doc is the single source of truth; code and doc aligned.

---

### WS-2: Payload capture — All sync endpoints (Juniors + Mid)

- **Owners**: dev-junior, qa-junior (capture); dev-mid, qa-mid (review)  
- **Tasks**:
  1. **Juniors**: For each endpoint in the baseline doc, call `GET <baseUrl><path>` (and key query param variants, e.g. `?opcode=25`, `?icode=RP13`) and save response in a shared baseline file (e.g. `sprints/Sprint-Jay-Feedback/baseline-api-responses.json` or a dedicated `dar-baseline-payloads.json`).
  2. **Juniors**: Record count (e.g. `activity: 317`, `incentive: 5`, `incentive?opcode=25: 1`).
  3. **Mid**: Review payloads; confirm they match baseline doc and [baseline-expected-results.md](sprints/Sprint-Jay-Feedback/baseline-expected-results.md); flag mismatches to backend.
- **Output**: One JSON (or set of files) with all sync payloads/counts; reviewed and signed off by mid.

---

### WS-3: Frontend mapping — Endpoint → screen/dropdown (Mid + Juniors)

- **Owners**: frontend-squad, dev-mid (design); dev-junior (fill table)  
- **Tasks**:
  1. **Mid**: Define table columns: Endpoint, Query params (if any), Screen/Form, Dropdown/List name, Repository/Provider used.
  2. **Juniors**: From `main_dar_app` (Config endpoints + repo usage), fill one row per usage (e.g. GenServ incentive → `GET /api/incentive`, no opcode; Sucker Pruning → `GET /api/incentive?icode=RP13`).
  3. **Mid**: Review; add table to baseline doc or to a linked “Frontend sync mapping” section.
- **Output**: Endpoint → screen/dropdown map as baseline for “what the frontend expects.”

---

### WS-4: Validation and UAT — DB vs requirements vs frontend (QA + Mid)

- **Owners**: qa-senior, qa-mid, dev-mid  
- **Tasks**:
  1. **QA-mid**: For each dropdown in UAT checklist, note expected count/source from baseline doc and baseline payloads.
  2. **QA-junior**: Execute UAT: open each form, compare dropdown count to baseline; log pass/fail.
  3. **QA-senior**: Sign off when all critical dropdowns match baseline; escalate mismatches (DB vs requirements) to backend/PO.
- **Output**: UAT checklist completed; baseline-expected-results and test-matrix updated; any bugs filed.

---

## Scaling with more juniors and mid-levels

- **Juniors**: WS-2 (payload capture, count recording), WS-3 (filling endpoint → screen table from codebase search).
- **Mid**: WS-1 (backend doc alignment with dev-senior), WS-2 (review payloads), WS-3 (design table, review), WS-4 (validation, UAT).
- **Senior**: WS-1 (final baseline doc and SQL alignment), WS-4 (sign-off and escalation).

Split WS-2 by endpoint groups (e.g. Junior A: activity, incentive, location, disease; Junior B: materials, fertilizer-*, chem-mixing; Junior C: fruit-care, other-operations, etc.). Mid reviews each group.

---

## Definition of done (full course)

- [ ] [dar-sync-queries-baseline.md](dar-sync-queries-baseline.md) lists every sync endpoint with SQL/params and is aligned with code.
- [ ] All sync endpoints have at least one captured payload (and key variants) in a shared baseline JSON.
- [ ] Frontend mapping (endpoint → screen/dropdown) is documented and reviewed.
- [ ] UAT checklist run against baseline; dropdown counts match baseline; QA sign-off or bugs logged.
- [ ] Delivery-orchestrator / SM has marked “sync baseline full course” complete for the sprint/release.

---

## Links

- **Baseline (single source of truth)**: [dar-sync-queries-baseline.md](dar-sync-queries-baseline.md)  
- **Sprint baseline expected results**: [Sprint-Jay-Feedback/baseline-expected-results.md](sprints/Sprint-Jay-Feedback/baseline-expected-results.md)  
- **Sprint baseline payloads**: [Sprint-Jay-Feedback/baseline-api-responses.json](sprints/Sprint-Jay-Feedback/baseline-api-responses.json)  
- **Orchestrator**: [agents/delivery-orchestrator.md](../agents/delivery-orchestrator.md)
