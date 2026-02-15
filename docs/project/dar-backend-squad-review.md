# DAR Middleware — Backend Squad Deep Review

**Reviewer**: Backend Squad (DAR context). **Scope**: DAR_Middleware (Node.js, Express, Knex, MSSQL). **Branch**: MDAG-339.

---

## 1. Executive Summary

The DAR Middleware is a **REST API** layer between the Flutter app and the DAR MSSQL database. It handles **batch → header → details → accomplishments → materials** submission per operation type, **user auth** (login/register), **report** (operator) save/get, and **sync** endpoints for reference data. The architecture is consistent and the API surface is well-documented (Swagger). Findings below focus on correctness, safety, consistency, and maintainability.

---

## 2. Architecture & Stack

| Layer | Technology |
|-------|------------|
| Runtime | Node.js |
| Framework | Express |
| DB | MSSQL via Knex |
| Auth | bcryptjs (passwords), session via client |
| Docs | swagger-jsdoc, swagger-ui-express |

**Routes**:

- `/sendDARForm/*` — DAR form CRUD (batch, hdr, dtls, accomp, materials) per operation.
- `/user/*` — login, register, validate submission, assign user ID.
- `/report/*` — save-report, get-report, delete-report (supervisor operators: individual/group).
- `/api/*` — controller: sync (users, operation-config, mixing-operations, location, parcel, material, variety, fruit-care, fertilizer, etc.), hectarage, parcel, activity, erad, schema.

---

## 3. Findings

### 3.1 Positive

- **Swagger**: Endpoints are documented with request/response schemas; `/api-docs` is dynamic by host (ngrok-friendly).
- **Batch idempotency**: Creating a batch checks for existing `(Entrydate, Doctype, userid)` and returns existing `Batch_id` without incrementing `tblbatchno`, avoiding duplicate batches.
- **Document number**: Per-user document numbers via `tbldocno`; header creation assigns `DocumentNo` consistently.
- **Validation**: `validateOperation.js` centralizes max accomplishments, max materials, and operation-type checks (bagging, weedspray manual, sucker prunning, chemical mixing). Accomplishment and material inserts use it before insert.
- **Auto-calculation**: `autoCalculate.js` updates header `Qty1` and batch `Accomp`/`AccompEntered` after accomplishment changes; logic is doctype-aware (GENSERV, PPM, PDCM, etc.).
- **CORS**: Configured for all origins and OPTIONS; headers applied on error and 404 responses.
- **Logging**: Winston logger used for request and error logging.
- **Migrations**: Knex migrations and seeds for sync tables and operation config; `MIGRATION_GUIDE.md` and `IMPLEMENTATION_STATUS.md` exist.

### 3.2 Gaps & Risks

1. **getExistingBatchId — Swagger vs implementation**  
   Swagger describes **GET** with query params `PHCode`, `FarmCode`, `Entrydate`, `Ptype`. Implementation is **POST** with body `Entrydate`, `Doctype`, `userid`. Flutter already uses POST and the correct params; Swagger should be updated to match to avoid confusion.

2. **Input validation**  
   Most routes trust `req.body` and pass it to the DB. There is no centralized request validation (e.g. Joi, express-validator). Risk: invalid or oversized payloads, type confusion (e.g. string vs int for IDs). **Recommendation**: Add validation layer for required fields and types for batch, hdr, dtls, accomp, materials, and report endpoints.

3. **Error handling**  
   Try/catch returns 500 with `error.message` in non-production. No structured error codes or validation-error vs server-error distinction. **Recommendation**: Normalize error responses (code, message, optional details) and avoid leaking stack or internal messages in production.

4. **Auth on sendDARForm and report**  
   `/user/login` returns user/employee info, but there is no middleware that attaches or validates a token/session on `/sendDARForm/*` or `/report/*`. Anyone who can reach the API can submit DAR data or overwrite operator lists if they know or guess IDs. **Recommendation**: Add auth middleware (e.g. JWT or session) and enforce it on all state-changing and sensitive read endpoints.

