# Backend Tasks — Submission Queue

**Feature**: Submission Queue  
**Branch**: `feature/submission-queue` (from `dev`)  
**Squad**: Backend  
**Stack**: Node.js / Express / Knex / MSSQL  

---

## Task Index

| ID | Task | Files | Effort | Depends On |
|----|------|-------|--------|------------|
| BE-SQ01 | Knex migration — `tbl_SubmissionQueue` | `db/migrations/20260414_create_tbl_SubmissionQueue.js` | S | — |
| BE-SQ02 | Raw SQL setup script | `db/sqlsetup/07_create_tbl_SubmissionQueue.sql`, `db/sqlsetup/00_run_all_setup.sql` | S | — |
| BE-SQ03 | Expire-stale shared function | `queue/expire-stale.js` | S | BE-SQ01 |
| BE-SQ04 | Enqueue endpoint | `queue/enqueue.js` | M | BE-SQ01 |
| BE-SQ05 | Check-turn endpoint (critical locking) | `queue/check-turn.js` | L | BE-SQ01, BE-SQ03 |
| BE-SQ06 | Complete endpoint | `queue/complete.js` | S | BE-SQ01 |
| BE-SQ07 | Fail endpoint | `queue/fail.js` | S | BE-SQ01 |
| BE-SQ08 | Queue router & mount in server.js | `queue/routes/index.js`, `server.js`, `swagger.js` | S | BE-SQ04–07 |
| BE-SQ09 | Swagger JSDoc annotations | All `queue/*.js` route files | M | BE-SQ04–07 |
| BE-SQ10 | Integration tests | `tests/queue/*.test.js` | L | BE-SQ04–08 |

---

## BE-SQ01 — Knex Migration

**File**: `db/migrations/20260414_create_tbl_SubmissionQueue.js`  
**Effort**: S  
**Dependencies**: None  

### Description

Create the `tbl_SubmissionQueue` table using the same `hasTable` guard pattern used in the existing migration `20251205_create_tblsupervisor_operators.js`.

### Schema

| Column | Type | Constraints |
|--------|------|-------------|
| `SessionId` | INT IDENTITY(1,1) | PRIMARY KEY |
| `UserId` | NVARCHAR(50) | NOT NULL |
| `SupervisorId` | NVARCHAR(50) | NOT NULL |
| `Status` | NVARCHAR(20) | NOT NULL, DEFAULT `'WAITING'` |
| `CreatedAt` | DATETIME | NOT NULL, DEFAULT `GETDATE()` |
| `ActivatedAt` | DATETIME | NULL |
| `CompletedAt` | DATETIME | NULL |
| `ReportCount` | INT | DEFAULT `0` |
| `DeviceInfo` | NVARCHAR(200) | NULL |

**Index**: Composite non-clustered on `(Status, SessionId)`.

### Implementation Notes

- Use `table.increments('SessionId').primary()` for the IDENTITY column.
- `table.string('Status', 20).notNullable().defaultTo('WAITING')` for the Status column.
- Use `table.datetime('CreatedAt').notNullable().defaultTo(knex.fn.now())` to map to `GETDATE()`.
- Add index via `table.index(['Status', 'SessionId'], 'IX_SubmissionQueue_Status_SessionId')`.
- `exports.down` must drop the table inside a `hasTable` guard.

### Skeleton

```js
exports.up = async function (knex) {
  const exists = await knex.schema.hasTable('tbl_SubmissionQueue');
  if (!exists) {
    return knex.schema.createTable('tbl_SubmissionQueue', (table) => {
      table.increments('SessionId').primary();
      table.string('UserId', 50).notNullable();
      table.string('SupervisorId', 50).notNullable();
      table.string('Status', 20).notNullable().defaultTo('WAITING');
      table.datetime('CreatedAt').notNullable().defaultTo(knex.fn.now());
      table.datetime('ActivatedAt').nullable();
      table.datetime('CompletedAt').nullable();
      table.integer('ReportCount').defaultTo(0);
      table.string('DeviceInfo', 200).nullable();
      table.index(['Status', 'SessionId'], 'IX_SubmissionQueue_Status_SessionId');
    });
  }
};

exports.down = async function (knex) {
  const exists = await knex.schema.hasTable('tbl_SubmissionQueue');
  if (exists) {
    return knex.schema.dropTable('tbl_SubmissionQueue');
  }
};
```

