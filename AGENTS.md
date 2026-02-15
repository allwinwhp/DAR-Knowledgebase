# Agent Index

**Purpose**: Index of all agents for squad-based development, design sprints, sprint planning, and oversight.

**Playbook**: [docs/development/development-playbook.md](../docs/development/development-playbook.md)  
**Sprint planning**: [sprint-planning-playbook](../docs/development/sprint-planning-playbook.md) | [sprints](../docs/project/sprints/) — use **sprint-planning-facilitator** skill with subagent delegation.

---

## Squads

| Agent | Squad | Use For |
|-------|-------|---------|
| [backend-squad](agents/backend-squad.md) | Backend | API, GraphQL, MongoDB, auth, payment/email integrations |
| [frontend-squad](agents/frontend-squad.md) | Frontend | Next.js, marketplace, Organizer Console, Validator UI |

---

## Developers (Senior / Mid / Junior)

| Agent | Level | Authority | Use For |
|-------|-------|-----------|---------|
| [dev-senior](agents/dev-senior.md) | Senior | Approves feat→us; reviews Mid/Junior | Code review, approval, architecture |
| [dev-mid](agents/dev-mid.md) | Mid | Reviews Junior; no approval | Implementation, review junior work |
| [dev-junior](agents/dev-junior.md) | Junior | Implementation only | Assigned tasks under guidance |

---

## QA (Senior / Mid / Junior)

| Agent | Level | Authority | Use For |
|-------|-------|-----------|---------|
| [qa-senior](agents/qa-senior.md) | Senior | QA sign-off us→epic; test strategy | Test strategy, critical path, sign-off |
| [qa-mid](agents/qa-mid.md) | Mid | Test cases, traceability | Story test cases, regression, traceability |
| [qa-junior](agents/qa-junior.md) | Junior | Manual execution | Manual testing, test data |

---

## Oversight (PM / SM / Orchestrator)

| Agent | Role | Oversight |
|-------|------|-----------|
| [pm](agents/pm.md) | PM | Epic completion, release readiness, timeline, risk |
| [development-sm](agents/development-sm.md) | SM | Sprint backlog completeness, ceremonies, blockers |
| [delivery-orchestrator](agents/delivery-orchestrator.md) | Orchestrator | Sprint **execution** only: agent order, governance, Sprint Package, UAT/Review/Audit/Release. Use with **sprint-execution** skill. |

---

## Roles (Architect, Business, Design, DevOps, PO, QA Lead)

| Agent | Role | Use For |
|-------|------|---------|
| [architect](agents/architect.md) | SolArch | Architecture, technical stories |
| [business](agents/business.md) | BA | BRD, business plan, traceability |
| [design-authority](agents/design-authority.md) | Design | Design system, color base, component standards, UX for all personas |
| [qa-lead](agents/qa-lead.md) | QA Lead | Test plan, traceability matrix |
| [po](agents/po.md) | PO | Scope, prioritization, backlog |
| [pm](agents/pm.md) | PM | Timeline, roadmap, risks |
| [devops](agents/devops.md) | DevOps | CI/CD, deployment, observability |

---

## Discovery & Ideation (feature ideas, marketplace vision)

| Agent | Role | Use For |
|-------|------|---------|
| [digital-marketing-specialist](agents/digital-marketing-specialist.md) | Digital Marketing | Feature ideas from marketing/growth lens; acquisition, retention, campaigns; ideation with the team |
| [ideation-specialist](agents/ideation-specialist.md) | Ideation | New feature lists, brainstorming, collaborative ideation; consolidate team ideas (Ticketmaster meets Apple) |

**Vision**: Events marketplace — *Ticketmaster meets Apple* (scale + discovery + premium UX). Use these agents with PO, business, design-authority, and the rest of the team to generate and curate a list of new features for prioritization.

---

## DAR project (farm operations)

When working on **DAR** (farm supervisor daily accomplishment tracking): use **DAR context**, not the events marketplace. Repos: **DAR_Middleware** + **main_dar_app**; branch **MDAG-339**.

- **Project context & alignment**: [docs/project/dar-system-context.md](docs/project/dar-system-context.md)
- **Discovery summary** (DISCOVER → DESIGN → DECIDE): [docs/project/dar-discovery-summary.md](docs/project/dar-discovery-summary.md)
- **Delivery orchestrator**: use the "Project context: DAR" section in [agents/delivery-orchestrator.md](agents/delivery-orchestrator.md)
- **Design sprint**: [skills/design-sprint-facilitator](skills/design-sprint-facilitator) — Product Context includes DAR; point to the docs above when facilitating for DAR.

---

## Sprint planning (subagenting)

Use the **sprint-planning-facilitator** skill to run sprint planning. PM ensures SM has all agents contribute; SM delegates to po, architect, business, qa-lead, devops, dev-senior, design-authority, backend-squad, frontend-squad, dev-mid, dev-junior, qa-senior, qa-mid, qa-junior; output goes to docs/project/sprints/Sprint-NN/.

- *"Run sprint planning for Sprint-02 using sprint-planning-facilitator; delegate to all agents"*
- *"Run sprint execution for docs/project/sprints/Sprint-01"* (uses **sprint-execution** skill + delivery-orchestrator)

---

## Invocation

Use agents by name when delegating work. Example:

- *"Use backend-squad and dev-senior to review this GraphQL resolver PR"*
- *"Use qa-senior to verify traceability for US-015"*
- *"Use development-sm to check sprint backlog completeness"*
- *"Use ideation-specialist and digital-marketing-specialist to give me a list of new features for the events marketplace; Ticketmaster meets Apple"*
- *"Run an ideation session: have ideation-specialist, digital-marketing-specialist, po, business, and design-authority throw in feature ideas and consolidate into one list"*
