# DAR Deployment MOP — Method of Procedure & Checklist

**Purpose**: Checklist for deploying DAR (UAT) — Middleware, UAT database, and Flutter APK. Use with the Delivery Orchestrator and team roles. **Sign-off**: PM confirms go-live; DevOps approves deployment steps; QA Lead signs off UAT; **Business Unit** signs off scope (pre-implementation) and UAT outcome (post-implementation).

**References**: [dar-system-context.md](../project/dar-system-context.md) | [dar-user-journey-and-form-specifications.md](../project/dar-user-journey-and-form-specifications.md) | [delivery-orchestrator](../../agents/delivery-orchestrator.md)

---

## Deployment overview

| Phase | Owner (primary) | Outcome |
|-------|------------------|--------|
| **Pre-implementation** | MDAG IT + Delivery Team + DevOps | PROD backup; UAT DB restored; DB scripts run; middleware port and APK base URL agreed; UAT APK built. |
| **Implementation** | Delivery Team + DevOps | Middleware running on UAT; users install APK. |
| **Post-implementation** | QA Lead + SM + PM | UAT executed with test user; sign-off and handover. |

---

## Pre-implementation

### 1. Database backup & restore (MDAG IT)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 1.1 | Create full backup of current **DAR PROD** database (MSSQL). | MDAG IT | ☐ | Document backup location, filename, and timestamp. |
| 1.2 | Verify backup integrity (e.g. restore to temp or checksum). | MDAG IT | ☐ | |
| 1.3 | Restore the PROD backup to **DAR UAT** database. | MDAG IT | ☐ | Confirm UAT DB name and server. |
| 1.4 | Confirm UAT DB is accessible (login, list tables). | MDAG IT / Delivery | ☐ | Share UAT connection details (server, port, database name, user) with Delivery Team securely. |

### 2. Identify exact migrations: PROD DB vs DEV DB (Delivery Team — Backend / Architect)

**Objective**: Before running any migrations on UAT, determine exactly which Knex migrations (and equivalent SQL scripts) are missing on PROD so that only the required scripts are run on UAT after restore. UAT will be a copy of PROD; we then apply the delta so UAT matches DEV / application requirements.

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 2.1 | **Compare PROD DB with DEV DB**: Capture schema of **DAR PROD** (tables, columns, indexes, FKs) and **DAR DEV** (same). Use SSMS (e.g. Script Database As → CREATE, or INFORMATION_SCHEMA / sys tables) or a schema-compare tool. | Backend / Architect | ☐ | Document tool and output location (e.g. schema_PROD.sql, schema_DEV.sql, or diff report). |
| 2.2 | **Compare with migration set in repo**: List all migrations in DAR_Middleware **dev** branch `db/migrations/` (see **MIGRATION_GUIDE.md** for order). Cross-check which of these are already reflected in PROD (e.g. table/column already exists) vs missing. | Backend / Architect | ☐ | If PROD has no `knex_migrations` table, infer from schema only (tables/columns present). |
| 2.3 | **Document exact migration list**: Produce a short list of migrations **to run on UAT** (post-restore): (a) Knex migration names (e.g. `20251118_add_userid_to_tblaccounts.js`, `20251118_create_sync_tables.js`, …), (b) corresponding `.sql` files for SSMS backup (same order as MIGRATION_GUIDE.md). | Backend / Architect | ☐ | Attach or link this list to the MOP (e.g. in Notes or appendix). |
| 2.4 | **Review and agree**: Architect or Backend Lead confirms the list is correct; no unnecessary or duplicate scripts. | Backend / Architect | ☐ | |

### 3. Database alignment (Delivery Team — Backend / Architect)