---

## BE-SQ02 — Raw SQL Setup Script

**Files**: `db/sqlsetup/07_create_tbl_SubmissionQueue.sql`, `db/sqlsetup/00_run_all_setup.sql`  
**Effort**: S  
**Dependencies**: None  

### Description

Mirror the migration as a raw SQL script following the `05_create_tbl_SyncLog.sql` pattern: `IF NOT EXISTS` guard, `CREATE TABLE`, separate `CREATE INDEX` blocks with their own `IF NOT EXISTS` guards.

### Implementation Notes

- Use the standard header comment block with `=============================================`.
- After creating the table, add the composite index `IX_SubmissionQueue_Status_SessionId` on `(Status, SessionId)`.
- Add a secondary index `IX_SubmissionQueue_SupervisorId` on `(SupervisorId)` — this supports the idempotency lookup in `enqueue`.
- Append the `:r` include to `00_run_all_setup.sql`:
  ```sql
  PRINT '';
  PRINT '7. Creating tbl_SubmissionQueue...';
  :r "07_create_tbl_SubmissionQueue.sql"
  GO
  ```
- Place this block before the final `Setup Completed` print section.

### Full SQL

```sql
-- =============================================
-- Create tbl_SubmissionQueue Table
-- =============================================
-- Description: Submission queue for serializing DAR form submissions
-- Used for: Preventing concurrent writes per supervisor
-- =============================================

IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'tbl_SubmissionQueue')
BEGIN
    CREATE TABLE tbl_SubmissionQueue (
        SessionId       INT IDENTITY(1,1) PRIMARY KEY,
        UserId          NVARCHAR(50)  NOT NULL,
        SupervisorId    NVARCHAR(50)  NOT NULL,
        Status          NVARCHAR(20)  NOT NULL DEFAULT 'WAITING',
        CreatedAt       DATETIME      NOT NULL DEFAULT GETDATE(),
        ActivatedAt     DATETIME      NULL,
        CompletedAt     DATETIME      NULL,
        ReportCount     INT           DEFAULT 0,
        DeviceInfo      NVARCHAR(200) NULL
    );

    PRINT 'Created table tbl_SubmissionQueue';
END
ELSE
BEGIN
    PRINT 'Table tbl_SubmissionQueue already exists';
END
GO

IF EXISTS (SELECT * FROM sys.tables WHERE name = 'tbl_SubmissionQueue')
BEGIN
    IF NOT EXISTS (SELECT * FROM sys.indexes
                   WHERE name = 'IX_SubmissionQueue_Status_SessionId'
                     AND object_id = OBJECT_ID('tbl_SubmissionQueue'))
    BEGIN
        CREATE INDEX IX_SubmissionQueue_Status_SessionId
            ON tbl_SubmissionQueue(Status, SessionId);
        PRINT 'Created index IX_SubmissionQueue_Status_SessionId';
    END

    IF NOT EXISTS (SELECT * FROM sys.indexes
                   WHERE name = 'IX_SubmissionQueue_SupervisorId'
                     AND object_id = OBJECT_ID('tbl_SubmissionQueue'))
    BEGIN
        CREATE INDEX IX_SubmissionQueue_SupervisorId
            ON tbl_SubmissionQueue(SupervisorId);
        PRINT 'Created index IX_SubmissionQueue_SupervisorId';
    END
END
GO

PRINT 'tbl_SubmissionQueue setup completed';
GO
```

---

## BE-SQ03 — Expire-Stale Shared Function

**File**: `queue/expire-stale.js`  
**Effort**: S  
**Dependencies**: BE-SQ01  

### Description

