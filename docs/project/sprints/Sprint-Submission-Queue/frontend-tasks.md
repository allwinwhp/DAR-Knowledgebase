# Frontend Tasks — Submission Queue Feature

**Branch**: `feature/submission-queue` (from `MDAG-331`)
**Repos**: `main_dar_app` (Flutter) + `DAR_Middleware` (Node.js)
**Date**: 2026-04-14

---

## Task Summary

| ID | Task | Files | Effort | Depends On |
|----|------|-------|--------|------------|
| FE-SQ01 | Add queue endpoint constants to `Config` | `config.dart` | S | — |
| FE-SQ02 | Add queue UI string constants | `constants.dart` | S | — |
| FE-SQ03 | Create `QueueApiService` | `queue_api_service.dart` (new) | M | FE-SQ01 |
| FE-SQ04 | Create `QueueSessionService` (orchestrator) | `queue_session_service.dart` (new) | M | FE-SQ03 |
| FE-SQ05 | Create queue waiting dialog widget | `queue_waiting_dialog.dart` (new) | M | FE-SQ02 |
| FE-SQ06 | Modify `review_action.dart` — wrap send flow | `review_action.dart` | L | FE-SQ04, FE-SQ05 |
| FE-SQ07 | Manual QA — end-to-end queue flow | — | M | FE-SQ06 |

---

## FE-SQ01 — Add Queue Endpoint Constants

**Effort**: S (Small)
**Dependencies**: None
**File**: `lib/core/presentation/theme/config.dart`

Add the four queue endpoints at the end of the existing endpoint list, before the closing brace of the `Config` class.

### Code — config.dart (append before closing `}`)

```dart
  //SUBMISSION QUEUE
  static const String queueEnqueueEndpoint = "/queue/enqueue";
  static const String queueCheckTurnEndpoint = "/queue/check-turn";
  static const String queueCompleteEndpoint = "/queue/complete";
  static const String queueFailEndpoint = "/queue/fail";
```

**Notes**:
- Uses `static const String` to match the pattern of the newer sync endpoints (lines 132–196 of the current file).
- Paths must match the middleware router mounts exactly.

---

## FE-SQ02 — Add Queue UI String Constants

**Effort**: S (Small)
**Dependencies**: None
**File**: `lib/core/presentation/theme/constants.dart`

Add queue-specific dialog strings after the existing `completedSubtitle` block (line 296).

### Code — constants.dart (append after `completedSubtitle`)

```dart
  // QUEUE DIALOG MESSAGES
  static String queueJoiningTitle = "Joining Queue";
  static String queueJoiningSubtitle = "Requesting your place in line...";
  static String queueWaitingTitle = "Waiting in Queue";
  static String queueWaitingSubtitle = "Your reports will be sent when it's your turn.";
  static String queuePositionPrefix = "You are #";
  static String queuePositionSuffix = " in line";
  static String queueYourTurnTitle = "It's Your Turn!";
  static String queueYourTurnSubtitle = "Sending your reports now...";
  static String queueTimeoutMessage =
      "Queue timed out. Please try again later.";
  static String queueCancelledMessage = "Submission cancelled.";
  static String queueFailedMessage =
      "Something went wrong while waiting in queue. Please try again.";
```

---

## FE-SQ03 — Create `QueueApiService`

**Effort**: M (Medium)
**Dependencies**: FE-SQ01
**File**: `lib/app/service/queue/queue_api_service.dart` (new)

Thin HTTP layer — four static methods matching the middleware endpoints. Follows the same pattern as `BaggingApiService` (`http.post`, `Config.baseURL + Config.xxxEndpoint`, JSON encode/decode).

### Code — queue_api_service.dart

