# DAR Deployment MOP — Method of Procedure & Checklist

**Purpose**: Checklist for deploying DAR (UAT) — Middleware, UAT database, and Flutter APK. Use with the Delivery Orchestrator and team roles. **Sign-off**: PM confirms go-live; DevOps approves deployment steps; QA Lead signs off UAT.

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

### 2. Database alignment (Delivery Team — Backend / Architect)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 2.1 | Obtain list of DB scripts/migrations required for DAR Application and Middleware (see **MIGRATION_GUIDE.md** in DAR_Middleware repo). | Backend / Architect | ☐ | Typically: `npx knex migrate:latest --knexfile db/knexfile.js` and optionally seeds. |
| 2.2 | Run migrations on **DAR UAT DB** (e.g. from DAR_Middleware repo: `npx knex migrate:latest --knexfile db/knexfile.js` with knex_config pointing to UAT). | Delivery Team | ☐ | Use UAT connection only. |
| 2.3 | Run seed data if required (e.g. `npx knex seed:run --knexfile db/knexfile.js`). | Delivery Team | ☐ | E.g. operation config, mixing ops. |
| 2.4 | Verify schema: key tables present (e.g. tblDARbatch, tblDARhdr, tblDARdtls, tblDARaccomp, tblDARmaterials, tbl_OperationConfig, tblsupervisor_operators, tblusers, tblaccounts, tblemployee). | Backend / Architect | ☐ | |

### 3. Middleware host & port (DevOps + MDAG IT)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 3.1 | Identify **server** (hostname or IP) where DAR Middleware will run for UAT. | DevOps / MDAG IT | ☐ | E.g. same DB server or app server. |
| 3.2 | Identify **port** on that server for the Middleware (e.g. 3000 or agreed port). | DevOps / MDAG IT | ☐ | Ensure firewall/security allows inbound on this port for UAT users/devices. |
| 3.3 | Document **UAT Middleware base URL** (e.g. `http://<host>:<port>` or `https://...`). | DevOps | ☐ | This URL will be used in the Flutter APK `Config.baseURL`. |

### 4. Flutter APK build (Delivery Team — Frontend)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 4.1 | Set **base URL** in main_dar_app to the identified UAT Middleware server and port (e.g. in `lib/core/presentation/theme/config.dart`: `Config.baseURL = "http://<host>:<port>"`). | Frontend | ☐ | No trailing slash. Use build flavor or env if available. |
| 4.2 | Build **release APK** (e.g. `flutter build apk --release` from main_dar_app). | Frontend | ☐ | |
| 4.3 | Store APK in agreed location (e.g. shared drive, link) for distribution to test users. | Delivery / PM | ☐ | |
| 4.4 | Document APK version and build date; confirm base URL in build. | Frontend / SM | ☐ | |

### 5. Pre-implementation sign-off

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 5.1 | PM confirms timeline and scope for deployment day. | PM | ☐ | |
| 5.2 | DevOps confirms port, host, and UAT DB access are ready. | DevOps | ☐ | |
| 5.3 | SM confirms checklist 1.1–4.4 complete before implementation phase. | SM | ☐ | |

---

## Implementation (go-live day)

### 6. DAR Middleware deployment (Delivery Team + DevOps)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 6.1 | Clone **dev** branch of DAR Middleware from GitHub (or agreed branch, e.g. `dev` / `MDAG-339`). | Delivery Team | ☐ | `git clone ... && git checkout dev` |
| 6.2 | Run **npm install** in repository root. | Delivery Team | ☐ | |
| 6.3 | Configure **knex_config** to point to **UAT DB**: set `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE` (or edit `db/knex_config.js` for UAT). | Delivery Team / DevOps | ☐ | Use env vars or UAT-specific config; do not commit prod credentials. |
| 6.4 | Start Middleware: **npm run dev** (or `node server.js`). Ensure it binds to the **identified port** (e.g. set `PORT` env if needed). | Delivery Team | ☐ | |
| 6.5 | Verify Middleware is up: e.g. `GET /health` returns 200; `/api-docs` loads. | DevOps / QA | ☐ | |
| 6.6 | Smoke test: login and one sync endpoint (e.g. `/api/operation-config` or `/user/login` with test user). | Backend / QA | ☐ | |

### 7. APK distribution & install (Delivery / PM)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 7.1 | Share APK download link or file with test users (or on-site install). | PM / Delivery | ☐ | |
| 7.2 | **User downloads APK** and installs on device (enable “Install from unknown sources” if required). | End user / Delivery | ☐ | |
| 7.3 | Confirm app opens and can reach Middleware (e.g. login screen loads, no connection error). | QA / SM | ☐ | |

---

## Post-implementation

### 8. UAT execution (QA Lead + QA)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 8.1 | Run **UAT with test user(s)** per UAT checklist (login, sync, create report — individual and group — submit DAR form, verify data in UAT DB or reports). | QA Lead / QA | ☐ | Use [dar-user-journey-and-form-specifications.md](../project/dar-user-journey-and-form-specifications.md) for flows. |
| 8.2 | Log defects or issues; communicate to Delivery and PM. | QA | ☐ | |
| 8.3 | **QA Lead sign-off**: UAT passed for agreed scope (or document known issues and go/no-go). | QA Lead | ☐ | |
| 8.4 | SM confirms Definition of Done for deployment (artifacts, rollback plan if needed). | SM | ☐ | |
| 8.5 | **PM sign-off**: Release accepted; handover to ops or next phase. | PM | ☐ | |

### 9. Rollback (if needed)

| # | Task | Owner | Done | Notes |
|---|------|--------|------|--------|
| 9.1 | If rollback: stop Middleware; point users back to previous APK or environment; restore DB from backup if required. | DevOps / Delivery | ☐ | Document rollback steps in deployment runbook. |
| 9.2 | Post-rollback: incident log and retrospective. | PM / SM | ☐ | |

---

## Quick reference

| Item | Where |
|------|--------|
| **Middleware repo** | DAR_Middleware (GitHub); branch **dev** (or MDAG-339 per context). |
| **Middleware config** | `db/knex_config.js` — connection from env: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`. |
| **Middleware start** | `npm run dev` (or `node server.js`); `PORT` env or default 3000. |
| **Migrations** | From repo root: `npx knex migrate:latest --knexfile db/knexfile.js`. |
| **Seeds** | `npx knex seed:run --knexfile db/knexfile.js`. |
| **Flutter base URL** | main_dar_app `lib/core/presentation/theme/config.dart` → `Config.baseURL`. |
| **APK build** | From main_dar_app: `flutter build apk --release`. |

---

## Document control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | (fill) | Delivery / PM | Initial MOP and checklist. |

**Contributors (team)**: PM (timeline, sign-off), SM (checklist completeness, DoD), DevOps (middleware host/port, deployment steps), Architect/Backend (DB scripts, schema), Frontend (APK, base URL), QA Lead (UAT execution, sign-off). Use this doc as the single checklist for the deployment; update “Done” and “Notes” as steps complete.