Run **only** the migrations identified in section 2. **Primary**: Knex. **Backup**: If Knex fails, run the equivalent **SQL scripts in SSMS** (see step 3.3).

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 3.1 | **Primary**: Run the **exact migrations identified in section 2** on **DAR UAT DB**. Option A: `npx knex migrate:latest --knexfile db/knexfile.js` (knex_config pointing to UAT) — Knex will run only migrations not yet in `knex_migrations`. Option B: If UAT has no knex_migrations, run only the scripts from the section-2 list in order. | Delivery Team | ☐ | Use UAT connection only. |
| 3.2 | Run seed data if required: `npx knex seed:run --knexfile db/knexfile.js` (e.g. operation config, mixing ops). | Delivery Team | ☐ | |
| 3.3 | **If Knex migrations don’t work**: Run the **exact .sql scripts from the section-2 list** in **SSMS** against **DAR UAT**, in the documented order. | Delivery Team / MDAG IT | ☐ | Execute in a single SSMS session; confirm each script completes before the next. |
| 3.4 | Verify schema: key tables/columns present per DEV (e.g. tblDARbatch, tblDARhdr, tbl_OperationConfig, tblsupervisor_operators, tblaccounts.userid, etc.). | Backend / Architect | ☐ | |

### 4. Middleware host & port (DevOps + MDAG IT)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 4.1 | Identify **server** (hostname or IP) where DAR Middleware will run for UAT. | DevOps / MDAG IT | ☐ | E.g. same DB server or app server. |
| 4.2 | Identify **port** on that server for the Middleware (e.g. 3000 or agreed port). | DevOps / MDAG IT | ☐ | Ensure firewall/security allows inbound on this port for UAT users/devices. |
| 4.3 | Document **UAT Middleware base URL** (e.g. `http://<host>:<port>` or `https://...`). | DevOps | ☐ | This URL will be used in the Flutter APK `Config.baseURL`. |

### 5. Flutter APK build (Delivery Team — Frontend)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 5.1 | Set **base URL** in main_dar_app to the identified UAT Middleware server and port (e.g. in `lib/core/presentation/theme/config.dart`: `Config.baseURL = "http://<host>:<port>"`). | Frontend | ☐ | No trailing slash. Use build flavor or env if available. |
| 5.2 | Build **release APK** (e.g. `flutter build apk --release` from main_dar_app). | Frontend | ☐ | |
| 5.3 | Store APK in agreed location (e.g. shared drive, link) for distribution to test users. | Delivery / PM | ☐ | |
| 5.4 | Document APK version and build date; confirm base URL in build. | Frontend / SM | ☐ | |