Shared utility that expires ACTIVE sessions older than a configurable timeout. Used in two ways:

1. **`setInterval` background timer** — Called on a configurable interval (default 30s) from `server.js` startup. Since we deploy on a physical server with a persistent Node.js process, `setInterval` is reliable. On startup, an immediate call clears any sessions orphaned during downtime.
2. **Inline in `check-turn`** — Called before evaluating the queue as a belt-and-suspenders approach.

### Interface

```js
/**
 * Expire ACTIVE sessions whose ActivatedAt is older than `timeoutMinutes`.
 * @param {object} db  - Knex instance
 * @param {number} [timeoutMinutes=30] - Stale threshold
 * @returns {Promise<number>} Number of rows expired
 */
async function expireStale(db, timeoutMinutes = 30) { ... }
```

### Implementation Notes

- Use `db.raw()` for the `DATEDIFF` function since Knex's query builder doesn't natively support it.
- Raw SQL:
  ```sql
  UPDATE tbl_SubmissionQueue
  SET    Status = 'EXPIRED',
         CompletedAt = GETDATE()
  WHERE  Status = 'ACTIVE'
    AND  DATEDIFF(MINUTE, ActivatedAt, GETDATE()) > @timeout
  ```
- Return the number of affected rows from the raw result (available via `result.rowCount` or `result[0]` depending on the mssql driver — verify at implementation time).
- Log expired count via `logger.info` if > 0.
- The timeout value should be sourced from `process.env.QUEUE_STALE_TIMEOUT_MINUTES` falling back to the parameter default (30).

---

## BE-SQ04 — Enqueue Endpoint

**File**: `queue/enqueue.js`  
**Effort**: M  
**Dependencies**: BE-SQ01  

### Description

`POST /queue/enqueue` — Add a supervisor to the submission queue. **Idempotent**: if the supervisor already has a WAITING or ACTIVE session, return the existing session instead of creating a new one.

### Request Body

```json
{
  "userId": "string",
  "supervisorId": "string",
  "deviceInfo": "string (optional)"
}
```

### Response (200)

```json
{
  "sessionId": 42,
  "status": "WAITING",
  "position": 3,
  "isNew": true
}
```

### Logic

1. Validate required fields (`userId`, `supervisorId`). Return `400` if missing.
2. Query for an existing session:
   ```sql
   SELECT SessionId, Status
   FROM   tbl_SubmissionQueue
   WHERE  SupervisorId = @supervisorId
     AND  Status IN ('WAITING', 'ACTIVE')
   ```
3. If found → compute position and return with `isNew: false`.
4. If not found → INSERT using `insertWithId(db, 'tbl_SubmissionQueue', 'SessionId', data)`.
5. Compute position:
   ```sql
   SELECT COUNT(*) AS position
   FROM   tbl_SubmissionQueue
   WHERE  Status IN ('WAITING', 'ACTIVE')
     AND  SessionId <= @sessionId
   ```
6. Return `200` with `isNew: true`.

### Implementation Notes

- Use the project's `insertWithId` helper from `db/insertWithId.js` — it handles the `OUTPUT INTO` pattern required for IDENTITY columns on tables that may have triggers in the future.
- Follow the error handling pattern from `validateSubmission.js`: catch block returns `500` with conditional detail.
- Log at `info` level on successful enqueue, `warn` level on idempotent hit.

---

## BE-SQ05 — Check-Turn Endpoint (Critical Locking)

**File**: `queue/check-turn.js`  
**Effort**: L  
**Dependencies**: BE-SQ01, BE-SQ03  

### Description

`POST /queue/check-turn` — Determine whether a session is next in line and, if so, atomically activate it. This is the critical concurrency path and **must** use SQL Server row-level locking to prevent two sessions from being activated simultaneously.

### Request Body

```json
{
  "sessionId": 42,
  "supervisorId": "string"
}
```

### Response (200)

```json
{
  "isMyTurn": true,
  "position": 1,
  "activeSessionId": 42,
  "status": "ACTIVE"
}
```