5. **SQL injection**  
   Knex is used with parameterized queries in most places. `getExistingBatchId` uses `db.raw(query, [Entrydate, Doctype, userid])` — safe. Spot-check of other raw usage recommended; avoid concatenating user input into raw SQL.

6. **Concurrency**  
   Batch and document number increments are read-then-write (tblbatchno, tbldocno). Under high concurrency for the same user, two requests could get the same batchno/docno before either updates. **Recommendation**: Use DB-level increment (e.g. `increment`) or serializable transaction/single-statement update to avoid race conditions.

7. **Report delete-report**  
   Confirmed: `delete-report` deletes only where `supervisorId` matches the request body; correctly scoped so one supervisor cannot delete another’s operator records.

8. **Doctype / OpCode consistency**  
   Operation config and validation use string and numeric doctypes in places (e.g. `doctype === '08' || doctype === 8`). Seeds use string codes (e.g. `'WEED'`, `'FERT'`). Ensure Flutter and backend agree on Doctype and OpCode enums and that `tbl_OperationConfig` matches what the app sends.

### 3.3 Consistency & Maintainability

- **Operation duplication**: Each operation (mps, survey, fertilization, …) has its own `tblDARhdr`, `tblDARdtls`, `tblDARaccomp` (and sometimes `tblDARmaterials`) route file. Logic is repeated with small variations. Consider shared handlers parameterized by operation/table name to reduce duplication and drift.
- **Table naming**: Physical tables are shared (e.g. `tblDARhdr`) with operation implied by route; some operations may use different tables (e.g. bag-week). Document which route writes to which table for each operation.
- **Connection config**: `db/knex_config.js` uses a single `development` env; production and test environments are not shown. Ensure `NODE_ENV` and env-specific configs are used in production.

---

## 4. API Contract Summary (for Frontend)

- **Batch**: POST getExistingBatchId (body: `Entrydate`, `Doctype`, `userid`) → `Batch_id`. POST batch (body: batch fields) → `id.Batch_id`.
- **Header**: POST `/<operation>/tblDARhdr` (body: `Batch_id`, …) → `id.Hdr_id`.
- **Details**: POST `/<operation>/tblDARdtls` (body: `Hdr_id`, `Batch_id`, …).
- **Accomplishments**: POST `/<operation>/tblDARaccomp` (body: `Hdr_id`, `Batch_id`, …); max accomplishments enforced.
- **Materials**: POST `/<operation>/tblDARmaterials` where applicable; max materials enforced.
- **Report**: POST `/report/save-report` (body: `supervisorId`, `individual_operators`, `group_operators`). POST `/report/get-report` (body: `supervisorId`) → `individual_operators`, `group_operators`.
- **Login**: POST `/user/login` (body: `Username`, `Password`) → user/employee info including `userId`.

---

## 5. Recommendations (Prioritized)

1. **High**: Add authentication/authorization middleware to `/sendDARForm/*`, `/report/*`, and sensitive `/api/*` endpoints.
2. **High**: Align Swagger with actual HTTP method and body for getExistingBatchId (POST, body).
3. **Medium**: Add request validation (required fields, types) for all public POST bodies.
4. **Medium**: Harden batch and document number generation for concurrency (atomic increment or transaction).
5. **Low**: Normalize error responses and avoid leaking internal errors in production.
6. **Low**: Consider shared route/handler pattern for operation-specific DAR tables to reduce duplication.

---

## 6. References

- [dar-user-journey-and-form-specifications.md](dar-user-journey-and-form-specifications.md) — user journey and form specs.
- [dar-system-context.md](dar-system-context.md) — DAR context and repos.
- DAR_Middleware: `server.js`, `sendDARForm/routes`, `controller/routes`, `report/routes`, `sendDARForm/utilities/validateOperation.js`, `sendDARForm/utilities/autoCalculate.js`.