### 6. Pre-implementation sign-off

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 6.1 | PM confirms timeline and scope for deployment day. | PM | ☐ | |
| 6.2 | DevOps confirms port, host, and UAT DB access are ready. | DevOps | ☐ | |
| 6.3 | SM confirms checklist 1.1–5.4 complete before implementation phase. | SM | ☐ | |
| 6.4 | **Business Unit sign-off**: Scope, approach, and pre-implementation outcomes (PROD backup, UAT restored, migration list agreed, APK build) accepted for proceeding to implementation. | Business Unit | ☐ | See [Sign-off](#sign-off) section. |

---

## Implementation (go-live day)

### 7. DAR Middleware deployment (Delivery Team + DevOps)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 7.1 | Clone **dev** branch of DAR Middleware from GitHub. | Delivery Team | ☐ | `git clone ... && git checkout dev` |
| 7.2 | Run **npm install** in repository root. | Delivery Team | ☐ | |
| 7.3 | Configure **knex_config** to point to **UAT DB**: set `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE` (or edit `db/knex_config.js` for UAT). | Delivery Team / DevOps | ☐ | Use env vars or UAT-specific config; do not commit prod credentials. |
| 7.4 | Start Middleware: **npm run dev** (or `node server.js`). Ensure it binds to the **identified port** (e.g. set `PORT` env if needed). | Delivery Team | ☐ | |
| 7.5 | Verify Middleware is up: e.g. `GET /health` returns 200; `/api-docs` loads. | DevOps / QA | ☐ | |
| 7.6 | Smoke test: login and one sync endpoint (e.g. `/api/operation-config` or `/user/login` with test user). | Backend / QA | ☐ | |

### 8. APK distribution & install (Delivery / PM)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 8.1 | Share APK download link or file with test users (or on-site install). | PM / Delivery | ☐ | |
| 8.2 | **User downloads APK** and installs on device (enable “Install from unknown sources” if required). | End user / Delivery | ☐ | |
| 8.3 | Confirm app opens and can reach Middleware (e.g. login screen loads, no connection error). | QA / SM | ☐ | |

---

## Post-implementation

### 9. UAT execution (QA Lead + QA)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 9.1 | Run **UAT with test user(s)** per UAT checklist (login, sync, create report — individual and group — submit DAR form, verify data in UAT DB or reports). | QA Lead / QA | ☐ | Use [dar-user-journey-and-form-specifications.md](../project/dar-user-journey-and-form-specifications.md) for flows. |
| 9.2 | Log defects or issues; communicate to Delivery and PM. | QA | ☐ | |
| 9.3 | **QA Lead sign-off**: UAT passed for agreed scope (or document known issues and go/no-go). | QA Lead | ☐ | |
| 9.4 | SM confirms Definition of Done for deployment (artifacts, rollback plan if needed). | SM | ☐ | |
| 9.5 | **PM sign-off**: Release accepted; handover to ops or next phase. | PM | ☐ | |
| 9.6 | **Business Unit sign-off**: UAT outcome accepted; approval to proceed (e.g. pilot, wider rollout, or next phase). | Business Unit | ☐ | See [Sign-off](#sign-off) section. |

### 10. Rollback (if needed)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 10.1 | If rollback: stop Middleware; point users back to previous APK or environment; restore DB from backup if required. | DevOps / Delivery | ☐ | Document rollback steps in deployment runbook. |
| 10.2 | Post-rollback: incident log and retrospective. | PM / SM | ☐ | |

---

## Sign-off

Formal sign-off for deployment. Complete and retain with the MOP.

| Role | Sign-off (pre-implementation) | Date | Name |
|------|--------------------------------|------|------|
| **Business Unit** | Scope, approach, and pre-implementation outcomes accepted; proceed to implementation. | | |
| PM | Timeline and scope confirmed. | | |
| DevOps | Port, host, UAT DB access ready. | | |
| SM | Pre-implementation checklist complete. | | |

| Role | Sign-off (post-implementation) | Date | Name |
|------|--------------------------------|------|------|
| **Business Unit** | UAT outcome accepted; approval to proceed (pilot / rollout / next phase). | | |
| QA Lead | UAT passed for agreed scope (or go/no-go documented). | | |
| PM | Release accepted; handover confirmed. | | |
| SM | Definition of Done for deployment confirmed. | | |

---

## Quick reference

| Item | Where |
|------|--------|
| **Middleware repo** | DAR_Middleware (GitHub); branch **dev**. |
| **Middleware config** | `db/knex_config.js` — connection from env: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`. |
| **Middleware start** | `npm run dev` (or `node server.js`); `PORT` env or default 3000. |
| **Migrations (primary)** | From repo root: `npx knex migrate:latest --knexfile db/knexfile.js`. |
| **Migrations (backup)** | If Knex fails: run the **exact .sql scripts from the migration list** (section 2) in SSMS against UAT, in the documented order. |
| **Seeds** | `npx knex seed:run --knexfile db/knexfile.js`. |
| **Flutter base URL** | main_dar_app `lib/core/presentation/theme/config.dart` → `Config.baseURL`. |
| **APK build** | From main_dar_app: `flutter build apk --release`. |

---

## Document control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | (fill) | Delivery / PM | Initial MOP and checklist. |

**Contributors (team)**: PM (timeline, sign-off), SM (checklist completeness, DoD), DevOps (middleware host/port, deployment steps), Architect/Backend (DB scripts, schema, PROD vs DEV comparison), Frontend (APK, base URL), QA Lead (UAT execution, sign-off), **Business Unit** (scope and UAT outcome sign-off). Use this doc as the single checklist for the deployment; update “Done” and “Notes” as steps complete.