### Logic

1. Validate required fields. Return `400` if missing.
2. Call `expireStale(db)` to clean up stale ACTIVE sessions.
3. Execute the **atomic activation query** (see below).
4. Query caller's session to determine current status and position.
5. Return result.

### Critical Locking Query — Atomic Activation

This is the most important piece of the feature. It must run inside an explicit transaction with `UPDLOCK, HOLDLOCK` to serialize access.

```sql
BEGIN TRANSACTION;

-- Step 1: Check if there is already an ACTIVE session.
-- If so, no activation happens — the current active session holds the lock.
IF NOT EXISTS (
    SELECT 1
    FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
    WHERE  Status = 'ACTIVE'
)
BEGIN
    -- Step 2: Activate the oldest WAITING session (lowest SessionId).
    -- The UPDLOCK on the SELECT above prevents another connection from
    -- passing the same IF NOT EXISTS check concurrently.
    UPDATE tbl_SubmissionQueue
    SET    Status      = 'ACTIVE',
           ActivatedAt = GETDATE()
    WHERE  SessionId = (
        SELECT TOP 1 SessionId
        FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
        WHERE  Status = 'WAITING'
        ORDER BY SessionId ASC
    );
END

COMMIT TRANSACTION;
```

#### Why UPDLOCK + HOLDLOCK?

| Hint | Purpose |
|------|---------|
| `UPDLOCK` | Acquires an update lock on the rows read, preventing other transactions from acquiring the same lock. This serializes the "check if anyone is ACTIVE" step. |
| `HOLDLOCK` | Holds the lock until the end of the transaction (equivalent to `SERIALIZABLE` isolation on these rows). Without it, the lock could be released after the SELECT, opening a race window. |

Together they guarantee that between the `IF NOT EXISTS` check and the `UPDATE`, no other connection can insert or update a row to `ACTIVE`.

### Implementation via Knex `db.raw()`

```js
const activationSQL = `
  BEGIN TRANSACTION;

  IF NOT EXISTS (
      SELECT 1
      FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
      WHERE  Status = 'ACTIVE'
  )
  BEGIN
      UPDATE tbl_SubmissionQueue
      SET    Status      = 'ACTIVE',
             ActivatedAt = GETDATE()
      WHERE  SessionId = (
          SELECT TOP 1 SessionId
          FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
          WHERE  Status = 'WAITING'
          ORDER BY SessionId ASC
      );
  END

  COMMIT TRANSACTION;
`;

await db.raw(activationSQL);
```

After the activation query completes, a **follow-up read** determines the caller's status:

```js
const session = await db('tbl_SubmissionQueue')
  .select('SessionId', 'Status', 'ActivatedAt')
  .where({ SessionId: sessionId })
  .first();

const position = await db('tbl_SubmissionQueue')
  .where('Status', 'IN', ['WAITING', 'ACTIVE'])
  .andWhere('SessionId', '<=', sessionId)
  .count('* as position')
  .first();
```

### Response Mapping

| `session.Status` | `isMyTurn` | Notes |
|-------------------|-----------|-------|
| `ACTIVE` | `true` | This session was just activated (or was already ACTIVE). |
| `WAITING` | `false` | Another session is ACTIVE; caller must poll. |
| `COMPLETED` / `FAILED` / `EXPIRED` | — | Return `410 Gone` with a descriptive message. |
| Not found | — | Return `404`. |

### Implementation Notes

- The activation query and the follow-up reads should NOT share the same explicit transaction; the activation transaction must commit first so the follow-up read sees the updated state.
- If `db.raw()` throws, wrap in try/catch, log with `logger.error`, and return `500`.
- The mobile client will poll this endpoint at ~3-second intervals. Keep the query fast — the composite index `IX_SubmissionQueue_Status_SessionId` covers both the `IF NOT EXISTS` check and the `TOP 1 WAITING` subquery.

---

## BE-SQ06 — Complete Endpoint