```dart
import 'dart:convert';
import 'dart:developer';

import 'package:http/http.dart' as http;
import 'package:main_dar_app/core/presentation/theme/config.dart';

class QueueApiService {
  /// Enqueue the current user. Returns the full queue entry JSON (ticketId, position, etc.)
  /// or throws on non-200 responses.
  static Future<Map<String, dynamic>> enqueue(int userId) async {
    final url = Config.baseURL + Config.queueEnqueueEndpoint;
    log('[QueueApiService] POST $url  userId=$userId');

    final response = await http.post(
      Uri.parse(url),
      headers: {"Content-Type": "application/json"},
      body: jsonEncode({"userId": userId}),
    );

    if (response.statusCode == 200 || response.statusCode == 201) {
      final data = jsonDecode(response.body) as Map<String, dynamic>;
      log('[QueueApiService] enqueue response: $data');
      return data;
    }

    throw http.Response(response.body, response.statusCode);
  }

  /// Poll the queue to check position / whether it's this user's turn.
  /// Expected response shape: { "isMyTurn": bool, "position": int, "ticketId": "..." }
  static Future<Map<String, dynamic>> checkTurn(String ticketId) async {
    final url = Config.baseURL + Config.queueCheckTurnEndpoint;
    log('[QueueApiService] POST $url  ticketId=$ticketId');

    final response = await http.post(
      Uri.parse(url),
      headers: {"Content-Type": "application/json"},
      body: jsonEncode({"ticketId": ticketId}),
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body) as Map<String, dynamic>;
      log('[QueueApiService] checkTurn response: $data');
      return data;
    }

    throw http.Response(response.body, response.statusCode);
  }

  /// Signal that the user has finished submitting successfully.
  static Future<void> complete(String ticketId) async {
    final url = Config.baseURL + Config.queueCompleteEndpoint;
    log('[QueueApiService] POST $url  ticketId=$ticketId');

    final response = await http.post(
      Uri.parse(url),
      headers: {"Content-Type": "application/json"},
      body: jsonEncode({"ticketId": ticketId}),
    );

    if (response.statusCode != 200) {
      log('[QueueApiService] complete FAILED: ${response.statusCode} ${response.body}');
      throw http.Response(response.body, response.statusCode);
    }
  }

  /// Signal that submission failed (allows the queue to release the slot).
  static Future<void> fail(String ticketId) async {
    final url = Config.baseURL + Config.queueFailEndpoint;
    log('[QueueApiService] POST $url  ticketId=$ticketId');

    final response = await http.post(
      Uri.parse(url),
      headers: {"Content-Type": "application/json"},
      body: jsonEncode({"ticketId": ticketId}),
    );

    if (response.statusCode != 200) {
      log('[QueueApiService] fail FAILED: ${response.statusCode} ${response.body}');
    }
  }
}
```

### Design decisions

| Decision | Rationale |
|----------|-----------|
| All methods are `static` | Consistent with `BaggingApiService`, `TotalsApiService`, etc. |
| `enqueue` / `checkTurn` return `Map<String, dynamic>` | Keeps the API layer contract-agnostic; the orchestrator (`QueueSessionService`) extracts the fields it needs. |
| `fail()` swallows errors silently | Failure to report failure is non-critical — the queue TTL will eventually expire the ticket on the server. |
| Non-200 responses throw `http.Response` | Matches the pattern used in `ErrorService.getUserFriendlyMessage()` which handles `http.Response` as an error type. |

---

## FE-SQ04 — Create `QueueSessionService` (Orchestrator)

**Effort**: M (Medium)
**Dependencies**: FE-SQ03
**File**: `lib/app/service/queue/queue_session_service.dart` (new)

This is the polling orchestrator. It calls `enqueue`, then enters a `checkTurn` polling loop (3 s interval, max 100 attempts = ~5 minutes). It accepts a callback so the UI can update the position number in real time.

### Code — queue_session_service.dart

