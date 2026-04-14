# Submission Queue — Technical Design

**Feature**: Submission Queue for DAR Form Pipeline  
**Status**: DRAFT  
**Author**: Solution Architect  
**Date**: 2026-04-14  
**Branch**: `MDAG-339`  
**Repos**: DAR_Middleware (backend), main_dar_app (frontend)

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Architecture Overview](#2-architecture-overview)
3. [Database Design](#3-database-design)
4. [API Contract](#4-api-contract)
5. [Concurrency Control](#5-concurrency-control)
6. [Stale Session Handling](#6-stale-session-handling)
7. [Frontend Integration](#7-frontend-integration)
8. [Sequence Diagram](#8-sequence-diagram)
9. [Non-Functional Requirements](#9-non-functional-requirements)
10. [Migration Plan](#10-migration-plan)
11. [Risks and Mitigations](#11-risks-and-mitigations)

---

## 1. Problem Statement

### Current State

The DAR write pipeline is a **multi-step, client-orchestrated** sequence of independent HTTP calls per report:

```
checkBatch → createBatch → createHeader → createDetails
  → createAccomplishments → createMaterials → calculateTotalsHdr
  → calculateTotalsBatch → deleteLocalReport
```

Each step is a separate Express endpoint. The backend uses **no database transactions, no row-level locking, and no application-level serialization**.

### Race Conditions Identified

| Resource | Pattern | Failure Mode |
|----------|---------|--------------|
| `tblbatchno.batchno` | Read current → increment → update | Two concurrent users read same value → duplicate `Batchno` on `tblDARbatch` |
| `tbldocno.docno` | Read current → increment → update | Two concurrent headers read same value → duplicate `DocumentNo` on `tblDARhdr` |
| `tblDARbatch` existence check | SELECT → INSERT if not found | Two requests both find no batch → two batches created for same (userid, Entrydate, Doctype) |
| `SeqNo` on dtls/accomp/materials | MAX(SeqNo) + 1 | Concurrent inserts get same SeqNo → constraint violation or data corruption |

### Why a Queue

Wrapping each individual SQL statement in a transaction is insufficient — the pipeline spans **6–8 separate HTTP requests** over several seconds. A database transaction held that long would cause lock escalation and deadlocks under load. Instead, we serialize access at the **session level**: only one user's pipeline runs at a time, eliminating all four race conditions without modifying any existing endpoint code.

### Design Goal

Introduce a **lightweight submission queue** that gates access to the DAR write pipeline so that exactly one user's multi-step submission runs at a time. The queue is transparent to the existing endpoints — no changes to batch, header, detail, accomplishment, material, or totals routes.

---

## 2. Architecture Overview

### Component Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     main_dar_app (Flutter)                   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              SubmissionQueueService                  │    │
│  │  enqueue() → pollUntilActive() → runPipeline()      │    │
│  │  → complete() / fail()                              │    │
│  └──────────┬──────────────────────────────┬───────────┘    │
│             │ Queue API calls               │ Existing       │
│             │                               │ pipeline calls │
└─────────────┼───────────────────────────────┼───────────────┘
              │                               │
              ▼                               ▼
┌─────────────────────────────────────────────────────────────┐
│                   DAR_Middleware (Express)                    │
│                                                             │
│  ┌──────────────────┐    ┌────────────────────────────┐     │
│  │   /queue/*        │    │   /sendDARForm/*            │     │
│  │   (NEW endpoints) │    │   (UNCHANGED)              │     │
│  │                   │    │   batch, hdr, dtls, accomp │     │
│  │   POST /enqueue   │    │   materials, totals        │     │
│  │   GET  /check-turn│    └──────────────┬─────────────┘     │
│  │   POST /complete  │                   │                   │
│  │   POST /fail      │                   │                   │
│  └────────┬──────────┘                   │                   │
│           │                              │                   │
│           ▼                              ▼                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                     MSSQL (DAR)                     │     │
│  │                                                     │     │
│  │   tbl_SubmissionQueue (NEW)                         │     │
│  │   tblDARbatch, tblDARhdr, tbldocno, ...  (EXISTING)│     │
│  └─────────────────────────────────────────────────────┘     │
│                                                             │
│  ┌──────────────────┐                                       │
│  │  Stale Reaper    │  (runs on /check-turn + optional      │
│  │  (inline)        │   scheduled interval)                 │
│  └──────────────────┘                                       │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Session-level serialization** (not row-level) | Pipeline spans 6–8 HTTP requests over seconds; row locks would escalate |
| **Polling from client** (not WebSocket/push) | Simpler; Flutter `http` client is already in use; polling is straightforward and sufficient for the expected concurrency |
| **`setInterval` reaper** (background timer) | Physical server runs a persistent Node.js process; `setInterval` fires reliably. Also called inline in `check-turn` as a belt-and-suspenders approach |
| **Queue per global scope** (not per-userid) | Race conditions cross users (same `tbldocno.userid` can race between devices; `tblbatchno` races across users sharing same data) |
| **No changes to existing endpoints** | De-risks implementation; queue wraps the pipeline externally |

---

## 3. Database Design

### DDL — `tbl_SubmissionQueue`

```sql
CREATE TABLE [dbo].[tbl_SubmissionQueue] (
    [SessionId]     INT IDENTITY(1,1)   NOT NULL,
    [UserId]        NVARCHAR(100)       NOT NULL,
    [SupervisorId]  NVARCHAR(100)       NULL,
    [Status]        NVARCHAR(20)        NOT NULL
                    CONSTRAINT [DF_SubmissionQueue_Status] DEFAULT ('WAITING'),
    [CreatedAt]     DATETIME2(3)        NOT NULL
                    CONSTRAINT [DF_SubmissionQueue_CreatedAt] DEFAULT (SYSUTCDATETIME()),
    [ActivatedAt]   DATETIME2(3)        NULL,
    [CompletedAt]   DATETIME2(3)        NULL,
    [ReportCount]   INT                 NOT NULL
                    CONSTRAINT [DF_SubmissionQueue_ReportCount] DEFAULT (0),
    [DeviceInfo]    NVARCHAR(500)       NULL,
    [ErrorMessage]  NVARCHAR(2000)      NULL,

    CONSTRAINT [PK_SubmissionQueue] PRIMARY KEY CLUSTERED ([SessionId]),

    CONSTRAINT [CK_SubmissionQueue_Status] CHECK (
        [Status] IN ('WAITING', 'ACTIVE', 'COMPLETED', 'FAILED', 'EXPIRED')
    )
);
```

### Indexes

```sql
-- Primary query path: find next WAITING session to activate
CREATE NONCLUSTERED INDEX [IX_SubmissionQueue_Status_CreatedAt]
ON [dbo].[tbl_SubmissionQueue] ([Status], [CreatedAt] ASC)
INCLUDE ([SessionId], [UserId])
WHERE [Status] IN ('WAITING', 'ACTIVE');

-- Reaper query: find stale ACTIVE sessions
CREATE NONCLUSTERED INDEX [IX_SubmissionQueue_ActiveSessions]
ON [dbo].[tbl_SubmissionQueue] ([Status], [ActivatedAt])
WHERE [Status] = 'ACTIVE';

-- Prevent duplicate active/waiting sessions per user
CREATE UNIQUE NONCLUSTERED INDEX [UX_SubmissionQueue_UserPending]
ON [dbo].[tbl_SubmissionQueue] ([UserId])
WHERE [Status] IN ('WAITING', 'ACTIVE');
```

### Status State Machine

```
  ┌─────────┐   check-turn    ┌────────┐   complete    ┌───────────┐
  │ WAITING ├─────────────────►│ ACTIVE ├──────────────►│ COMPLETED │
  └────┬────┘                  └───┬────┘               └───────────┘
       │                           │
       │ (user cancels / expires)  │  fail / reaper timeout
       │                           │
       ▼                           ▼
  ┌─────────┐               ┌────────┐
  │ EXPIRED │               │ FAILED │
  └─────────┘               └────────┘
```

Valid transitions:
- `WAITING → ACTIVE` — check-turn activates when no other ACTIVE session exists
- `WAITING → EXPIRED` — reaper expires WAITING sessions older than `WAITING_TIMEOUT`
- `ACTIVE → COMPLETED` — client calls `/complete` after successful pipeline
- `ACTIVE → FAILED` — client calls `/fail` on pipeline error
- `ACTIVE → EXPIRED` — reaper expires ACTIVE sessions older than `ACTIVE_TIMEOUT`

---

## 4. API Contract

All endpoints are mounted under `/queue` on the Express app.

### 4.1 POST `/queue/enqueue`

Enqueue a new submission session.

**Request Body:**

```json
{
  "userId": "string (required)",
  "supervisorId": "string (optional)",
  "reportCount": 3,
  "deviceInfo": "string (optional — device model, OS version)"
}
```

**Success Response — 201 Created:**

```json
{
  "sessionId": 42,
  "status": "WAITING",
  "position": 1,
  "estimatedWaitSec": 0,
  "createdAt": "2026-04-14T08:30:00.000Z"
}
```

`position` is 1-based count of WAITING sessions ahead (including self). `position = 1` with no ACTIVE session means this session is next.

**Error Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 400 | Missing `userId` | `{ "error": "userId is required" }` |
| 409 | User already has a WAITING or ACTIVE session | `{ "error": "User already has a pending session", "existingSessionId": 41, "status": "ACTIVE" }` |
| 503 | Queue depth exceeds `MAX_QUEUE_DEPTH` | `{ "error": "Queue is full. Try again later.", "queueDepth": 10 }` |
| 500 | Server error | `{ "error": "Failed to enqueue", "details": "..." }` |

### 4.2 GET `/queue/check-turn?sessionId=42`

Poll to check whether this session is now ACTIVE. Also triggers the inline reaper.

**Query Parameters:**

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `sessionId` | INT | Yes | The session to check |

**Success Response — 200 OK:**

```json
{
  "sessionId": 42,
  "status": "ACTIVE",
  "position": 0,
  "activatedAt": "2026-04-14T08:30:05.000Z"
}
```

When `status` is still `WAITING`:

```json
{
  "sessionId": 42,
  "status": "WAITING",
  "position": 2,
  "estimatedWaitSec": 60
}
```

**Error Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 400 | Missing `sessionId` | `{ "error": "sessionId is required" }` |
| 404 | Session not found | `{ "error": "Session not found" }` |
| 410 | Session was EXPIRED by reaper | `{ "error": "Session expired", "status": "EXPIRED" }` |
| 500 | Server error | `{ "error": "Failed to check turn", "details": "..." }` |

### 4.3 POST `/queue/complete`

Mark the session as COMPLETED after the entire pipeline succeeds.

**Request Body:**

```json
{
  "sessionId": 42
}
```

**Success Response — 200 OK:**

```json
{
  "sessionId": 42,
  "status": "COMPLETED",
  "completedAt": "2026-04-14T08:31:15.000Z",
  "durationMs": 70000
}
```

**Error Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 400 | Missing `sessionId` | `{ "error": "sessionId is required" }` |
| 404 | Session not found | `{ "error": "Session not found" }` |
| 409 | Session not in ACTIVE status | `{ "error": "Session is not ACTIVE", "currentStatus": "EXPIRED" }` |
| 500 | Server error | `{ "error": "Failed to complete session", "details": "..." }` |

### 4.4 POST `/queue/fail`

Mark the session as FAILED when the pipeline encounters an unrecoverable error.

**Request Body:**

```json
{
  "sessionId": 42,
  "errorMessage": "Failed to create header: 500 - Internal Server Error"
}
```

**Success Response — 200 OK:**

```json
{
  "sessionId": 42,
  "status": "FAILED",
  "completedAt": "2026-04-14T08:30:45.000Z"
}
```

**Error Responses:**

| Status | Condition | Body |
|--------|-----------|------|
| 400 | Missing `sessionId` | `{ "error": "sessionId is required" }` |
| 404 | Session not found | `{ "error": "Session not found" }` |
| 409 | Session not in ACTIVE status | `{ "error": "Session is not ACTIVE", "currentStatus": "COMPLETED" }` |
| 500 | Server error | `{ "error": "Failed to mark session as failed", "details": "..." }` |

### 4.5 GET `/queue/status`

Admin/debug endpoint returning current queue state.

**Success Response — 200 OK:**

```json
{
  "activeSession": { "sessionId": 42, "userId": "USR001", "activatedAt": "..." },
  "waitingCount": 3,
  "waitingSessions": [
    { "sessionId": 43, "userId": "USR002", "position": 1, "createdAt": "..." },
    { "sessionId": 44, "userId": "USR003", "position": 2, "createdAt": "..." },
    { "sessionId": 45, "userId": "USR004", "position": 3, "createdAt": "..." }
  ]
}
```

---

## 5. Concurrency Control

### The Core Problem

Two clients polling `/check-turn` simultaneously could both see "no ACTIVE session exists" and both promote themselves to ACTIVE. Standard `READ COMMITTED` isolation does not prevent this because the reads are non-blocking.

### Solution: Atomic Activation with UPDLOCK + HOLDLOCK

The check-turn endpoint uses a **single SQL statement** that atomically:
1. Expires stale sessions (reaper inline)
2. Checks if any ACTIVE session exists
3. Activates the oldest WAITING session if the queue is clear

```sql
-- Step 1: Expire stale ACTIVE sessions (inline reaper)
UPDATE tbl_SubmissionQueue
SET    [Status] = 'EXPIRED',
       [CompletedAt] = SYSUTCDATETIME(),
       [ErrorMessage] = 'Expired by reaper: active timeout exceeded'
WHERE  [Status] = 'ACTIVE'
AND    [ActivatedAt] < DATEADD(MINUTE, -@activeTimeoutMin, SYSUTCDATETIME());

-- Step 2: Expire stale WAITING sessions
UPDATE tbl_SubmissionQueue
SET    [Status] = 'EXPIRED',
       [CompletedAt] = SYSUTCDATETIME(),
       [ErrorMessage] = 'Expired by reaper: waiting timeout exceeded'
WHERE  [Status] = 'WAITING'
AND    [CreatedAt] < DATEADD(MINUTE, -@waitingTimeoutMin, SYSUTCDATETIME());

-- Step 3: Atomic activation — single statement, no TOCTOU gap
UPDATE tbl_SubmissionQueue
SET    [Status] = 'ACTIVE',
       [ActivatedAt] = SYSUTCDATETIME()
OUTPUT inserted.[SessionId],
       inserted.[Status],
       inserted.[ActivatedAt]
WHERE  [SessionId] = (
    SELECT TOP 1 sq.[SessionId]
    FROM   tbl_SubmissionQueue sq WITH (UPDLOCK, HOLDLOCK)
    WHERE  sq.[Status] = 'WAITING'
    AND    NOT EXISTS (
        SELECT 1
        FROM   tbl_SubmissionQueue active WITH (UPDLOCK, HOLDLOCK)
        WHERE  active.[Status] = 'ACTIVE'
    )
    ORDER BY sq.[CreatedAt] ASC
)
AND [Status] = 'WAITING';
```

### Why This Works

| Concern | Mechanism |
|---------|-----------|
| **Two pollers race to activate** | `UPDLOCK` on the subquery locks the candidate WAITING row and all ACTIVE rows. The second poller blocks on the lock until the first commits. By then, an ACTIVE row exists, so the `NOT EXISTS` check fails and no activation occurs. |
| **HOLDLOCK** (serializable range) | Prevents phantom inserts of ACTIVE rows between the `NOT EXISTS` check and the UPDATE. |
| **Single-statement atomicity** | The UPDATE + subquery is one atomic operation within an implicit transaction. No application-level BEGIN/COMMIT needed. |
| **OUTPUT clause** | Returns the activated row (if any) in one round-trip, avoiding a second SELECT. |

### Knex Implementation

```javascript
async function tryActivateSession(db, sessionId, config) {
  // Step 1 & 2: Reaper (can run outside the critical section)
  await db.raw(`
    UPDATE tbl_SubmissionQueue
    SET [Status] = 'EXPIRED', [CompletedAt] = SYSUTCDATETIME(),
        [ErrorMessage] = 'Expired: active timeout'
    WHERE [Status] = 'ACTIVE'
    AND [ActivatedAt] < DATEADD(MINUTE, -?, SYSUTCDATETIME())
  `, [config.ACTIVE_TIMEOUT_MIN]);

  await db.raw(`
    UPDATE tbl_SubmissionQueue
    SET [Status] = 'EXPIRED', [CompletedAt] = SYSUTCDATETIME(),
        [ErrorMessage] = 'Expired: waiting timeout'
    WHERE [Status] = 'WAITING'
    AND [CreatedAt] < DATEADD(MINUTE, -?, SYSUTCDATETIME())
  `, [config.WAITING_TIMEOUT_MIN]);

  // Step 3: Atomic activation
  const result = await db.raw(`
    UPDATE tbl_SubmissionQueue
    SET [Status] = 'ACTIVE', [ActivatedAt] = SYSUTCDATETIME()
    OUTPUT inserted.[SessionId], inserted.[Status], inserted.[ActivatedAt]
    WHERE [SessionId] = (
      SELECT TOP 1 sq.[SessionId]
      FROM tbl_SubmissionQueue sq WITH (UPDLOCK, HOLDLOCK)
      WHERE sq.[Status] = 'WAITING'
      AND NOT EXISTS (
        SELECT 1 FROM tbl_SubmissionQueue active WITH (UPDLOCK, HOLDLOCK)
        WHERE active.[Status] = 'ACTIVE'
      )
      ORDER BY sq.[CreatedAt] ASC
    )
    AND [Status] = 'WAITING'
  `);

  const rows = Array.isArray(result) ? result : [];
  return rows.length > 0 ? rows[0] : null;
}
```

---

## 6. Stale Session Handling

### Why Sessions Go Stale

| Scenario | Result |
|----------|--------|
| App crashes mid-pipeline | ACTIVE session never calls `/complete` or `/fail` |
| Network loss during submission | Client cannot reach `/fail`; session stays ACTIVE |
| User kills app | Same as crash |
| User enqueues but never polls | WAITING session never progresses |

### Reaper Strategy

The reaper runs via two complementary mechanisms:

1. **`setInterval` background timer** — Since the backend runs as a persistent Node.js process on a physical server, `setInterval` fires reliably every N seconds. This is the primary reaper. On process startup, an immediate cleanup pass runs to clear any sessions orphaned during downtime.
2. **Inline in `/check-turn`** — Belt-and-suspenders: the reaper also runs at the top of every `check-turn` call before activation decisions. This ensures stale sessions are cleared even if the timer hasn't fired yet.

This dual approach is safe because:
- `/check-turn` is the only write path for activation, so inline reaping always runs before activation decisions
- The reaper UPDATEs are cheap (filtered index on `Status = 'ACTIVE'`)
- `setInterval` on a persistent process guarantees cleanup even when no clients are polling

### Timeout Configuration

| Parameter | Default | Env Variable | Description |
|-----------|---------|--------------|-------------|
| `ACTIVE_TIMEOUT_MIN` | 5 | `QUEUE_ACTIVE_TIMEOUT_MIN` | Max time a session can remain ACTIVE before forced expiry |
| `WAITING_TIMEOUT_MIN` | 15 | `QUEUE_WAITING_TIMEOUT_MIN` | Max time a session can remain WAITING before forced expiry |

**Rationale for 5-minute active timeout**: A single report pipeline takes ~5–15 seconds. A session with 10 reports would take ~2.5 minutes. The 5-minute timeout provides a 2× safety margin.

### Client-Side Heartbeat (Future Enhancement)

Not included in v1. If needed, `/queue/heartbeat` could extend the ACTIVE timeout. For now, the 5-minute window is sufficient.

---

## 7. Frontend Integration

### New Service: `SubmissionQueueService`

Location: `lib/app/service/queue/submission_queue_service.dart`

```dart
class SubmissionQueueService {
  static const Duration _pollInterval = Duration(seconds: 3);
  static const Duration _pollTimeout = Duration(minutes: 15);

  /// Enqueue, poll until ACTIVE, return sessionId.
  /// Throws on timeout, expiry, or queue-full.
  static Future<int> waitForTurn({
    required String userId,
    String? supervisorId,
    required int reportCount,
    String? deviceInfo,
    required void Function(int position) onPositionUpdate,
  }) async {
    // POST /queue/enqueue
    final enqueueResponse = await _enqueue(
      userId: userId,
      supervisorId: supervisorId,
      reportCount: reportCount,
      deviceInfo: deviceInfo,
    );
    final sessionId = enqueueResponse['sessionId'] as int;
    onPositionUpdate(enqueueResponse['position'] as int);

    // Poll /queue/check-turn until ACTIVE
    final deadline = DateTime.now().add(_pollTimeout);
    while (DateTime.now().isBefore(deadline)) {
      await Future.delayed(_pollInterval);
      final check = await _checkTurn(sessionId);
      final status = check['status'] as String;

      if (status == 'ACTIVE') return sessionId;
      if (status == 'EXPIRED') throw QueueExpiredException();

      onPositionUpdate(check['position'] as int);
    }
    throw QueueTimeoutException();
  }

  static Future<void> complete(int sessionId) async { /* POST /queue/complete */ }
  static Future<void> fail(int sessionId, String error) async { /* POST /queue/fail */ }
}
```

### Modified `review_action.dart`

The existing send loop is wrapped — no changes to any `*DialogService` or `*ApiService`.

```dart
Future<void> sendToServerButtonClicked() async {
  // 1. Existing validation
  final validationResult = await SubmissionValidationService
      .validateBeforeSubmission(context);
  if (validationResult == null) return;

  // 2. Confirmation dialog (existing)
  final confirmed = await showSubmissionConfirmationDialog(context);
  if (!confirmed) return;

  final reports = reviewProvider.completedReports;

  // 3. NEW — Enqueue and wait for turn
  int? sessionId;
  try {
    sessionId = await SubmissionQueueService.waitForTurn(
      userId: userId,
      supervisorId: supervisorId,
      reportCount: reports.length,
      deviceInfo: _getDeviceInfo(),
      onPositionUpdate: (pos) => _updateQueuePositionUI(pos),
    );
  } on QueueFullException {
    _showError('Server is busy. Please try again in a few minutes.');
    return;
  } on QueueExpiredException {
    _showError('Your queue session expired. Please try again.');
    return;
  } on QueueTimeoutException {
    _showError('Timed out waiting for your turn. Please try again.');
    return;
  }

  // 4. EXISTING pipeline — unchanged
  bool hasError = false;
  Set<String> batchIds = {};
  try {
    // ... existing individual + group report loops ...
    // ... existing batch totals loop ...
  } catch (e) {
    hasError = true;
  }

  // 5. NEW — Release queue slot
  if (hasError) {
    await SubmissionQueueService.fail(sessionId, 'Pipeline error');
  } else {
    await SubmissionQueueService.complete(sessionId);
  }
}
```

### Queue Position UI

While waiting, display a modal or inline indicator:

```
┌─────────────────────────────────┐
│   Waiting for your turn...      │
│                                 │
│   Position in queue: 2          │
│   ████████░░░░░░░░ Waiting...   │
│                                 │
│          [ Cancel ]             │
└─────────────────────────────────┘
```

On cancel, the client can call `/queue/fail` with `errorMessage: "Cancelled by user"` to release the slot.

### Error Handling Matrix

| Scenario | Client Behavior |
|----------|-----------------|
| `/enqueue` returns 409 (already queued) | Show "You already have a submission in progress" with the existing `sessionId`; resume polling |
| `/enqueue` returns 503 (queue full) | Show "Server is busy, try again later" |
| `/check-turn` returns 410 (expired) | Show "Session expired, please retry" |
| Pipeline fails mid-report | Call `/queue/fail`, show error, stop loop |
| App killed during ACTIVE | Reaper expires after 5 min; next poll by any user clears it |
| Network loss during polling | `http` throws `SocketException`; retry with backoff, up to `_pollTimeout` |

---

## 8. Sequence Diagram

```
┌──────────┐          ┌──────────────┐          ┌────────┐          ┌──────┐
│  Flutter  │          │  /queue/*    │          │ /send  │          │ MSSQL│
│  App      │          │  (new)       │          │ DARForm│          │      │
└────┬─────┘          └──────┬───────┘          └───┬────┘          └──┬───┘
     │                       │                      │                  │
     │  1. POST /queue/enqueue                      │                  │
     │  { userId, reportCount }                     │                  │
     ├──────────────────────►│                      │                  │
     │                       │  INSERT WAITING      │                  │
     │                       ├─────────────────────────────────────────►
     │                       │  ◄── SessionId=42 ──────────────────────┤
     │  ◄── 201 { sessionId:42, position:2 } ──────┤                  │
     │                       │                      │                  │
     │  [Show "Position: 2"]│                      │                  │
     │                       │                      │                  │
     │  2. GET /queue/check-turn?sessionId=42       │                  │
     ├──────────────────────►│                      │                  │
     │                       │  Run reaper SQL      │                  │
     │                       ├─────────────────────────────────────────►
     │                       │  Atomic activation   │                  │
     │                       │  (UPDLOCK+HOLDLOCK)  │                  │
     │                       ├─────────────────────────────────────────►
     │                       │  ◄── still WAITING ─────────────────────┤
     │  ◄── 200 { status: "WAITING", position: 1 } │                  │
     │                       │                      │                  │
     │  [wait 3 seconds]     │                      │                  │
     │                       │                      │                  │
     │  3. GET /queue/check-turn?sessionId=42       │                  │
     ├──────────────────────►│                      │                  │
     │                       │  Reaper + activation  │                  │
     │                       ├─────────────────────────────────────────►
     │                       │  ◄── ACTIVE (row updated) ──────────────┤
     │  ◄── 200 { status: "ACTIVE" } ──────────────┤                  │
     │                       │                      │                  │
     │  ═══════════════ EXISTING PIPELINE BEGINS ══════════════════    │
     │                       │                      │                  │
     │  4. POST /sendDARForm/batch/tblDARbatch      │                  │
     ├─────────────────────────────────────────────►│                  │
     │                       │                      │  INSERT batch    │
     │                       │                      ├─────────────────►│
     │  ◄── 201 { Batch_id } ──────────────────────┤                  │
     │                       │                      │                  │
     │  5. POST /sendDARForm/mps/tblDARhdr          │                  │
     ├─────────────────────────────────────────────►│                  │
     │                       │                      │  docno++ + INSERT│
     │                       │                      ├─────────────────►│
     │  ◄── 201 { Hdr_id } ────────────────────────┤                  │
     │                       │                      │                  │
     │  6-8. Details, Accomp, Materials, Totals     │                  │
     ├─────────────────────────────────────────────►│                  │
     │  ◄── 200/201 ──────────────────────────────-┤                  │
     │                       │                      │                  │
     │  [repeat 4-8 for each report]                │                  │
     │                       │                      │                  │
     │  ═══════════════ PIPELINE COMPLETE ═════════════════════════    │
     │                       │                      │                  │
     │  9. POST /queue/complete                     │                  │
     │  { sessionId: 42 }    │                      │                  │
     ├──────────────────────►│                      │                  │
     │                       │  UPDATE COMPLETED    │                  │
     │                       ├─────────────────────────────────────────►
     │  ◄── 200 { status: "COMPLETED" } ───────────┤                  │
     │                       │                      │                  │
     ▼                       ▼                      ▼                  ▼
```

### Failure Path

```
     │  6. POST /sendDARForm/mps/tblDARdtls → 500 error              │
     │  ◄── 500 ──────────────────────────────────-┤                  │
     │                       │                      │                  │
     │  7. POST /queue/fail  │                      │                  │
     │  { sessionId: 42, errorMessage: "..." }      │                  │
     ├──────────────────────►│                      │                  │
     │                       │  UPDATE FAILED       │                  │
     │                       ├─────────────────────────────────────────►
     │  ◄── 200 { status: "FAILED" } ──────────────┤                  │
```

---

## 9. Non-Functional Requirements

### Performance

| Metric | Target | Rationale |
|--------|--------|-----------|
| `/enqueue` latency | < 200ms | Single INSERT + COUNT query |
| `/check-turn` latency | < 300ms | Reaper UPDATE + activation UPDATE with locks |
| `/complete` / `/fail` latency | < 100ms | Single UPDATE by PK |
| End-to-end single report pipeline | < 15s | Existing; no change |
| End-to-end 10-report session | < 3 min | 10 × 15s + overhead |

### Timeout Configuration

| Parameter | Value | Notes |
|-----------|-------|-------|
| Client poll interval | 3 seconds | Balances responsiveness vs server load |
| Client poll timeout | 15 minutes | Max time client waits in WAITING state |
| Active session timeout (reaper) | 5 minutes | 2× worst-case pipeline for 10 reports |
| Waiting session timeout (reaper) | 15 minutes | Matches client-side timeout |
| Max queue depth | 10 | Prevents unbounded queue; returns 503 |

### Scalability

| Concern | Approach |
|---------|----------|
| Physical server performance | Queue endpoints are lightweight (< 50ms per call); persistent Node process has no cold start overhead |
| Connection pool exhaustion | Knex pool max is 7; queue endpoints use 1 connection per call; lock hold time is < 50ms |
| Queue depth under load | `MAX_QUEUE_DEPTH = 10` caps waiting sessions; 503 returned when full |
| Database growth | Terminal states (COMPLETED, FAILED, EXPIRED) can be purged after 30 days via maintenance script |

### Observability

- All queue operations log via the existing `logger` (Winston/console)
- Log format: `[QUEUE] <operation> sessionId=<id> userId=<uid> status=<status>`
- Failed sessions log the `errorMessage` for debugging

---

## 10. Migration Plan

### Knex Migration File

File: `db/migrations/20260414_create_tbl_submission_queue.js`

```javascript
exports.up = async function (knex) {
  const exists = await knex.schema.hasTable('tbl_SubmissionQueue');

  if (!exists) {
    await knex.schema.createTable('tbl_SubmissionQueue', (table) => {
      table.increments('SessionId').primary();
      table.string('UserId', 100).notNullable();
      table.string('SupervisorId', 100).nullable();
      table.string('Status', 20).notNullable().defaultTo('WAITING');
      table.datetime('CreatedAt', { precision: 3 }).notNullable()
        .defaultTo(knex.raw('SYSUTCDATETIME()'));
      table.datetime('ActivatedAt', { precision: 3 }).nullable();
      table.datetime('CompletedAt', { precision: 3 }).nullable();
      table.integer('ReportCount').notNullable().defaultTo(0);
      table.string('DeviceInfo', 500).nullable();
      table.string('ErrorMessage', 2000).nullable();
    });

    // CHECK constraint
    await knex.raw(`
      ALTER TABLE [tbl_SubmissionQueue]
      ADD CONSTRAINT [CK_SubmissionQueue_Status]
      CHECK ([Status] IN ('WAITING','ACTIVE','COMPLETED','FAILED','EXPIRED'))
    `);

    // Filtered index for activation query
    await knex.raw(`
      CREATE NONCLUSTERED INDEX [IX_SubmissionQueue_Status_CreatedAt]
      ON [tbl_SubmissionQueue] ([Status], [CreatedAt] ASC)
      INCLUDE ([SessionId], [UserId])
      WHERE [Status] IN ('WAITING', 'ACTIVE')
    `);

    // Filtered index for reaper
    await knex.raw(`
      CREATE NONCLUSTERED INDEX [IX_SubmissionQueue_ActiveSessions]
      ON [tbl_SubmissionQueue] ([Status], [ActivatedAt])
      WHERE [Status] = 'ACTIVE'
    `);

    // Unique filtered index — one pending session per user
    await knex.raw(`
      CREATE UNIQUE NONCLUSTERED INDEX [UX_SubmissionQueue_UserPending]
      ON [tbl_SubmissionQueue] ([UserId])
      WHERE [Status] IN ('WAITING', 'ACTIVE')
    `);
  }
};

exports.down = async function (knex) {
  const exists = await knex.schema.hasTable('tbl_SubmissionQueue');
  if (exists) {
    await knex.schema.dropTable('tbl_SubmissionQueue');
  }
};
```

### Deployment Steps

1. **Run migration** on the DAR database: `npx knex migrate:latest --knexfile db/knexfile.js`
2. **Deploy backend** with new `/queue` routes (additive — no existing routes change)
3. **Deploy Flutter app** with `SubmissionQueueService` wrapper
4. **Verify** via `/queue/status` that the table is live and empty

### Rollback Strategy

| Step | Action |
|------|--------|
| 1. Revert Flutter app | Push previous APK (queue-unaware app works without `/queue` endpoints) |
| 2. Revert backend | Remove `/queue` route mount from `server.js` |
| 3. Rollback migration | `npx knex migrate:rollback --knexfile db/knexfile.js` drops `tbl_SubmissionQueue` |

Rollback is safe because:
- The queue table has no foreign keys to existing tables
- No existing tables are modified
- The Flutter app without the queue wrapper falls back to the original concurrent behavior (known issue, but functional)

---

## 11. Risks and Mitigations

| # | Risk | Impact | Likelihood | Mitigation |
|---|------|--------|------------|------------|
| 1 | **Reaper doesn't fire** if no one polls and `setInterval` somehow stops | ACTIVE session blocks queue indefinitely | Very Low (persistent process) | `setInterval` runs reliably on physical server. Inline reaper in `check-turn` is the secondary mechanism. Admin can manually expire via `/queue/status` + direct DB update. PM2 auto-restart recovers the timer if the process crashes. |
| 2 | **Server restart** orphans active sessions | Queue blocked until process restarts and reaper runs | Low | PM2/systemd auto-restart. On startup, an immediate `expireStale()` call clears orphaned sessions before accepting new requests. |
| 3 | **Client app killed between ACTIVE and pipeline start** | 5-minute dead slot in queue | Low | Acceptable — 5 min is the worst case; subsequent reaper call clears it. |
| 4 | **Queue starvation** — one user submits 20 reports, holds slot for 4+ minutes | Other users wait | Medium | `ReportCount` is tracked; future enhancement can cap reports-per-session or implement fair scheduling. For v1, the 5-min timeout bounds the worst case. |
| 5 | **Knex pool exhaustion** from lock contention | 503 errors on all endpoints | Very Low | Lock hold time is < 50ms (single UPDATE statement). Pool max is 7; queue endpoints use 1 connection. Contention would require 7+ simultaneous check-turn calls, which is unlikely with 3s poll intervals. |
| 6 | **`OUTPUT` clause fails on tables with triggers** | Activation SQL errors | None (new table) | `tbl_SubmissionQueue` is a new table with no triggers. If triggers are added later, switch to `insertWithId` pattern (OUTPUT INTO temp table). |
| 7 | **Network partition** — client thinks it's ACTIVE but server expired it | Client sends pipeline calls that succeed (no server-side gate) | Medium | v1 accepts this — pipeline calls succeeding without queue validation is the current behavior anyway. Future: add a middleware that checks `SessionId` header against ACTIVE status on every `/sendDARForm/*` call. |
| 8 | **Clock skew** between app server and DB server | Reaper timeout calculations may be off | Very Low | Using `SYSUTCDATETIME()` (database clock) for all timestamp comparisons, not application time. All timeout decisions are server-side SQL. |
| 9 | **Unique index violation** on re-enqueue after expiry | 409 error if EXPIRED row not yet visible | Very Low | The `UX_SubmissionQueue_UserPending` filter only covers `WAITING` and `ACTIVE` statuses; EXPIRED rows are excluded. User can re-enqueue immediately after expiry. |
| 10 | **Backward compatibility** — old app version without queue wrapper | Old clients bypass queue, reintroduce race conditions | Medium (during rollout) | Acceptable during transition. The queue does not gate the existing endpoints. Once all field devices are updated, the old path is eliminated. For strict enforcement, add server-side middleware (see Risk #7 future enhancement). |

---

## Appendix A: File Structure (Backend)

```
DAR_Middleware/
├── queue/
│   ├── routes/
│   │   └── index.js          # Express router: /enqueue, /check-turn, /complete, /fail, /status
│   ├── queueService.js       # tryActivateSession(), enqueue(), complete(), fail() logic
│   └── queueConfig.js        # Timeout and queue-depth constants from env
├── db/
│   └── migrations/
│       └── 20260414_create_tbl_submission_queue.js
└── server.js                  # Add: app.use('/queue', queueRoutes)
```

## Appendix B: File Structure (Frontend)

```
main_dar_app/lib/
├── app/
│   └── service/
│       └── queue/
│           ├── submission_queue_service.dart    # waitForTurn(), complete(), fail()
│           └── queue_exceptions.dart            # QueueFullException, QueueExpiredException, QueueTimeoutException
│   └── screen/
│       └── review/
│           └── widgets/
│               ├── action/
│               │   └── review_action.dart       # MODIFIED — wrap existing loop with queue
│               └── queue/
│                   └── queue_position_dialog.dart  # "Waiting for turn" UI
└── core/
    └── presentation/
        └── theme/
            └── config.dart                      # Add queue endpoint paths
```

## Appendix C: Configuration Reference

| Env Variable | Default | Used In |
|--------------|---------|---------|
| `QUEUE_ACTIVE_TIMEOUT_MIN` | `5` | queueConfig.js |
| `QUEUE_WAITING_TIMEOUT_MIN` | `15` | queueConfig.js |
| `QUEUE_MAX_DEPTH` | `10` | queueConfig.js |
| `QUEUE_ENABLED` | `true` | queueConfig.js — kill switch to bypass queue |

When `QUEUE_ENABLED=false`, the `/queue/enqueue` endpoint returns an immediate mock ACTIVE response with `sessionId: -1`, allowing the client to proceed without waiting. This provides a server-side kill switch without requiring a client update.