**File**: `queue/complete.js`  
**Effort**: S  
**Dependencies**: BE-SQ01  

### Description

`POST /queue/complete` — Mark an ACTIVE session as COMPLETED after all reports are submitted.

### Request Body

```json
{
  "sessionId": 42,
  "reportCount": 5
}
```

### Response (200)

```json
{
  "success": true,
  "sessionId": 42,
  "status": "COMPLETED",
  "completedAt": "2026-04-14T10:30:00.000Z"
}
```

### Logic

1. Validate `sessionId` is present. Return `400` if missing.
2. Update:
   ```sql
   UPDATE tbl_SubmissionQueue
   SET    Status      = 'COMPLETED',
          CompletedAt = GETDATE(),
          ReportCount = @reportCount
   WHERE  SessionId = @sessionId
     AND  Status    = 'ACTIVE'
   ```
3. If affected rows = 0 → return `409 Conflict` with message indicating session is not ACTIVE.
4. Return success response.

### Implementation Notes

- Use `db('tbl_SubmissionQueue').update({...}).where({...})` then check the return value for affected row count.
- Knex on MSSQL returns the row count from `.update()`.
- Log at `info` level.

---

## BE-SQ07 — Fail Endpoint

**File**: `queue/fail.js`  
**Effort**: S  
**Dependencies**: BE-SQ01  

### Description

`POST /queue/fail` — Mark a session as FAILED. Accepts sessions in ACTIVE or WAITING status so the client can abort from either state.

### Request Body

```json
{
  "sessionId": 42,
  "reason": "string (optional)"
}
```

### Response (200)

```json
{
  "success": true,
  "sessionId": 42,
  "status": "FAILED"
}
```

### Logic

1. Validate `sessionId`. Return `400` if missing.
2. Update:
   ```sql
   UPDATE tbl_SubmissionQueue
   SET    Status      = 'FAILED',
          CompletedAt = GETDATE()
   WHERE  SessionId = @sessionId
     AND  Status IN ('ACTIVE', 'WAITING')
   ```
3. If affected rows = 0 → return `409 Conflict`.
4. Return success.

### Implementation Notes

- Use Knex `.whereIn('Status', ['ACTIVE', 'WAITING'])` for the multi-status condition.
- Log the optional `reason` at `warn` level for debugging failed submissions.

---

## BE-SQ08 — Queue Router & Server Mount

**Files**: `queue/routes/index.js`, `server.js`, `swagger.js`  
**Effort**: S  
**Dependencies**: BE-SQ04, BE-SQ05, BE-SQ06, BE-SQ07  

### Description

Wire all queue endpoints together via an Express router and mount it on the main app.

### `queue/routes/index.js`

```js
const express = require('express');
const router = express.Router();

router.use('/enqueue', require('../enqueue'));
router.use('/check-turn', require('../check-turn'));
router.use('/complete', require('../complete'));
router.use('/fail', require('../fail'));

module.exports = router;
```

### `server.js` Changes

Add after the existing route registrations (line 86):

```js
const queueRoutes = require('./queue/routes');
// ...
app.use('/queue', queueRoutes);
```

### `swagger.js` Changes

Add to the `apis` array:

```js
"./queue/*.js",
```

### Implementation Notes

- Follow the existing pattern from `user/routes/index.js`.
- Place the `require` and `app.use` between the existing `app.use('/api', ...)` and the health check endpoint.

---

## BE-SQ09 — Swagger JSDoc Annotations

**Files**: `queue/enqueue.js`, `queue/check-turn.js`, `queue/complete.js`, `queue/fail.js`  
**Effort**: M  
**Dependencies**: BE-SQ04, BE-SQ05, BE-SQ06, BE-SQ07  

### Description

Add `@swagger` JSDoc blocks to each route handler, following the annotation style established in `user/validateSubmission.js`.

### Tags

All queue endpoints should use the tag `Submission Queue`:

```yaml
tags: [Submission Queue]
```

### Annotations to Write

#### POST /queue/enqueue