```dart
import 'dart:async';
import 'dart:developer';

import 'package:main_dar_app/app/service/queue/queue_api_service.dart';

class QueueTimeoutException implements Exception {
  final String message;
  QueueTimeoutException([this.message = 'Queue wait timed out.']);
  @override
  String toString() => 'QueueTimeoutException: $message';
}

class QueueCancelledException implements Exception {
  @override
  String toString() => 'QueueCancelledException';
}

class QueueSession {
  final String ticketId;
  QueueSession({required this.ticketId});
}

class QueueSessionService {
  static const int _pollIntervalMs = 3000;
  static const int _maxAttempts = 100;

  /// Enqueue, poll until it's our turn, then return the session.
  ///
  /// [userId]             — the logged-in user's numeric ID.
  /// [onPositionUpdate]   — called every poll with the current position (1-based).
  /// [cancelToken]        — set the .value to true from UI to abort the wait.
  static Future<QueueSession> enqueueAndWaitForTurn({
    required int userId,
    required void Function(int position) onPositionUpdate,
    required CancelToken cancelToken,
  }) async {
    // Step 1: Enqueue
    final enqueueResult = await QueueApiService.enqueue(userId);
    final ticketId = enqueueResult['ticketId']?.toString() ?? '';

    if (ticketId.isEmpty) {
      throw Exception('Enqueue failed — no ticketId returned.');
    }

    // Immediately report initial position
    final initialPosition = enqueueResult['position'] as int? ?? 0;
    onPositionUpdate(initialPosition);

    // Step 2: Poll
    for (int attempt = 0; attempt < _maxAttempts; attempt++) {
      if (cancelToken.isCancelled) {
        // Best-effort: tell the server we're leaving
        await QueueApiService.fail(ticketId);
        throw QueueCancelledException();
      }

      await Future.delayed(const Duration(milliseconds: _pollIntervalMs));

      if (cancelToken.isCancelled) {
        await QueueApiService.fail(ticketId);
        throw QueueCancelledException();
      }

      final checkResult = await QueueApiService.checkTurn(ticketId);
      final isMyTurn = checkResult['isMyTurn'] as bool? ?? false;
      final position = checkResult['position'] as int? ?? 0;

      onPositionUpdate(position);

      if (isMyTurn) {
        log('[QueueSessionService] It is our turn! ticketId=$ticketId');
        return QueueSession(ticketId: ticketId);
      }
    }

    // Timed out — release the slot
    await QueueApiService.fail(ticketId);
    throw QueueTimeoutException();
  }
}

/// Lightweight cancel token that the UI can flip to abort polling.
class CancelToken {
  bool _cancelled = false;
  bool get isCancelled => _cancelled;
  void cancel() => _cancelled = true;
}
```

### Design decisions

| Decision | Rationale |
|----------|-----------|
| 3 s poll interval | Balances responsiveness with server load; matches spec. |
| 100 max attempts (~5 min) | Prevents infinite waits. Users on slow queues can retry. |
| `CancelToken` pattern | Allows the cancel button in the dialog to abort the loop without tearing down the widget tree. |
| Custom exception types | Lets `review_action.dart` distinguish timeout vs cancel vs network error for appropriate UI feedback. |
| `onPositionUpdate` callback | Decouples UI updates from the polling logic — the dialog can `setState` on each call. |

---

## FE-SQ05 — Create Queue Waiting Dialog Widget

**Effort**: M (Medium)
**Dependencies**: FE-SQ02
**File**: `lib/core/presentation/widgets/common/dialogs/queue_waiting_dialog.dart` (new)

Follows the same `showDialog` + `StatefulBuilder` pattern used in `BaggingDialogService.showProgressDialog()`, but with queue-specific content.

### Code — queue_waiting_dialog.dart

```dart
import 'package:flutter/material.dart';
import 'package:gap/gap.dart';
import 'package:main_dar_app/core/presentation/theme/constants.dart';
import 'package:main_dar_app/core/presentation/theme/sizes.dart';
import 'package:main_dar_app/core/presentation/widgets/ui/text/styled_text.dart';

class QueueWaitingDialog {
  static bool _isVisible = false;
  static StateSetter? _dialogSetState;
  static String _title = "";
  static String _subtitle = "";
  static int _position = 0;
  static VoidCallback? _onCancel;

  static void show(
    BuildContext context, {
    required String title,
    required String subtitle,
    int position = 0,
    VoidCallback? onCancel,
  }) {
    if (_isVisible) return;

    _isVisible = true;
    _title = title;
    _subtitle = subtitle;
    _position = position;
    _onCancel = onCancel;

    showDialog(
      barrierDismissible: false,
      context: context,
      builder: (context) {
        return StatefulBuilder(
          builder: (context, setState) {
            _dialogSetState = setState;
            return AlertDialog.adaptive(
              content: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Gap(AppSizes.hugeSize),
                  const CircularProgressIndicator(),
                  Gap(AppSizes.hugeSize),
                  StyledHeadlineMedium(_title),
                  Gap(AppSizes.lgSize),
                  StyledBodyMedium(_subtitle),
                  Gap(AppSizes.lgSize),
                  if (_position > 0)
                    StyledHeadlineMedium(
                      '${AppConstants.queuePositionPrefix}$_position${AppConstants.queuePositionSuffix}',
                    ),
                  Gap(AppSizes.hugeSize),
                  if (_onCancel != null)
                    TextButton(
                      onPressed: _onCancel,
                      child: const Text('Cancel'),
                    ),
                ],
              ),
            );
          },
        );
      },
    );
  }

  static void updatePosition(int position) {
    _position = position;
    if (_position == 0) {
      _title = AppConstants.queueYourTurnTitle;
      _subtitle = AppConstants.queueYourTurnSubtitle;
    }
    _dialogSetState?.call(() {});
  }

  static void updateTitle(String title, String subtitle) {
    _title = title;
    _subtitle = subtitle;
    _dialogSetState?.call(() {});
  }

  static void hide(BuildContext context) {
    if (_isVisible) {
      Navigator.of(context).pop();
      _isVisible = false;
      _dialogSetState = null;
      _onCancel = null;
    }
  }

  static bool get isVisible => _isVisible;
}
```

