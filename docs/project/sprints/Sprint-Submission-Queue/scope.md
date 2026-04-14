# Sprint-Submission-Queue — Scope

**[PO]** — Product Owner sign-off on scope, prioritization, and acceptance criteria.

**Problem**: When multiple farm supervisors press "Send to Server" simultaneously, race conditions occur on shared resources — `tblbatchno` (batch number counter read-increment-write without locking) and `tblDARbatch` (batch creation). The multi-step pipeline (batch → header → details → accomplishments → materials → totals) executes without mutual exclusion, causing duplicate batch numbers, lost writes, and corrupted report data.

**Solution**: A database-backed FIFO Submission Queue that serializes the entire send pipeline so only one user's submission runs at a time.

**Branches**: `feature/submission-queue` from `dev` (DAR_Middleware) | `feature/submission-queue` from `MDAG-331` (main_dar_app)

---

## Epics

| Epic | Description |
|------|-------------|
| **Epic-SQ** | Submission Queue — Database-backed FIFO queue that serializes DAR report submissions to eliminate race conditions on `tblbatchno` and `tblDARbatch`. Backend queue table + API + reaper; frontend enqueue-poll-proceed integration wrapping existing `sendToServerButtonClicked()`. |

---

## User stories

| ID | Title | Acceptance criteria | Deps |
|----|-------|---------------------|------|
| US-SQ01 | Supervisor enqueues when pressing Send | 1. When supervisor taps "Send to Server" (in `ReviewAction`), the app calls the enqueue API *before* starting the batch→header→details pipeline. 2. The API returns a `SessionId` and queue `Position`. 3. If the enqueue call fails, the user sees an error and no pipeline step executes. | TS-SQ01, TS-SQ02 |
| US-SQ02 | Supervisor sees queue position while waiting | 1. After enqueue, the app displays a "Waiting in queue — Position N" indicator (inside the existing progress dialog). 2. Position updates every poll cycle (recommended 2–3 s). 3. When position reaches 1 and status is `ACTIVE`, the indicator transitions to the normal "Sending report…" progress flow. | US-SQ01, TS-SQ03 |
| US-SQ03 | Only one submission pipeline runs at a time | 1. The backend enforces that exactly one session has status `ACTIVE` at any time; all others remain `WAITING`. 2. The check-turn API returns `{ yourTurn: true }` only for the session with the lowest `SessionId` in `WAITING` status (promoted to `ACTIVE`). 3. Concurrent enqueue requests from different users receive distinct, monotonically increasing `SessionId` values (IDENTITY column). | TS-SQ01, TS-SQ02 |
| US-SQ04 | Session completes and next user proceeds | 1. After the pipeline finishes successfully, the app calls the complete API, setting the session to `COMPLETED`. 2. The next `WAITING` session is then eligible to become `ACTIVE` on its next poll. 3. If the pipeline fails, the app calls the fail API, setting the session to `FAILED`, and the next user proceeds. 4. The supervisor sees a clear success or failure message. | US-SQ01, US-SQ03, TS-SQ02 |
| US-SQ05 | Stale session expiry | 1. Sessions in `ACTIVE` status for longer than a configurable TTL (default: 120 s) are automatically reaped to `EXPIRED` by the backend. 2. Sessions in `WAITING` status for longer than a configurable TTL (default: 300 s) are reaped to `EXPIRED`. 3. Reaping unblocks the queue — the next `WAITING` session can proceed. 4. The reaper runs on a configurable interval (default: 30 s). | TS-SQ01, TS-SQ02 |
| US-SQ06 | Idempotent enqueue (double-tap protection) | 1. If the same `userId` calls enqueue while they already have a `WAITING` or `ACTIVE` session, the API returns the existing session instead of creating a duplicate. 2. The frontend disables the "Send to Server" button immediately on tap to prevent rapid double-taps. | US-SQ01, TS-SQ02 |
| US-SQ07 | Queue timeout on frontend | 1. The app enforces a maximum wait time (configurable, default: 180 s). 2. If the wait exceeds the timeout, the app cancels the session (calls fail API with reason `CLIENT_TIMEOUT`), shows a "Timed out waiting in queue — please try again" message, and re-enables the Send button. 3. The user's unsent reports remain intact locally in Hive. | US-SQ01, US-SQ02, TS-SQ03 |

---

## Technical stories

