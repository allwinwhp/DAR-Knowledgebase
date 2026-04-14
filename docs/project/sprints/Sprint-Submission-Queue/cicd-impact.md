# CI/CD Impact Assessment — Submission Queue

**[Senior DevOps]** | Sprint-Submission-Queue | 2026-04-14

---

## 1. Impact on the Existing Express App (Physical Server)

### New Route Mount

`server.js` currently mounts four route groups. The Submission Queue adds a fifth:

```js
app.use('/queue', queueRoutes);   // 4 POST endpoints
```

Since the backend runs as a persistent Node.js process on a physical server, no routing configuration changes are needed beyond the `server.js` mount. All paths are handled by Express directly.

### Stale Session Reaper

The reaper uses `setInterval` on server startup. Since the backend runs as a **persistent long-running Node process** (not serverless), `setInterval` works reliably. The reaper will fire on schedule for the lifetime of the process. If the process restarts (PM2/systemd), the reaper re-initializes on startup and should run an immediate cleanup pass for any sessions orphaned during downtime.

### New Dependencies

No new entries needed in `package.json`. The queue feature uses existing `knex`, `mssql`, `express`, and `winston`.

---

## 2. Branch Strategy

| Repo | Feature Branch | Base Branch | Merge Target |
|------|---------------|-------------|--------------|
| DAR_Middleware | `feature/submission-queue` | `dev` | `dev` |
| main_dar_app | `feature/submission-queue` | `MDAG-331` | `MDAG-331` |

**Rules**:
1. No direct pushes to `dev` or `MDAG-331` — all changes through feature branch and reviewed PR.
2. Backend merges first — middleware code deployed to physical server before Flutter APK is built and distributed.
3. Push feature branches to origin before opening PRs: `git push -u origin feature/submission-queue`
4. Delete feature branches after merge.

---

## 3. Dependency Check

No new entries needed in `package.json` or `pubspec.yaml`. All queue code uses existing dependencies.

---

## 4. Server Deployment Model

| Aspect | Detail |
|--------|--------|
| **Runtime** | Node.js on physical server |
| **Process manager** | PM2 or systemd (recommended: PM2 for zero-downtime restart) |
| **Deploy method** | `git pull` on server + `npm install` + process restart |
| **Rollback** | `git checkout <prev-sha>` + restart; or PM2 `pm2 deploy revert` |
| **Logs** | PM2 logs (`pm2 logs dar-middleware`) or journalctl for systemd |
| **Auto-restart** | PM2 auto-restart on crash; `--max-memory-restart` as safety net |

---

## 5. Testing Gates Before Merge

### Backend Gates

| # | Gate | Method | Required |
|---|------|--------|----------|
| 1 | Migration runs cleanly on a fresh DB | `npx knex migrate:latest` against test MSSQL | Yes |
| 2 | Migration is idempotent | Run twice — second must be no-op | Yes |
| 3 | All 4 `/queue/*` endpoints return correct responses | Manual/scripted HTTP tests | Yes |
| 4 | Idempotent enqueue (US-SQ06) | POST twice same supervisorId → same sessionId | Yes |
| 5 | Serialization: only one ACTIVE at a time (US-SQ03) | Enqueue 3 → check-turn each → only first ACTIVE | Yes |
| 6 | Complete/fail unblock next session | Complete session 1 → check-turn session 2 → ACTIVE | Yes |
| 7 | Stale reaper expires correctly | Create stale ACTIVE → wait for `setInterval` tick → verify EXPIRED | Yes |
| 8 | Existing `/sendDARForm/*` still work | Regression test | Yes |
| 9 | `GET /health` returns 200 | Smoke test | Yes |

### Frontend Gates

| # | Gate | Method | Required |
|---|------|--------|----------|
| 1 | App builds without errors | `flutter build apk --debug` | Yes |
| 2 | Queue flow: enqueue → wait → proceed → complete | E2E against running middleware on server | Yes |
| 3 | Queue timeout fires correctly | Short timeout → verify error message | Yes |
| 4 | Queue position UI updates | 2 sessions → position display → updates | Yes |
| 5 | Double-tap protection | Rapid tap → only one enqueue | Yes |
| 6 | Old app version bypass is safe | Build without queue → `/sendDARForm/*` still works | Yes |