### UI/UX Notes

| Aspect | Detail |
|--------|--------|
| **Visual layout** | Identical to the existing progress dialogs (spinner → title → subtitle → status line). Supervisors in the field are already familiar with this pattern. |
| **Position display** | Shows "You are #3 in line" when `position > 0`. When `position == 0` (your turn) it swaps to the "It's Your Turn!" messaging via `updatePosition(0)`. |
| **Cancel button** | Optional. Visible during queue wait phase. Hidden once the actual send pipeline starts. Triggers `CancelToken.cancel()` via the callback. |
| **Barrier dismissible** | `false` — prevents accidental taps from dismissing. Cancel is the explicit opt-out. |
| **Transition** | When the turn arrives, the caller hides this dialog, then the existing per-report `showProgressDialog()` takes over. The user sees a seamless handoff: queue dialog → report progress dialog → done. |
| **Timeout** | If `QueueTimeoutException` fires, the dialog is hidden and a standard `AlertDialog` error is shown with `queueTimeoutMessage`. |

---

## FE-SQ06 — Modify `review_action.dart` — Wrap Send Flow with Queue

**Effort**: L (Large)
**Dependencies**: FE-SQ04, FE-SQ05
**File**: `lib/app/screen/review/widgets/action/review_action.dart`

This is the key integration. The existing pipeline inside `sendToServerButtonClicked()` is preserved verbatim — we wrap it with queue enqueue/wait/complete/fail phases.

### BEFORE (current `sendToServerButtonClicked`, simplified)

```dart
Future<void> sendToServerButtonClicked() async {
  // 1. Validate
  final validationResult = await SubmissionValidationService.validateBeforeSubmission(context);
  if (validationResult == null) return;

  // 2. Confirm
  final confirmed = await SubmissionValidationService.showSubmissionConfirmationDialog(context);
  if (!confirmed) return;

  // 3. Build report lists
  List<IndividualReport> individualReports = ...;
  List<GroupReport> groupReports = ...;

  // 4. Send individual reports (loop with dialog services)
  // 5. Send group reports (loop with dialog services)
  // 6. Calculate batch totals
}
```

### AFTER (wrapped with queue)

