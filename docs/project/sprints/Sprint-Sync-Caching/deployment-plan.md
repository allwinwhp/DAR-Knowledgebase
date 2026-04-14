# Deployment Plan — Sync Caching

**[Senior DevOps]** | Sprint-Sync-Caching | 2026-04-15

---

## Pre-Conditions

- [ ] All testing gates in `cicd-impact.md` pass
- [ ] Feature branches pushed and PRs approved
- [ ] Server access ready
- [ ] No active user submissions in progress

---

## Deployment Sequence

**Order: Backend first → verify → Frontend second.**
No database step — zero schema changes.

---

### Step 1: Backend Deployment

#### 1.1 Pre-deployment snapshot

```bash
cd DAR_Middleware
git log -1 --oneline   # Save SHA for rollback
pm2 status dar-middleware
```

#### 1.2 Pull and install

```bash
git fetch origin && git checkout dev && git pull origin dev
npm install
npm ls node-cache   # Verify installed
```

#### 1.3 Restart

```bash
pm2 restart dar-middleware
```

#### 1.4 Post-restart verification

| # | Check | Expected |
|---|-------|----------|
| 1 | Process running | `online` |
| 2 | Health endpoint | 200 OK |
| 3 | First sync call | X-Cache: MISS |
| 4 | Repeat sync call | X-Cache: HIT |
| 5 | Response body identical | MISS and HIT match |
| 6 | POST endpoints work | Normal (not cached) |
| 7 | Startup logs clean | No errors |
| 8 | Memory baseline | Note RSS value |

---

### Step 2: Frontend Deployment

#### 2.1 Build

```bash
cd main_dar_app
flutter build apk --release
```

#### 2.2 Distribute

Rename APK: `DAR APP - SyncCaching.apk`. Distribute via existing channel.

#### 2.3 On-device verification

| # | Check | Expected |
|---|-------|----------|
| 1 | App opens | Login screen loads |
| 2 | Login works | Home screen loads |
| 3 | Sync completes | All data loads; faster |
| 4 | Repeat sync | Faster (cache warm) |
| 5 | Data integrity | All dropdowns populated |
| 6 | Report submission | Queue + send works |

---

## Backward Compatibility

| Scenario | Behavior | Acceptable? |
|----------|----------|-------------|
| Old app + new backend | Gets cached responses; ignores X-Cache header | Yes |
| New app + old backend | Parallel sync works; no cache benefit | Yes |
| New app + new backend | Full benefit | Yes |

---

## Rollback

### Backend (~2 minutes)

```bash
pm2 stop dar-middleware
git checkout <saved-sha>
npm install
pm2 start dar-middleware
```

### Frontend

Redistribute previous APK.

No data cleanup required. Cache evaporates on restart.

---

## Post-Deployment Monitoring (First 24 Hours)

| # | What | How | Threshold |
|---|------|-----|-----------|
| 1 | X-Cache header ratio | Spot-check with curl | HIT >50% after warm-up |
| 2 | Node process memory | pm2 monit | No growth beyond 2x baseline |
| 3 | Response times | Server logs | Cached <50ms |
| 4 | Error rate | pm2 logs | Zero new error types |
| 5 | Stale data complaints | User feedback | Adjust TTLs if reported |

---

## Sign-Off

| Role | Sign-off | Date |
|------|----------|------|
| DevOps | | |
| QA Lead | | |
| PM | | |
| SM | | |