```yaml
/queue/enqueue:
  post:
    summary: Join the submission queue
    description: >
      Adds the supervisor to the submission queue. Idempotent — returns existing
      WAITING/ACTIVE session if one exists for the supervisorId.
    tags: [Submission Queue]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [userId, supervisorId]
            properties:
              userId:
                type: string
              supervisorId:
                type: string
              deviceInfo:
                type: string
    responses:
      200:
        description: Queue entry created or existing entry returned
        content:
          application/json:
            schema:
              type: object
              properties:
                sessionId:
                  type: integer
                status:
                  type: string
                  enum: [WAITING, ACTIVE]
                position:
                  type: integer
                isNew:
                  type: boolean
      400:
        description: Missing required fields
      500:
        description: Server error
```

#### POST /queue/check-turn

```yaml
/queue/check-turn:
  post:
    summary: Check if session is next and activate it
    description: >
      Expires stale sessions, then atomically activates the next WAITING session
      using row-level locking. Returns the caller's current position and status.
    tags: [Submission Queue]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [sessionId, supervisorId]
            properties:
              sessionId:
                type: integer
              supervisorId:
                type: string
    responses:
      200:
        description: Turn check result
        content:
          application/json:
            schema:
              type: object
              properties:
                isMyTurn:
                  type: boolean
                position:
                  type: integer
                activeSessionId:
                  type: integer
                status:
                  type: string
                  enum: [ACTIVE, WAITING]
      404:
        description: Session not found
      410:
        description: Session expired, completed, or failed
      400:
        description: Missing required fields
      500:
        description: Server error
```

#### POST /queue/complete

```yaml
/queue/complete:
  post:
    summary: Mark session as completed
    description: Transitions an ACTIVE session to COMPLETED after all reports are submitted.
    tags: [Submission Queue]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [sessionId]
            properties:
              sessionId:
                type: integer
              reportCount:
                type: integer
    responses:
      200:
        description: Session completed
      409:
        description: Session is not in ACTIVE status
      400:
        description: Missing required fields
      500:
        description: Server error
```

#### POST /queue/fail

```yaml
/queue/fail:
  post:
    summary: Mark session as failed
    description: Transitions an ACTIVE or WAITING session to FAILED (client abort).
    tags: [Submission Queue]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [sessionId]
            properties:
              sessionId:
                type: integer
              reason:
                type: string
    responses:
      200:
        description: Session marked as failed
      409:
        description: Session is not in ACTIVE or WAITING status
      400:
        description: Missing required fields
      500:
        description: Server error
```

---

## BE-SQ10 — Integration Tests

**Files**: `tests/queue/*.test.js` (new directory)  
**Effort**: L  
**Dependencies**: BE-SQ04, BE-SQ05, BE-SQ06, BE-SQ07, BE-SQ08  

### Description

End-to-end tests against the running server (or supertest). Covers the full queue lifecycle and concurrency edge cases.

### Test Cases

| # | Test | Validates |
|---|------|-----------|
| 1 | Enqueue → returns sessionId, position = 1, isNew = true | Happy path |
| 2 | Enqueue same supervisorId twice → returns same sessionId, isNew = false | Idempotency |
| 3 | Enqueue two different supervisors → positions 1 and 2 | Ordering |
| 4 | Check-turn when first in line → isMyTurn = true, status = ACTIVE | Activation |
| 5 | Check-turn when second in line → isMyTurn = false, position = 2 | Waiting |
| 6 | Complete an ACTIVE session → status = COMPLETED | Lifecycle |
| 7 | Complete a non-ACTIVE session → 409 | Guard |
| 8 | Fail an ACTIVE session → status = FAILED | Abort |
| 9 | Fail a WAITING session → status = FAILED | Pre-activation abort |
| 10 | Stale expiry: ACTIVE session older than timeout → expires, next WAITING activates | Expiry |
| 11 | Check-turn on EXPIRED session → 410 Gone | Terminal state |
| 12 | Missing fields on all endpoints → 400 | Validation |
| 13 | Concurrent check-turn: two requests racing → only one activates | Locking correctness |

