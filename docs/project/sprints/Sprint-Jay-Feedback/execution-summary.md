# Sprint-Jay-Feedback — Execution Summary

**Purpose**: Document scope implemented, deviations, technical decisions, risks, and test coverage per execution principles.

---

## 1. Scope implemented (this execution run)

| Category | Implemented | Notes |
|----------|-------------|--------|
| **BE-1 (US-J1)** | Yes | Costcenter: getActivity already used `Active = 1`. Incentive: added `Doctype NOT IN ('03','01')` and optional `icode` in getIncentive.js. |
| **FE Sucker Pruning (US-J4)** | Partial | Incentive: use Icode RP13 only — added `getIncentivesByIcode('RP13')` and Sucker Pruning form uses it. Cycle/Sqno and backend persistence: not yet implemented. |
| **TS-SERVER** | Yes | Server selection: dropdown on login screen; Config.baseURL set on select; persisted via SharedPreferences; list in Config.serverOptions. |
| **Sprint Package** | Yes | risk-log.md, carryover.md, cicd-impact.md, deployment-plan.md, uat-checklist.md, test-matrix.md (expanded + Test Guide). |
| **Baseline for testing** | Yes | baseline-api-responses.json and baseline-expected-results.md: API payloads captured from localhost:3000; table of expected results for dropdown comparison per Jay SQL. |
| **BE-4 (US-J4) backend** | Partial | pseudostem-cgms/tblDARhdr.js accepts Cycle, Sqno from request body (DB columns must exist; migration if needed). |
| **BE-2, BE-3, BE-5–BE-8** | No | Survey/Gouging/ERAD header and accomplishment fixes, Chemical Mixing opcode, Gen Services Group, Popcount HAS — not implemented. |
| **FE-1–FE-3, FE-4–FE-9** | No | PD Survey material max 5, Gouging incentive Opcode 25 UI, Sucker Pruning Cycle/Sqno fields in form, Weedspray, CPMS decimal, Fertilizer, Gen Services Group, Planting/Replanting, Popcount — not implemented. |

---

## 2. Deviations from plan

- **Execution run scope**: This run focused on (1) completing Sprint Package artifacts, (2) BE-1 incentive filter and Sucker Pruning incentive (RP13), (3) TS-SERVER server selection. Remaining backend and frontend tasks (header fixes, Cycle/Sqno, other forms) are left for continued implementation.
- **No unit test automation added**: Backend (Node) and frontend (Flutter) do not yet have automated unit tests for the changed code in this run; TDD would require adding tests first in a follow-up.
- **Gouging incentive**: Backend supports `opcode=25`; frontend already uses Opcode filters (e.g. Gouging uses getIncentivesByOpcode('09') in current code — may need alignment to Opcode '25' per Jay feedback).

---

## 3. Technical decisions

| Decision | Rationale |
|----------|-----------|
| Incentive API: `whereNotIn('Doctype', ['03','01'])` | Matches Jay intake: exclude Harvest (03) and Fruit care (01) for General Services and applicable operations. |
| Incentive API: optional `icode` query param | Sucker Pruning needs Icode 'RP13' only; backend can return filtered set; app syncs all (Doctype filtered) and filters by Icode locally. |
| Sucker Pruning: `getIncentivesByIcode('RP13')` | Use Icode filter per US-J4; repository filters from already-synced incentives (Doctype filter from API). |
| Server selection on login screen | Single screen with dropdown above credentials; no extra route; Config.setBaseUrl() + SharedPreferences for “remember last server”. |
| Server list in Config | Hardcoded `serverOptions` (UAT ngrok, Local emulator, localhost); can be moved to asset later. |

---

## 4. Risks identified

- **R1–R3** (Survey/Gouging header, ERAD accomplishments, Chemical Mixing opcode): Still open; backend investigation and fixes pending.
- **R4** (Weedspray timekeeping): Product decision pending; then FE-4.
- **R5** (Server selection): Mitigated by single Config.baseURL and persistence.

---

## 5. Test coverage summary

| Area | Unit tests | Manual / UAT |
|------|------------|---------------|
| BE-1 (activity/incentive API) | None added this run | Covered by test-matrix TC-J1-1, TC-J1-2 and UAT checklist §2. |
| Sucker Pruning incentive RP13 | None added this run | TC-J4-2; UAT §5. |
| Server selection | None added this run | TC-SRV-1, TC-SRV-2, TC-SRV-3; UAT §1. |

Test matrix and Test Guide in test-matrix.md provide scenarios, steps, expected results, edge cases, and regression checklist for manual review.

---

## 6. Mapping to sprint plan

- **sprint-plan.md**: Goal (Jay findings + server selection) and deliverables — Sprint Package artifacts produced; partial implementation as above.
- **scope.md**: US-J1, US-J4 (partial), TS-SERVER addressed; others remain in backlog for next steps.
- **tasks-by-agent.md**: BE-1 done; FE-10 (TS-SERVER) done; FE-3 (Sucker Pruning incentive only); QA-1, QA-2, QA-3 supported by test-matrix and uat-checklist.
- **delivery-orchestrator / sprint-execution**: Execution sequence followed (artifacts first, then implementation); governance (QA test strategy before dev) respected via existing test-matrix; TDD not yet fulfilled by automated tests — manual and UAT cover current scope.

---

## 7. Environment

- **Backend**: DAR_Middleware; knex_config.js uses current live ngrok (0.tcp.ap.ngrok.io:10653) per environment configuration requirement.
- **Frontend**: main_dar_app; server selection uses Config.baseURL (default UAT ngrok) for consistency during manual validation.
