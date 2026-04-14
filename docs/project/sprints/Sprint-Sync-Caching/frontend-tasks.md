# Sprint-Sync-Caching — Frontend Tasks

**Squad**: Frontend (Flutter)
**Repo**: `main_dar_app` · Branch: `MDAG-339`
**Goal**: Reduce sync time by parallelizing API calls and converting per-item Hive writes to batch `addAll()`.

---

## Task Summary

| ID | Description | File(s) | Size | Depends On |
|----|-------------|---------|------|------------|
| FE-SC01 | Parallelize sync calls in `_syncData()` using `Future.wait` groups | `home_sync.dart` | L | FE-SC08, FE-SC09 |
| FE-SC02 | Batch Hive writes in `FruitCareRepository` | `fruit_care_repository.dart` | M | — |
| FE-SC03 | Batch Hive writes in `OtherOperationsRepository` | `other_operations_repository.dart` | M | — |
| FE-SC04 | Batch Hive writes in `UserRepository` | `user_repository.dart` | S | — |
| FE-SC05 | Batch Hive writes in `AccountRepository` | `account_repository.dart` | S | — |
| FE-SC06 | Batch Hive writes in `BatchNoRepository` | `batch_no_repository.dart` | S | — |
| FE-SC07 | Batch Hive writes in `DocNoRepository` | `doc_no_repository.dart` | S | — |
| FE-SC08 | Redesign `SyncDialogService` for group-based progress | `sync_dialog_service.dart` | M | — |
| FE-SC09 | Add error-resilient `Future.wait` wrapper | `home_sync.dart` | S | — |
| FE-SC10 | Manual QA — measure sync time before/after | — | S | FE-SC01–09 |

---

## Sync Groups

| Group | Label | Calls | Count |
|-------|-------|-------|-------|
| **G1** | Reference Data | `syncHolidays`, `syncLocations`, `syncDiseaseTypes`, `syncEmployees`, `syncCostCenters` | 5 |
| **G2** | Materials & Operations | `syncVarieties`, `syncMaterials`, `syncChemMixings`, `syncMixingOperations`, `syncIncentives` | 5 |
| **G3** | Fertilizer Data | `syncFertTypes`, `syncFertilizers`, `syncFertilizerCostCenters`, `syncFertilizerMaterials` | 4 |
| **G4** | Spatial Data | `syncFruitCareData`, `syncOtherOperationsData` | 2 |
| **G5** | User & Auth Data | `syncUsers`, `syncAccounts`, `syncBatchNumbers`, `syncDocNumbers` | 4 |

---

## FE-SC09 — Error-Resilient Group Helper

```dart
Future<List<SyncOutcome>> _syncGroup(List<Future<dynamic>> futures) async {
  final results = await Future.wait(
    futures.map((f) => f.then(
      (value) => SyncOutcome(success: true, result: value),
      onError: (error, stack) {
        developer.log('Sync call failed: $error', stackTrace: stack);
        return SyncOutcome(success: false, error: error.toString());
      },
    )),
    eagerError: false,
  );
  final failures = results.where((r) => !r.success).length;
  if (failures > 0) {
    developer.log('Group completed with $failures/${results.length} failures');
  }
  return results;
}

class SyncOutcome {
  final bool success;
  final dynamic result;
  final String? error;
  const SyncOutcome({required this.success, this.result, this.error});
}
```

---

## FE-SC02 — Batch Hive Writes: FruitCareRepository

**Before** (3 loops):
```dart
for (var areaJson in areasJson) {
  final area = FruitCareArea.fromJson(areaJson);
  await areasBox.add(area);
}
```

**After**:
```dart
final areas = areasJson.map((j) {
  final area = FruitCareArea.fromJson(j);
  area.farmCode = j['FarmCode']?.toString();
  area.opCode = j['Opcode']?.toString();
  return area;
}).toList();
await areasBox.addAll(areas);
```

Apply same pattern to parcels and layouts boxes.

---

## FE-SC03 — Batch Hive Writes: OtherOperationsRepository

Same `add()` → `addAll()` conversion for 3 boxes: parcels, layouts, survey-layouts.

---

## FE-SC04–07 — Batch Hive Writes: User/Account/BatchNo/DocNo

**Before**:
```dart
for (final user in parsedUsers) {
  await usersBox.add(user);
}
```

**After**:
```dart
await usersBox.addAll(parsedUsers);
```

Same pattern for AccountRepository, BatchNoRepository, DocNoRepository.

---

## FE-SC08 — Progress Dialog Redesign

Replace 20-step sequential progress with 5-group model:

```dart
static const List<String> syncGroupNames = [
  'Reference Data',
  'Materials & Operations',
  'Fertilizer Data',
  'Spatial Data',
  'User & Auth Data',
];
```

Each group shows: spinner (in-progress), checkmark (completed), circle (pending).

---

## File Change Summary

| File | Change | Task |
|------|--------|------|
| `home_sync.dart` | Rewrite `_syncData()` with `Future.wait` groups + `_syncGroup()` helper | FE-SC01, FE-SC09 |
| `fruit_care_repository.dart` | Replace 3 `add()` loops → `addAll()` | FE-SC02 |
| `other_operations_repository.dart` | Replace 3 `add()` loops → `addAll()` | FE-SC03 |
| `user_repository.dart` | Replace `add()` loop → `addAll()` | FE-SC04 |
| `account_repository.dart` | Replace `add()` loop → `addAll()` | FE-SC05 |
| `batch_no_repository.dart` | Replace `add()` loop → `addAll()` | FE-SC06 |
| `doc_no_repository.dart` | Replace `add()` loop → `addAll()` | FE-SC07 |
| `sync_dialog_service.dart` | Group-based progress model | FE-SC08 |

---

## Implementation Order

```
FE-SC02 ──┐
FE-SC03 ──┤
FE-SC04 ──┤  (all independent — can be done in parallel)
FE-SC05 ──┤
FE-SC06 ──┤
FE-SC07 ──┘
              ↓
FE-SC08 ─────→ FE-SC09 ─→ FE-SC01 ─→ FE-SC10
(dialog)       (helper)    (orchestrator)  (QA)
```