```dart
import 'package:main_dar_app/app/service/queue/queue_session_service.dart';
import 'package:main_dar_app/app/service/queue/queue_api_service.dart';
import 'package:main_dar_app/core/presentation/widgets/common/dialogs/queue_waiting_dialog.dart';

// (add these imports at the top of the file)

// Inside _ReviewActionState:

Future<void> sendToServerButtonClicked() async {
  // ── Pre-queue validation (unchanged) ──────────────────
  final validationResult =
      await SubmissionValidationService.validateBeforeSubmission(context);

  if (validationResult == null) return;

  final confirmed =
      await SubmissionValidationService.showSubmissionConfirmationDialog(context);

  if (!confirmed) return;

  // ── PHASE 1: Queue — enqueue and wait for turn ────────
  QueueSession? queueSession;
  final cancelToken = CancelToken();

  try {
    QueueWaitingDialog.show(
      context,
      title: AppConstants.queueJoiningTitle,
      subtitle: AppConstants.queueJoiningSubtitle,
      onCancel: () {
        cancelToken.cancel();
      },
    );

    queueSession = await QueueSessionService.enqueueAndWaitForTurn(
      userId: validationResult.userId,
      onPositionUpdate: (position) {
        QueueWaitingDialog.updateTitle(
          AppConstants.queueWaitingTitle,
          AppConstants.queueWaitingSubtitle,
        );
        QueueWaitingDialog.updatePosition(position);
      },
      cancelToken: cancelToken,
    );

    // Turn arrived — dismiss the queue dialog
    if (context.mounted) {
      QueueWaitingDialog.hide(context);
    }
  } on QueueCancelledException {
    if (context.mounted) QueueWaitingDialog.hide(context);
    return;
  } on QueueTimeoutException {
    if (context.mounted) {
      QueueWaitingDialog.hide(context);
      _showQueueErrorDialog(context, AppConstants.queueTimeoutMessage);
    }
    return;
  } catch (e) {
    if (context.mounted) {
      QueueWaitingDialog.hide(context);
      _showQueueErrorDialog(context, AppConstants.queueFailedMessage);
    }
    return;
  }

  // ── PHASE 2: Send pipeline (existing logic, unchanged) ─
  try {
    List<IndividualReport> individualReports = reviewProvider.completedReports
        .where((report) => !report.isGroup && report.individualReport != null)
        .map((report) => report.individualReport!)
        .toList();

    List<GroupReport> groupReports = reviewProvider.completedReports
        .where((report) => report.isGroup && report.groupReport != null)
        .map((report) => report.groupReport!)
        .toList();

    final Map<String, Future<String?> Function(BuildContext, IndividualReport, int, String)>
        individualDialogServices = {
      AppOperations.bagging: BaggingDialogService.sendDialogTesting,
      AppOperations.budCapping: BudCappingDialogService.sendDialogTesting,
      AppOperations.defloweringDefingering:
          DefloweringDefingeringDialogService.sendDialogTesting,
      AppOperations.propping: ProppingDialogService.sendDialogTesting,
      AppOperations.handTubing: HandTubingDialogService.sendDialogTesting,
      AppOperations.leafPruningFOR:
          LeafPruningFORDialogService.sendDialogTesting,
      AppOperations.budBunchSpray: BudBunchSprayDialogService.sendDialogTesting,
      AppOperations.budInjection: BudInjectionDialogService.sendDialogTesting,
      AppOperations.chemicalMixing:
          ChemicalMixingDialogService.sendDialogTesting,
      AppOperations.weedSpray: WeedsprayDialogService.sendDialogTesting,
      AppOperations.survey: SurveyDialogService.sendDialogTesting,
      AppOperations.suckerPruning: SuckerPruningDialogService.sendDialogTesting,
      AppOperations.gouging: GougingDialogService.sendDialogTesting,
      AppOperations.genServ: GenServDialogService.sendDialog,
    };

    final Map<String, Future<String?> Function(BuildContext, GroupReport, int, String)>
        groupDialogServices = {
      AppOperations.cpms: CPMSDialogService.sendDialogTesting,
      AppOperations.fertilizer: FertilizerDialogService.sendDialogTesting,
      AppOperations.erad: EradDialogService.sendDialogTesting,
      AppOperations.popcount: PopCountDialogService.sendDialogTesting,
      AppOperations.plantingReplanting:
          PlantingReplantingDialogService.sendDialogTesting,
      AppOperations.genServGroup: GenServGroupDialogService.sendDialogTesting,
    };

    Set<String> batchIds = {};
    bool hasError = false;

    for (var report in individualReports) {
      final operationName = report.doctype?.operation?.name;
      if (operationName == null) continue;

      final dialogService = individualDialogServices[operationName];

      if (dialogService != null) {
        String? batchId = await dialogService(
          context,
          report,
          validationResult.userId,
          validationResult.username,
        );

        if (batchId == null) {
          hasError = true;
          break;
        }

        batchIds.add(batchId);
        await reviewProvider.refetchCompletedReports();
        await homeProvider.refetchReports();
      }
    }

    if (!hasError) {
      for (var groupReport in groupReports) {
        final operationName = groupReport.doctype?.operation?.name;
        if (operationName == null) continue;

        final dialogService = groupDialogServices[operationName];

        if (dialogService != null) {
          String? batchId = await dialogService(
            context,
            groupReport,
            validationResult.userId,
            validationResult.username,
          );

          if (batchId == null) {
            hasError = true;
            break;
          }

          batchIds.add(batchId);
          await reviewProvider.refetchCompletedReports();
          await homeProvider.refetchReports();
        }
      }
    }

    for (var batchId in batchIds) {
      await TotalsApiService.calculateBatchTotal(batchId);
    }

    // ── PHASE 3: Queue complete ─────────────────────────
    if (queueSession != null) {
      await QueueApiService.complete(queueSession.ticketId);
    }
  } catch (e) {
    // ── Queue fail on any pipeline error ────────────────
    if (queueSession != null) {
      await QueueApiService.fail(queueSession.ticketId);
    }
    rethrow;
  }
}

/// Shows a simple error dialog for queue-related failures.
Future<void> _showQueueErrorDialog(BuildContext context, String message) async {
  return showDialog(
    context: context,
    builder: (context) => AlertDialog.adaptive(
      title: const Text("Queue Error"),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => Navigator.of(context).pop(),
          child: const Text("OK"),
        ),
      ],
    ),
  );
}
```

