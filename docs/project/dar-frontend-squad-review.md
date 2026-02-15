# main_dar_app (Flutter) — Frontend Squad Deep Review

**Reviewer**: Frontend Squad (DAR context). **Scope**: main_dar_app (Flutter). **Branch**: MDAG-339.

---

## 1. Executive Summary

The main_dar_app is a **Flutter** mobile app that provides the **input interface for farm supervisors** to create and submit **individual (solo)** and **gang/group** DAR reports. It uses **Hive** for local report storage, **Provider** for state, **go_router** for navigation, and **http** for REST calls to DAR Middleware. The app is feature-rich and aligns with the backend’s batch → header → details → accomplishments → materials flow. Findings below focus on correctness, API alignment, state management, and maintainability.

---

## 2. Stack & Structure

| Layer | Technology |
|-------|------------|
| UI | Flutter, Material |
| State | Provider, flutter_bloc (feature-level) |
| Routing | go_router |
| Local DB | Hive (individual_reports, group_reports, etc.) |
| HTTP | http package |
| Fonts | google_fonts |

**Notable structure**:

- **lib/app/screen/** — Screens: splash, login, home, operators, document, form, review, members, group, signup, log.
- **lib/app/feature/** — Operation-specific features: mps (bagging, bud capping, bud injection, deflowering, hand tubing, leaf pruning, propping, bud bunch spray), pdcm (survey, erad), plant_care (weedspray, fertilizer, chemical_mixing, cpms), ppm (gouging, sucker_pruning, popcount, planting_replanting), others (gen_serv).
- **lib/app/data/** — Hive models (report/individual, report/group, operator, supervisor, timekeeping, doctype, forms), API models, enums (doctypes, operations, cost centers).
- **lib/app/service/** — Login, report (individual/group, log), API services per operation (survey_api_service, erad_api_service, deflowering_defingering_api_service, etc.).
- **lib/core/** — Theme (config, constants), routes (router, path), widgets.

---

## 3. Findings

### 3.1 Positive

- **Report type split**: Individual vs Gang/Group is clear in UI (`AppConstants.individualButtonTitle`, `gangGroupButtonTitle`) and in data (`IndividualReport` vs `GroupReport`, separate Hive boxes and report services).
- **Journey**: Home → Create (Individual → Operators, or Gang/Group → Group) → Document/Form → Submit mirrors the backend flow (batch → hdr → dtls → accomp → materials).
- **Operation-specific API services**: Each operation has its own API service (e.g. SurveyApiService, EradApiService) that calls getExistingBatchId, createBatch, createHeader, createDetails, createAccomplishments, createMaterials, and totals utilities. Order of calls matches backend expectations.
- **Hive models**: IndividualReport (supervisor, operator, timekeeping, doctype, commonBaseForm) and operation-specific forms (e.g. SurveyForm, BudCappingForm) give a clear local model for each report type.
- **Validation**: Report completeness uses `BaseService.isTimekeepingComplete` and operation-specific `_isCommonBaseFormComplete`; status (e.g. completed/incomplete) is updated when all required sections are filled.
- **Config centralization**: `Config` (config.dart) holds baseURL and all endpoint paths in one place; easy to switch environments if baseURL is later made configurable (e.g. flavor or env).

### 3.2 Gaps & Risks

1. **Base URL hardcoded**  
   `Config.baseURL = "http://localhost:3000"` is hardcoded. For device/emulator, Android often needs `10.0.2.2:3000` (commented in config). **Recommendation**: Use build flavors or environment config (e.g. .env or compile-time constants) so dev/staging/prod and local vs device can be set without code change.

2. **MPS endpoint path case**  
   In `config.dart`, MPS endpoints use **capital** `MPS`: e.g. `"/sendDARForm/MPS/tblDARhdr"`. The middleware mounts routes with **lowercase** `mps`: e.g. `router.use("/mps/tblDARhdr", ...)`. Express routes are case-sensitive on some hosts; this can cause 404s. **Recommendation**: Use lowercase `mps` in Config to match middleware: `/sendDARForm/mps/tblDARhdr`, etc.

3. **getExistingBatchId endpoint path**  
   Config uses `getExistingBatchIdEndpoint = "/sendDARForm/utilities/getExistingBatchId"`. Middleware route is `router.use("/utilities/getExistingBatchId", ...)` under sendDARForm, so full path is `/sendDARForm/utilities/getExistingBatchId` — correct. No change needed; keep as-is.

4. **Erad details endpoint**  
   `eraddtlsEndpoint = "/sendDARForm/genserv3/tblDARdtls"` — Erad uses genserv3 for dtls. Confirm with backend/product that this is intentional (shared table or legacy); document in form specs if so.

5. **Erad materials endpoint**  
   `eradmaterialEndpoint = "/sendDARForm/survey/tblDARmaterials"` — Erad materials point to survey materials. Same as above: confirm intent and document.

6. **Duplicate logic across operation services**  
   Each operation’s API service repeats the same pattern (check batch → create batch → create header → create details → create accomplishments → create materials → totals). Small differences (endpoints, body mapping). Consider a shared “DAR submit pipeline” that takes operation-specific endpoint config and body mappers to reduce duplication and drift.

7. **Error handling and retry**  
   API calls use `http.post`/`http.get`; status codes are checked (e.g. 201, 200) but there is no visible centralized error handling (e.g. toast/snackbar, retry, offline queue). Failed partial submissions (e.g. batch created, header fails) can leave backend in an inconsistent state; app may not reflect it. **Recommendation**: Add user-visible error feedback, and consider idempotency or rollback guidance when a step in the pipeline fails.

8. **Auth token not observed**  
   Login returns user/employee info; no JWT or token was seen in the reviewed code. If the backend adds auth later, the app will need to send the token (e.g. Authorization header) on all requests to sendDARForm and report endpoints. **Recommendation**: Prepare for auth (e.g. store token after login, inject into a shared HTTP client/interceptor).

9. **Hive and report lifecycle**  
   Reports are stored in Hive; successful submit may delete or mark report. Ensure that “delete after send” or “move to log” is consistent and that the user can tell which reports are pending vs sent. Log and Review screens should reflect this clearly.

### 3.3 Consistency & Maintainability

- **Doctype and OpCode**: Flutter uses enums (e.g. `AppDoctypes.survey`) and cost centers (e.g. `AppCostCenters.survey`). These must stay in sync with backend `Doctype` and operation config (`tbl_OperationConfig`). Document the mapping (e.g. in dar-user-journey-and-form-specifications.md or a shared doc).
- **Form model proliferation**: Each operation has its own form model and commonBaseForm subtype (SurveyForm, BaggingForm, etc.). Good for type safety; ensure API body mapping (toJson/toApi) stays aligned with backend contract (field names, types). A single “API contract” doc (or generated types from OpenAPI) would help.
- **Routing**: Routes are string-based (RoutePath.*). Form screen selection (which operation form to show) is likely driven by report doctype or navigation args; ensure deep links or back stack don’t open the wrong form type.

---

## 4. User Journey Alignment with Backend

- **Individual report**: Create → Operators (individual list) → save to server via `/report/save-report` (individual_operators) → create report in Hive → fill form → submit (batch → hdr → dtls → accomp → materials). Matches backend.
- **Group report**: Create → Group (groups + operators) → save via `/report/save-report` (group_operators with groupId) → create GroupReport → fill form → submit. Matches backend.
- **Get operators**: `/report/get-report` with supervisorId → individual_operators, group_operators. Used to repopulate operator/group lists. Matches backend.

---

## 5. Recommendations (Prioritized)

1. **High**: Fix MPS endpoint case in Config: use `/sendDARForm/mps/...` (lowercase) to match middleware.
2. **High**: Make baseURL configurable (flavors, env, or build config) for local vs device vs staging/prod.
3. **Medium**: Add centralized API error handling and user-visible feedback (and consider retry or offline queue for submit).
4. **Medium**: Prepare for auth: store token after login and send it on all API requests once backend enforces auth.
5. **Low**: Consider a shared DAR submit pipeline (operation-agnostic) parameterized by endpoints and mappers to reduce duplication.
6. **Low**: Document Doctype/OpCode and cost-center mapping between Flutter and backend; document Erad’s use of genserv3 dtls and survey materials if intentional.

---

## 6. References

- [dar-user-journey-and-form-specifications.md](dar-user-journey-and-form-specifications.md) — user journey and form specs.
- [dar-backend-squad-review.md](dar-backend-squad-review.md) — middleware review.
- [dar-system-context.md](dar-system-context.md) — DAR context.
- main_dar_app: `lib/core/presentation/theme/config.dart`, `lib/core/routes/router.dart`, `lib/app/screen/home/widgets/create/home_create.dart`, `lib/app/data/hive/report/individual/model/individual_report.dart`, `lib/app/feature/pdcm/survey/service/api/survey_api_service.dart`, `lib/app/service/report/individual/individual_report_service.dart`.
