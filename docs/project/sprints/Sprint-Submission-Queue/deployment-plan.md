# Deployment Plan — Submission Queue

**[Senior DevOps]** | Sprint-Submission-Queue | 2026-04-14

---

## Pre-Conditions

- [ ] All testing gates in `cicd-impact.md` pass
- [ ] Feature branches pushed to `origin` and PRs approved
- [ ] MSSQL `DAR` database credentials available for migration
- [ ] Physical server access (SSH / RDP) for DAR_Middleware deployment
- [ ] Process manager (PM2 or systemd) configured on the server

---

## Deployment Sequence

**Order is strict: Database → Backend → Frontend (new APK build).**

### Step 1: Database Migration

**What**: Create `tbl_SubmissionQueue` table in the `DAR` MSSQL database.
**When**: Before any code deployment.

**Option A — Knex CLI** (preferred):
```powershell
cd DAR_Middleware\db
npx knex migrate:latest --knexfile knexfile.js
```

**Option B — Raw SQL** (if Knex CLI unavailable on the server):
Run `db/sqlsetup/07_create_tbl_SubmissionQueue.sql` via SSMS or `sqlcmd`.

**Verification**:
```sql
SELECT TOP 0 * FROM tbl_SubmissionQueue;
SELECT name FROM sys.indexes WHERE object_id = OBJECT_ID('tbl_SubmissionQueue');
```

**Rollback**: `DROP TABLE IF EXISTS tbl_SubmissionQueue;` (safe — no data, no existing code references it).

---

### Step 2: Backend Deployment (DAR_Middleware — Physical Server)

**What**: Deploy the updated DAR_Middleware with `/queue/*` endpoints to the physical server.
**When**: After Step 1 verified.

**Steps**:
1. SSH/RDP into the server.
2. Navigate to the DAR_Middleware directory.
3. Pull the latest code:
   ```bash
   git fetch origin
   git checkout dev
   git pull origin dev
   ```
   (Or merge `feature/submission-queue` → `dev` first, then pull `dev`.)
4. Install any new dependencies (none expected, but safety check):
   ```bash
   npm install
   ```
5. Restart the Node process:
   ```bash
   # If using PM2:
   pm2 restart dar-middleware

   # If using systemd:
   sudo systemctl restart dar-middleware

   # If running manually (dev/staging):
   # Stop the current process, then:
   node server.js
   ```
6. Verify the reaper starts (check logs for `Stale session reaper started` or similar).

**Environment Variables** (set on the server, e.g. in `.env` or PM2 ecosystem config):

| Variable | Default | Description |
|----------|---------|-------------|
| `QUEUE_STALE_TIMEOUT_MINUTES` | `5` | Max minutes before ACTIVE session is expired by reaper |
| `PORT` | `3000` | Server port (existing) |

**Verification**:
```bash
curl http://localhost:3000/health
curl -X POST http://localhost:3000/queue/enqueue \
  -H "Content-Type: application/json" \
  -d '{"userId":"9999","supervisorId":"TEST"}'
# Expected: {"sessionId": <N>, ...}

# Clean up test session:
curl -X POST http://localhost:3000/queue/complete \
  -H "Content-Type: application/json" \
  -d '{"sessionId": <N>}'
```

**Rollback**:
1. Stop the current process.
2. Revert to previous code:
   ```bash
   git checkout <previous-commit-sha>
   npm install
   ```
3. Restart the process. The `tbl_SubmissionQueue` table can remain — old code never references it.

---

### Step 3: Frontend Deployment (main_dar_app — New APK Build)

**What**: Build a new APK with queue-aware send flow and distribute to field supervisors.
**When**: After Step 2 verified and `/queue/*` endpoints are live on the physical server.

**Steps**:
1. Merge `feature/submission-queue` → `MDAG-331`.
2. Update `Config.baseURL` in `lib/core/presentation/theme/config.dart` to point to the physical server's IP/hostname.
3. Version bump in `pubspec.yaml` (e.g. `v0.8.0` or next appropriate version).
4. Build:
   ```bash
   flutter build apk --release
   ```
5. Distribute the APK to QA / field supervisors via the existing distribution channel (direct APK install).

**Rollback**: Redistribute the previous APK (`main_dar_app-v0.7.0-build26-qa-20260222.apk`).

---

## Backward Compatibility

| Scenario | Behavior | Acceptable? |
|----------|----------|-------------|
| Old app + new backend | App calls `/sendDARForm/*` directly, bypassing queue | Yes — no worse than status quo |
| New app + old backend | App calls `/queue/enqueue` → 404 → should show error | Must handle in Flutter code |
| New app + new backend + table missing | `/queue/enqueue` → 500 | Prevented by DB-first deployment |

---

## Post-Deployment Verification Checklist

- [ ] `GET /health` returns 200
- [ ] Node process is running under PM2/systemd with auto-restart
- [ ] `setInterval` reaper is active (check logs on startup)
- [ ] `POST /queue/enqueue` returns `sessionId` and `position`
- [ ] `POST /queue/check-turn` returns `isMyTurn: true` for single waiting session
- [ ] `POST /queue/complete` sets session to COMPLETED
- [ ] Two concurrent submissions serialize correctly
- [ ] Stale session reaper fires (create old session, wait for interval, verify EXPIRED)
- [ ] Existing `/sendDARForm/batch/tblDARbatch` still works (regression)
- [ ] New APK connects to physical server correctly
- [ ] New APK shows queue position UI during wait
- [ ] New APK completes full report submission through queue
- [ ] Old APK still submits reports (bypass queue — regression)