### Implementation Notes

- Use `supertest` for HTTP assertions.
- For test 13 (concurrency), use `Promise.all` to fire two check-turn requests simultaneously and assert that exactly one returns `isMyTurn: true`.
- Tests should clean up the `tbl_SubmissionQueue` table in `beforeEach`/`afterEach` hooks.
- If the project doesn't already have a test runner, add `jest` + `supertest` as dev dependencies.

---

## Execution Order

```
Phase 1 — Schema (parallel)
  BE-SQ01  Knex migration
  BE-SQ02  Raw SQL setup

Phase 2 — Core logic (parallel after Phase 1)
  BE-SQ03  expire-stale
  BE-SQ04  enqueue
  BE-SQ06  complete
  BE-SQ07  fail

Phase 3 — Critical path (after BE-SQ03)
  BE-SQ05  check-turn

Phase 4 — Wiring (after Phase 2 + 3)
  BE-SQ08  Router + server mount + swagger config
  BE-SQ09  Swagger annotations

Phase 5 — Verification (after Phase 4)
  BE-SQ10  Integration tests
```

---

## Appendix A — Full Activation SQL (Copy-Paste Ready)

This is the exact raw SQL to use inside `check-turn.js` via `db.raw()`:

```sql
BEGIN TRANSACTION;

-- Acquire UPDLOCK + HOLDLOCK on ACTIVE rows (or an empty range lock if none exist).
-- This prevents any concurrent transaction from passing the same check.
IF NOT EXISTS (
    SELECT 1
    FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
    WHERE  Status = 'ACTIVE'
)
BEGIN
    -- Activate the longest-waiting session (FIFO by SessionId).
    UPDATE tbl_SubmissionQueue
    SET    Status      = 'ACTIVE',
           ActivatedAt = GETDATE()
    WHERE  SessionId = (
        SELECT TOP 1 SessionId
        FROM   tbl_SubmissionQueue WITH (UPDLOCK, HOLDLOCK)
        WHERE  Status = 'WAITING'
        ORDER BY SessionId ASC
    );
END

COMMIT TRANSACTION;
```

### Locking Walkthrough

1. **Connection A** enters the transaction and runs `SELECT 1 ... WITH (UPDLOCK, HOLDLOCK) WHERE Status = 'ACTIVE'`.
   - If no ACTIVE row exists, MSSQL acquires a **range lock** on the key range in `IX_SubmissionQueue_Status_SessionId` where `Status = 'ACTIVE'` would be inserted (due to `HOLDLOCK` = serializable semantics).
2. **Connection B** enters and runs the same `SELECT`. It is **blocked** because Connection A holds the range lock.
3. Connection A proceeds to `UPDATE`, setting one row to `ACTIVE`, then `COMMIT`s. The lock is released.
4. Connection B unblocks, runs its `SELECT`, now finds one ACTIVE row → `IF NOT EXISTS` is false → skips the UPDATE.
5. Result: exactly one session is activated.

### Why Not Use Knex Transactions?

Knex `transaction()` uses `BEGIN TRAN` / `COMMIT` under the hood but does not support table hints (`WITH (UPDLOCK, HOLDLOCK)`). The only way to use these hints with the mssql driver is through `db.raw()`. The raw SQL approach is intentional and required for correctness.

---

## Appendix B — Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `QUEUE_STALE_TIMEOUT_MINUTES` | `30` | Minutes before an ACTIVE session is auto-expired |

---

## Appendix C — New Directory Structure

```
queue/
├── routes/
│   └── index.js        # Express router (BE-SQ08)
├── enqueue.js           # POST /queue/enqueue (BE-SQ04)
├── check-turn.js        # POST /queue/check-turn (BE-SQ05)
├── complete.js          # POST /queue/complete (BE-SQ06)
├── fail.js              # POST /queue/fail (BE-SQ07)
└── expire-stale.js      # Shared stale-expiry function (BE-SQ03)
```
