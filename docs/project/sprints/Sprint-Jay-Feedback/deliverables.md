# Sprint-Jay-Feedback — Deliverables

---

## Code / config

| Deliverable | Repo | Owner |
|-------------|------|--------|
| Backend filter endpoints (costcenter, incentive) aligned to Jay queries | DAR_Middleware | Backend |
| PD Survey header fix; Gouging header fix; ERAD accomplishment fix | DAR_Middleware | Backend |
| tblDarHdr Cycle/Sqno support; Chemical Mixing operation code fix | DAR_Middleware | Backend |
| Flutter: General Services, PD Survey, Gouging, Sucker Pruning, Weedspray, CPMS, Fertilizer, Gen Services Group, Planting/Replanting, Popcount form/field changes | main_dar_app | Frontend |
| Flutter: Server selection screen and session base URL | main_dar_app | Frontend |

---

## Documentation

| Deliverable | Location |
|-------------|----------|
| code-deep-dive.md | This sprint folder |
| server-selection-feature.md | This sprint folder |
| test-matrix.md | This sprint folder |
| uat-checklist.md | This sprint folder |
| baseline-api-responses.json | This sprint folder — baseline payloads from localhost:3000 for dropdown testing |
| baseline-expected-results.md | This sprint folder — table of expected results for comparison |
| **Sync baseline (single source of truth)** | [dar-sync-queries-baseline.md](../../dar-sync-queries-baseline.md) — all sync endpoints, SQL, params; use to check DB vs requirements and frontend |
| **Full-course team assignment** | [dar-sync-baseline-team-assignment.md](../../dar-sync-baseline-team-assignment.md) — roles, workstreams, juniors/mid scaling |

---

## Definition of done (per story)

- [ ] Acceptance criteria met
- [ ] Test cases passed (per test-matrix)
- [ ] QA sign-off
- [ ] No regression on existing UAT scenarios (per dar-uat-checklist)
