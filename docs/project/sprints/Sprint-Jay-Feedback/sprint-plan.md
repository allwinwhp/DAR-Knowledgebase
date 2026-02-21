# Sprint Plan — Jay Feedback II + Server Selection

**Sprint**: Sprint-Jay-Feedback  
**Goal**: Intake Jay Findings II (Feb 21, 2026), fix submission/filter/field issues, and add server selection before login.  
**Timeline**: *(fill: start – end)*  
**Branch**: From `MDAG-339` (or DEV after merge). Sprint branch: `sprint/Sprint-Jay-Feedback-*`.

---

## 1. Sprint goal

- **Business**: Resolve DAR mobile findings from Jay (filters, send-to-server failures, missing fields, Gang/Group timekeeping and incentives) so all reported operations send correctly and data matches business rules.
- **Feature**: Add **server selection** on the pre-login screen: user picks a server from a dropdown; that URL becomes the app base URL for the session.
- **Quality**: All changes covered by test cases; UAT scenarios updated.

---

## 2. Scope summary

| Category | Items |
|----------|--------|
| **Intake source** | [Jay feedback (DAR FINDINGS II 02212026)](../../jay%20feedback/feedback.md) → [Jay feedback intake](../../jay-feedback-intake.md) |
| **User stories** | US-J1–US-J12 (filters, submission, Cycle/Sqno, operation code, timekeeping, HAS, costcenter) |
| **Technical story** | TS-SERVER: Server selection dropdown before login; base URL for session |
| **Out of scope** | Fruit care changes (no issues reported); other operations not in Jay feedback |

---

## 3. Execution alignment

Per [delivery-orchestrator](../../../agents/delivery-orchestrator.md) (DAR context):

- **Frontend**: main_dar_app (Flutter)
- **Backend**: DAR_Middleware (Node/Express, MSSQL, Knex)
- **QA first**: Test strategy and test cases before dev implementation

---

## 4. Deliverables (mandatory)

| # | Artifact | Owner |
|---|----------|--------|
| 1 | scope.md | PO / Business |
| 2 | tasks-by-agent.md | Dev squads / SM |
| 3 | deliverables.md | Dev squads |
| 4 | test-matrix.md | QA Lead |
| 5 | code-deep-dive.md | Architect / Dev-Senior |
| 6 | server-selection-feature.md | Design Authority / Frontend |
| 7 | uat-checklist.md | QA Lead + PM |
| 8 | cicd-impact.md, deployment-plan.md, risk-log.md, carryover.md | DevOps / SM |

---

## 5. References

- [Jay feedback intake](../../jay-feedback-intake.md)
- [dar-mobile-feedback-baseline](../../dar-mobile-feedback-baseline.md)
- [dar-user-journey-and-form-specifications](../../dar-user-journey-and-form-specifications.md)
- [dar-system-context](../../dar-system-context.md)
- [delivery-orchestrator](../../../agents/delivery-orchestrator.md)