### Change summary

| Section | What changed |
|---------|-------------|
| **Imports** | +3 new imports (`QueueSessionService`, `QueueApiService`, `QueueWaitingDialog`) |
| **Before report lists** | New Phase 1 block: show queue dialog → `enqueueAndWaitForTurn()` → hide dialog |
| **Existing pipeline** | Wrapped in `try/catch` — on success calls `QueueApiService.complete()`, on error calls `QueueApiService.fail()` |
| **New method** | `_showQueueErrorDialog()` — queue-specific error dialog |
| **Existing dialog services** | Completely untouched. Each operation's `DialogService.sendDialogTesting()` still shows its own progress dialog as before. |

---

## FE-SQ07 — Manual QA (End-to-End)

**Effort**: M (Medium)
**Dependencies**: FE-SQ06 (all code complete)

### Test scenarios

| # | Scenario | Expected result |
|---|----------|-----------------|
| 1 | Single user, no queue contention | Enqueue → immediately `isMyTurn=true` → pipeline runs → complete |
| 2 | Two devices, same time | First user gets turn; second sees "You are #2 in line" → position drops to #1 → then pipeline |
| 3 | Cancel during wait | User taps Cancel → `fail()` called → dialog dismissed → no reports sent |
| 4 | Network loss during polling | `catch` fires → `fail()` called → queue error dialog shown |
| 5 | Queue timeout (simulate 100 polls) | `QueueTimeoutException` → dialog dismissed → timeout message shown |
| 6 | Pipeline error after turn acquired | Existing error dialog per-operation → `fail()` called → queue slot released |
| 7 | Success with mixed report types | Individual + group reports → all sent → `complete()` → batch totals calculated |

---

## File Tree (new/modified)

```
lib/
├── core/
│   └── presentation/
│       ├── theme/
│       │   ├── config.dart                          # MODIFIED (FE-SQ01)
│       │   └── constants.dart                       # MODIFIED (FE-SQ02)
│       └── widgets/
│           └── common/
│               └── dialogs/
│                   └── queue_waiting_dialog.dart     # NEW (FE-SQ05)
├── app/
│   ├── service/
│   │   └── queue/
│   │       ├── queue_api_service.dart               # NEW (FE-SQ03)
│   │       └── queue_session_service.dart            # NEW (FE-SQ04)
│   └── screen/
│       └── review/
│           └── widgets/
│               └── action/
│                   └── review_action.dart            # MODIFIED (FE-SQ06)
```

---

## Dependency Graph

```
FE-SQ01 ─────┐
              ├──► FE-SQ03 ──► FE-SQ04 ──┐
FE-SQ02 ──────────────────► FE-SQ05 ──────┼──► FE-SQ06 ──► FE-SQ07
                                           │
                                           └── (both FE-SQ04 and FE-SQ05
                                                must complete before FE-SQ06)
```

---

## Risk Register

| Risk | Impact | Mitigation |
|------|--------|------------|
| Middleware queue endpoints not ready | Blocks FE-SQ03 testing | Mock responses in `QueueApiService` behind a `kDebugMode` flag until middleware delivers |
| Poll interval too aggressive (3 s) | Server load under many concurrent supervisors | Make interval configurable; middleware can also return a `retryAfterMs` hint |
| Supervisor on unstable mobile data | Polling fails mid-wait | Each `checkTurn` failure is caught; after N consecutive failures, surface "connection lost" and auto-cancel |
| Dialog state leak if user navigates away | Orphan dialogs | `QueueWaitingDialog.hide()` called in `finally`; `_isVisible` guard prevents double-pop |