| ID | Title | Acceptance criteria |
|----|-------|---------------------|
| TS-SQ01 | Create `tbl_SubmissionQueue` table (migration) | 1. Table created in MSSQL `DAR` database with columns: `SessionId` (INT IDENTITY PK), `UserId` (INT NOT NULL), `Username` (VARCHAR), `Status` (VARCHAR(20) NOT NULL — values: `WAITING`, `ACTIVE`, `COMPLETED`, `FAILED`, `EXPIRED`), `EnqueuedAt` (DATETIME DEFAULT GETDATE()), `ActivatedAt` (DATETIME NULL), `CompletedAt` (DATETIME NULL), `FailReason` (VARCHAR(255) NULL). 2. Index on `(Status, SessionId)` for efficient queue ordering queries. 3. Index on `(UserId, Status)` for idempotent enqueue lookups. 4. Migration is idempotent (safe to re-run). |
| TS-SQ02 | Create queue API endpoints | 1. **POST `/queue/enqueue`** — accepts `{ userId, username }`, creates a `WAITING` row (or returns existing per US-SQ06), returns `{ sessionId, position }`. 2. **POST `/queue/check-turn`** — accepts `{ sessionId }`, returns `{ yourTurn: boolean, position: number, status: string }`. If the session is the lowest `WAITING` and no `ACTIVE` session exists, promotes it to `ACTIVE` and sets `ActivatedAt`. 3. **POST `/queue/complete`** — accepts `{ sessionId }`, sets status to `COMPLETED` and `CompletedAt`. 4. **POST `/queue/fail`** — accepts `{ sessionId, reason }`, sets status to `FAILED` and `CompletedAt`, stores `FailReason`. 5. All state transitions use a serializable transaction or row-level locking to prevent races. 6. Routes mounted under `/queue/*` in the Express app. |
| TS-SQ03 | Frontend queue service layer (API + polling orchestrator) | 1. New Dart service class `SubmissionQueueService` with methods: `enqueue(userId, username)`, `checkTurn(sessionId)`, `complete(sessionId)`, `fail(sessionId, reason)`. 2. New `SubmissionQueueOrchestrator` that wraps the existing `sendToServerButtonClicked()` flow: enqueue → poll `checkTurn` at interval → on `yourTurn` run the existing pipeline → complete/fail. 3. Polling interval configurable (default: 2 s). 4. Service uses the same base URL / HTTP client as existing API calls. |
| TS-SQ04 | Wrap `ReviewAction.sendToServerButtonClicked()` with queue logic | 1. `ReviewAction.sendToServerButtonClicked()` in `lib/app/screen/review/widgets/action/review_action.dart` calls `SubmissionQueueOrchestrator` instead of directly iterating dialog services. 2. The orchestrator enqueues, waits for turn, then calls the existing individual/group report loop (dialog services are unchanged). 3. On completion, calls `complete`; on any error, calls `fail`. 4. The progress dialog shows queue position while waiting (US-SQ02) and then transitions to the standard step-by-step progress (BATCH OK → HEADER OK → etc.). 5. Existing `home_send.dart` navigation to the Review screen is unchanged. |
| TS-SQ05 | Backend stale-session reaper | 1. A `setInterval`-based reaper runs on server startup (configurable interval, default: 30 s). 2. Reaper updates `ACTIVE` sessions older than TTL to `EXPIRED`. 3. Reaper updates `WAITING` sessions older than TTL to `EXPIRED`. 4. Reaper logs each expiration with `SessionId`, `UserId`, and age. 5. TTL values configurable via environment variables (`SQ_ACTIVE_TTL_MS`, `SQ_WAITING_TTL_MS`, `SQ_REAPER_INTERVAL_MS`). |

---

## Dependencies

```
TS-SQ01 (table)
  └── TS-SQ02 (API endpoints) ── requires table to exist
       ├── TS-SQ05 (reaper) ── requires queue table + status conventions
       └── TS-SQ03 (frontend service) ── requires API endpoints to call
            └── TS-SQ04 (wrap ReviewAction) ── requires frontend service
                 ├── US-SQ01 (enqueue on Send) ── requires TS-SQ04
                 ├── US-SQ02 (queue position UI) ── requires TS-SQ03, TS-SQ04
                 ├── US-SQ03 (one-at-a-time) ── requires TS-SQ02
                 ├── US-SQ04 (complete/fail) ── requires TS-SQ02, TS-SQ04
                 ├── US-SQ05 (stale expiry) ── requires TS-SQ05
                 ├── US-SQ06 (idempotent) ── requires TS-SQ02
                 └── US-SQ07 (frontend timeout) ── requires TS-SQ03, TS-SQ04
```

**Critical path**: TS-SQ01 → TS-SQ02 → TS-SQ03 → TS-SQ04 (backend-first, then frontend integration).

**Parallel work**: TS-SQ05 (reaper) can be developed in parallel with TS-SQ03/TS-SQ04 once TS-SQ02 is done.

---

## Out of scope

- Refactoring `tblbatchno` or `tblDARbatch` internals — the queue eliminates the race condition without modifying existing submission logic.
- Database-level row locking on `tblbatchno` — the queue approach is chosen over per-table locking to avoid modifying every operation module's `tblDARhdr.js`.
- Multi-server / distributed queue — DAR runs on a single middleware instance; the queue table provides sufficient serialization.
- Offline queue — reports are already stored locally in Hive; this feature only governs the server submission pipeline.

---

## Risks

| Risk | Mitigation |
|------|------------|
| Reaper TTL too aggressive — kills legitimate long-running submissions | Default ACTIVE TTL of 120 s is 4× the typical pipeline duration (~30 s). Monitor logs and tune. |
| Poll interval too frequent — unnecessary server load | 2 s default; configurable. With typical queue depths of 2–5 users, this is ~2.5 req/s at peak. |
| Session stuck in `ACTIVE` if app crashes mid-pipeline | Reaper handles this; session expires after TTL and next user proceeds. |
| Frontend timeout fires during a legitimately slow pipeline | Timeout (180 s) is separate from active pipeline; timer resets (or stops) once `yourTurn` is received and pipeline starts. |

---

*[PO] Scope approved. All MVP user stories cover the farm supervisor persona. Acceptance criteria are testable. No scope creep beyond the queue feature. Stories are sized for a single sprint.*
