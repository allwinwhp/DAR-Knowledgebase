# References — Sprint-Sync-Caching

---

## Project Context

- [dar-system-context.md](../../project/dar-system-context.md) — System architecture, repos, branch strategy
- [dar-discovery-summary.md](../../project/dar-discovery-summary.md) — Discovery summary
- [dar-sync-queries-baseline.md](../../project/dar-sync-queries-baseline.md) — Sync queries baseline
- [dar-sync-baseline-team-assignment.md](../../project/dar-sync-baseline-team-assignment.md) — Sync baseline team assignment

## Prior Sprints

- [Sprint-Jay-Feedback](../Sprint-Jay-Feedback/sprint-plan.md) — Jay feedback sprint
- [Sprint-Submission-Queue](../Sprint-Submission-Queue/sprint-plan.md) — Submission queue sprint (predecessor)

## Agents

- [delivery-orchestrator](../../../agents/delivery-orchestrator.md) — DAR delivery context
- [AGENTS.md](../../../AGENTS.md) — Full agent index

## Skills

- [sprint-planning-facilitator](../../../skills/sprint-planning-facilitator/SKILL.md) — This sprint was planned using this skill

## Backend Codebase

- `DAR_Middleware/controller/data_sync/` — All sync endpoint files
- `DAR_Middleware/controller/routes/index.js` — Route registry

## Frontend Codebase

- `main_dar_app/lib/app/screen/home/widgets/sync/home_sync.dart` — Sync orchestration
- `main_dar_app/lib/app/data/sync/` — Sync repositories
- `main_dar_app/lib/app/services/sync_dialog_service.dart` — Progress dialog
