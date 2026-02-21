# Sprint-Jay-Feedback — CI/CD Impact

**Owner**: DevOps

---

## Branch / tag strategy

| Item | Value |
|------|--------|
| Base branch | `MDAG-339` (or DEV after merge) |
| Sprint branch | `sprint/Sprint-Jay-Feedback-*` |
| Merge target | `MDAG-339` → DEV (per project); no direct push to main |
| Tag / release | Per release-playbook when release is cut |

---

## Pipeline changes

| Change | Impact |
|--------|--------|
| No new CI jobs required for this sprint | Existing pipeline applies |
| Backend: DAR_Middleware | Run existing tests (if any) on sprint branch |
| Frontend: main_dar_app | Flutter build/test on sprint branch |

---

## Gates

| Gate | When | Owner |
|------|------|--------|
| Unit tests pass | Before task complete (TDD) | Dev |
| Peer review | Before story complete | Dev-Senior / peer |
| QA sign-off | Before story done | QA Lead |
| UAT (stakeholder manual) | Before release sign-off | PM + stakeholder |
| Playwright (if applicable) | Per deployment strategy; DAR may use manual UAT only | QA Lead |

---

## Playwright audit

- **DAR context**: Playwright E2E may not be in place for Flutter app; manual UAT and test-matrix drive validation.
- If Playwright is introduced later, gates will be added per deployment-strategy.
