# Sprint-Jay-Feedback — Deployment Plan

**Owner**: DevOps

---

## Environment

| Env | Backend | Frontend | Notes |
|-----|---------|----------|--------|
| Local dev | Node (DAR_Middleware); ngrok for DB | Flutter (main_dar_app) | DB: knex_config.js — current live ngrok (0.tcp.ap.ngrok.io:10653) |
| UAT | Per team setup | Built app / emulator | Use same ngrok base URL for backend during manual validation |

---

## Pre-deployment

- [ ] All sprint tasks complete per tasks-by-agent.md
- [ ] Test matrix executed; test cases passed
- [ ] QA sign-off on stories
- [ ] risk-log.md reviewed; no open blockers

---

## Deployment steps

1. **Backend (DAR_Middleware)**  
   - Ensure branch `sprint/Sprint-Jay-Feedback-*` or `MDAG-339` is up to date.  
   - No DB migrations required for filter/API changes (existing tables).  
   - Deploy/run Node server; ensure env (DB_HOST, DB_PORT, etc.) points to live ngrok when validating.

2. **Frontend (main_dar_app)**  
   - Build Flutter app from same branch.  
   - Configure app to use target backend base URL (server selection or config).  
   - Distribute build for UAT (e.g. emulator/device).

3. **UAT**  
   - Stakeholder runs uat-checklist.md against deployed/env configuration.  
   - Backend configuration must use current live ngrok setup for consistency.

---

## Rollback

- Revert to previous branch/commit; redeploy backend and/or redistribute app build.
- No schema rollback needed (no new migrations).

---

## Post-deployment

- Update sprint-review.md and audit.md.
- Record release in docs/project/releases.md when release is cut (per release-playbook).
