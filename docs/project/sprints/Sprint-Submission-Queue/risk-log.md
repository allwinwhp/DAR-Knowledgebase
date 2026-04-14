# Risk Log — Sprint-Submission-Queue

**[DevOps + PM]** | 2026-04-14

---

| # | Risk | Severity | Likelihood | Impact | Mitigation | Owner |
|---|------|----------|------------|--------|------------|-------|
| R-01 | **Dead session blocking queue** — App crashes mid-pipeline; session stays ACTIVE, blocking all others | Critical | Likely | Queue completely blocked until reaper fires | 1. `setInterval` reaper on the persistent Node process expires ACTIVE sessions after TTL. 2. Frontend calls `/queue/fail` in error handlers. 3. Frontend timeout (180s) calls fail as fallback. | Backend + QA |
| R-02 | **Network connectivity in farm environments** — Intermittent cellular/Wi-Fi | High | Likely | Supervisor stuck in queue; session may expire server-side | 1. Retry polls with backoff. 2. Retry complete/fail calls. 3. Reaper handles abandoned sessions. | Frontend |
| R-03 | **Old app version bypasses queue** — Direct `/sendDARForm/*` calls reintroduce race conditions | High | Likely | Race conditions persist for old-version users | 1. Acceptable during rollout. 2. Push field teams to update. 3. Future: require `X-Queue-Session` header. | PO + DevOps |
| R-04 | **Poll battery drain on mobile** — 3s polling keeps radio active | Medium | Possible | Battery drain on budget Android devices | 1. Configurable interval. 2. Polling only during active wait. 3. Max timeout caps duration. | Frontend |
| R-05 | **Migration not run before deploy** — Code deployed before table exists | High | Possible | All `/queue/*` endpoints return 500 | 1. Deployment plan enforces DB-first. 2. Rollback: stop the Node process and restart previous version. | DevOps |
| R-06 | **Reaper TTL too aggressive** — 120s may be too short for slow connections | Medium | Possible | Legitimate submissions expired | 1. Default 120s = 4x typical pipeline. 2. Configurable via env var. 3. Monitor logs. | Backend + DevOps |
| R-07 | **DB connection pool exhaustion** — Poll requests under peak load | Low | Unlikely | Poll requests fail with timeout | 1. Queue queries < 50ms. 2. At 5 req/s, ~0.25 connections avg. 3. Monitor if scaling past 20 users. | Backend |
| R-08 | **Physical server downtime** — Server restart or crash kills Node process, reaper stops | Medium | Possible | Active sessions orphaned until server restarts; reaper resumes and clears them | 1. Use a process manager (PM2/systemd) with auto-restart. 2. Reaper runs immediately on process start, clearing any orphaned sessions. 3. Frontend timeout handles gap. | DevOps |
| R-09 | **Server process memory leak** — `setInterval` reaper accumulating references over time | Low | Unlikely | Gradual memory growth; eventual OOM | 1. Reaper function is stateless — no accumulation. 2. PM2 memory-limit restart as safety net. 3. Monitor Node heap in production. | Backend + DevOps |

---

## Risk Heatmap

```
                Likelihood →
Severity ↓    Rare    Unlikely   Possible    Likely    Almost Certain
─────────────────────────────────────────────────────────────────────
Critical                                      R-01
High                              R-05        R-02,R-03
Medium                            R-06,R-08   R-04
Low                    R-07,R-09
```

---

## Top 3 Actions Required Before Implementation

1. **R-01**: Verify reaper + frontend error handlers + frontend timeout all correctly handle dead sessions. Test by killing the app mid-pipeline and confirming the queue unblocks within TTL.
2. **R-08**: Ensure the physical server uses PM2 or systemd to auto-restart the Node process. Reaper must run `expireStale()` once on startup to clear any sessions orphaned during downtime.
3. **R-05**: Enforce strict deployment order — DB migration before code deploy. Verify table exists before starting new backend.
