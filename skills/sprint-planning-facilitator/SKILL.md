---
name: sprint-planning-facilitator
description: Facilitates sprint planning with subagent delegation. PM ensures SM has all agents contribute; SM delegates to po, architect, business, qa-lead, devops, dev-senior, design-authority, backend-squad, frontend-squad, dev-mid, dev-junior, qa-senior, qa-mid, qa-junior; consolidates into per-sprint folder. Use when building or updating sprint plans. Consumes docs/backlog, docs/development, docs/roadmap, docs/qa, docs/specs.
---

# Sprint Planning Facilitator

Facilitates sprint planning so that **sprint plans are laid down per sprint, per folder**. The **PM** ensures the **SM** has **all agents** contribute to building the plans. Use **subagenting** for faster, collaborative execution: SM delegates tasks to each agent; agents return their section; SM consolidates into the sprint folder.

## Playbook and structure

- **Playbook**: [docs/development/sprint-planning-playbook.md](../../../docs/development/sprint-planning-playbook.md)
- **Sprint folders**: docs/project/sprints/ — one folder per sprint (Sprint-01, Sprint-02, …)
- **Template**: docs/project/sprints/_template/ — copy for new sprints
- **Plan contents**: sprint-plan.md, scope.md, tasks-by-agent.md, deliverables.md, references.md

Plans must consider: **docs/backlog**, **docs/development**, **docs/roadmap**, **docs/qa**, **docs/specs** (see playbook §1).

---

## Roles: PM and SM

| Role | Agent | Responsibility |
|------|--------|----------------|
| **PM** | pm | Ensures SM has all agents contribute; approves sprint scope and timeline; signs off sprint plan. |
| **SM** | development-sm | Facilitates planning; delegates tasks to all agents (subagenting); consolidates inputs into the sprint folder. |

---

## Subagent delegation (use all agents)

For **collaborative execution**, the SM delegates to these agents. Invoke in parallel where possible for speed.

### Scope and prioritization

| Agent | Delegate for |
|-------|---------------|
| **po** | Prioritized scope for sprint (from backlog); acceptance criteria alignment; backlog items for sprint. |
| **architect** | Technical scope, dependencies, sequence diagrams for sprint stories; NFRs and integration notes. |
| **business** | BRD/epic traceability for sprint scope; business acceptance criteria. |

### QA and DevOps

| Agent | Delegate for |
|-------|---------------|
| **qa-lead** | Test scope for sprint (from test-plan); test cases / TC-ids for sprint stories; QA deliverables. |
| **devops** | CI/CD and deploy readiness for sprint; branch/tag strategy; pipeline tasks. |

### Development

| Agent | Delegate for |
|-------|---------------|
| **dev-senior** | Implementation scope; capacity; technical risks; code review and merge expectations. |
| **design-authority** | Design scope; design system updates; component specs and UX tasks for sprint; design deliverables. |
| **backend-squad** | Backend tasks (API, GraphQL, MongoDB, auth) for sprint stories; backend deliverables. |
| **frontend-squad** | Frontend tasks (Next.js, pages, components) for sprint stories; frontend deliverables. |
| **dev-mid** | Mid-level implementation tasks; review assignments. |
| **dev-junior** | Junior tasks; pairing and learning goals. |

### QA execution

| Agent | Delegate for |
|-------|---------------|
| **qa-senior** | Critical path and test strategy tasks for sprint. |
| **qa-mid** | Test cases and traceability tasks for sprint stories. |
| **qa-junior** | Manual test execution and test data tasks. |

### Oversight

| Agent | Delegate for |
|-------|---------------|
| **development-sm** | Facilitate; pull scope from docs/backlog; consolidate all agent inputs into sprint folder. |
| **pm** | Ensure every agent has tasks; approve scope and timeline; sign off plan. |

---

## Execution flow

1. **User** specifies sprint (e.g. Sprint-02) or asks to create a new sprint plan.
2. **SM (development-sm)** leads:
   - Read docs/backlog/sprint-backlog.md, user-stories.md, technical-stories.md for scope.
   - Read docs/development/sprint-planning-playbook.md.
   - Create or open sprint folder (e.g. docs/project/sprints/Sprint-02/); use _template if new.
3. **SM** delegates in parallel (invoke agents by name):
   - *"Use po to produce prioritized scope and backlog items for this sprint"*
   - *"Use architect to produce technical scope and dependencies for this sprint"*
   - *"Use business to produce BRD/epic traceability for this sprint scope"*
   - *"Use qa-lead to produce test scope and test cases for this sprint"*
   - *"Use devops to produce CI/CD and release readiness tasks for this sprint"*
  - *"Use dev-senior to produce implementation scope and capacity for this sprint"*
  - *"Use design-authority to produce design scope and design tasks for this sprint"*
  - *"Use backend-squad to produce backend tasks for this sprint"*
  - *"Use frontend-squad to produce frontend tasks for this sprint"*
   - *"Use qa-senior, qa-mid, qa-junior to produce QA tasks for this sprint"*
   - *"Use dev-mid, dev-junior to produce dev tasks for this sprint"*
4. **SM** consolidates all inputs into the sprint folder:
   - sprint-plan.md (goal, dates, success criteria)
   - scope.md (epics, US-xxx, TS-xxx)
   - tasks-by-agent.md (each agent’s tasks)
   - deliverables.md (artifacts to produce)
   - references.md (links to backlog, development, roadmap, qa, specs)
5. **PM (pm)** reviews; ensures all agents have tasks; signs off.

---

## Knowledge base (sources)

Before and during planning, read and use:

| Area | Path | Use |
|------|------|-----|
| Backlog | docs/backlog/ | epics.md, user-stories.md, technical-stories.md, sprint-backlog.md, sequence-diagrams.md |
| Development | docs/development/ | development-playbook.md, release-playbook.md, sprint-planning-playbook.md, e2e-testability.md |
| Roadmap | docs/roadmap/ | development-roadmap.md, cicd-devops-roadmap.md |
| QA | docs/qa/ | test-plan.md |
| Specs | docs/specs/ | business/, functional/ (BRD, architecture) |

---

## Output format

- All plan artifacts in **Markdown** in the sprint folder.
- Attribute sections by agent where helpful (e.g. *[backend-squad]*).
- Keep tasks concrete and assignable; link to US-xxx / TS-xxx / E-x.

---

## Invocation examples

- *"Run sprint planning for Sprint-02 using the sprint-planning-facilitator; delegate to all agents and produce the plan in docs/project/sprints/Sprint-02/"*
- *"Use the sprint-planning-facilitator to build the Sprint-01 plan with subagenting"*
- *"SM: facilitate sprint planning for Sprint-03; PM ensure all agents contribute; output to docs/project/sprints/Sprint-03/"*
