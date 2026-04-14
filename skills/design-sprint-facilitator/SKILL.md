---
name: design-sprint-facilitator
description: Facilitates Design Sprint-style product discovery and delivery. Simulates cross-functional team (PO, PM, SolArch, BA, Eng Lead, QA, SM), produces BRD, user stories, backlog, architecture, QA plan, roadmap. Use when running design sprints, product discovery, or translating product ideas into build-ready artifacts. Consumes knowledge base docs; delegates to subagents for architecture and QA when available.
---

# Design Sprint Facilitator

Facilitates a Design Sprint–style session that translates a product idea into validated, build-ready artifacts. Simulates a virtual cross-functional team. Produces deliverables suitable for backlog creation, sprint planning, architecture review, and business approval.

## Knowledge Base Integration

**Before starting the sprint**, read and use these as source of truth (when they exist):

| Doc | Path | Use |
|-----|------|-----|
| BRD | docs/specs/business/ | Business objectives, personas, scope, assumptions |
| Epics | docs/backlog/epics.md | Epic/feature structure, MVP vs Post-MVP |
| User Stories | docs/backlog/user-stories.md | Story set, acceptance criteria, traceability |

Extend, refine, or create new artifacts from this base. Do not duplicate blindly—evolve based on sprint insights.

---

## Subagent Delegation

When producing role-heavy outputs, delegate to these subagents when available:

| Subagent | Use For | Location |
|----------|---------|----------|
| business | BRD, business plan, 3-year PHP projections, revenue model | .cursor/agents/ |
| architect | Architecture, system design, integrations | .cursor/agents/ |
| design-authority | Design system, color base, UI/UX standards, component traceability; design-system.md | .cursor/agents/ |
| qa-lead | Test strategy, test plan, traceability | .cursor/agents/ |
| po | Scope, prioritization, user stories, backlog readiness; PO sign-off | .cursor/agents/ |
| pm | Timeline, milestones, roadmap, risks; PM sign-off | .cursor/agents/ |
| devops | CI/CD pipeline, deployment, self-healing, observability, TDD/automation strategy | .cursor/agents/ |

Invoke with: *"Use the business agent to produce the business plan and PHP projections"* (and similar for architect, qa-lead, po, pm, devops). If agents are not present, produce the output yourself using the role perspective.

---

## Sprint Flow: DISCOVER → DESIGN → DECIDE

### 1. DISCOVER

**Goals**: Business problems, market opportunities, constraints.

| Role | Contribution |
|------|--------------|
| BA | Business problem statement, personas |
| PO | Business goals, success metrics |
| PM | Constraints (budget, timeline), risks |
| SolArch | Technical feasibility, boundary awareness |

**Outputs**:
- Business problem statement
- Target users and personas
- High-level assumptions and risks
- PH market context (Philippine Peso, local constraints) when applicable

### 2. DESIGN

**Goals**: Core journeys, feature breakdown, system boundaries.

| Role | Contribution |
|------|--------------|
| BA | User journeys |
| PO | Epics, initial user stories |
| SolArch | System boundaries, integrations |
| System Analyst | Traceability, requirements mapping |

**Outputs**:
- High-level product design decisions
- Initial user story set
- Draft system architecture overview

### 3. DECIDE

**Goals**: MVP scope, refined stories, backlog structure.

| Role | Contribution |
|------|--------------|
| PO | MVP scope, prioritization |
| PM | Timeline, milestones |
| Eng Lead | Technical stories, dependencies |
| Senior DevOps | Pipeline, deploy strategy, test automation gates |
| SM | Backlog format, sprint readiness |

**Outputs**:
- Refined MVP user stories
- Technical stories
- Prioritized backlog

---

## Required Deliverables (Produce All)

1. **Business Requirement & Business Plan**
   - Business objectives and KPIs
   - Revenue model and assumptions
   - 3-year financial projections in **Philippine Peso (PHP)**: Revenue, Costs, Gross margin (high-level)

2. **User Stories & Technical Stories**
   - By persona: Organizer, Attendee, Validator, Admin
   - Acceptance criteria per story
   - Technical stories: Architecture, Infrastructure, Security, Performance

3. **Backlog Reflection**
   - Epics, Features, Sprint-ready items
   - Dependencies and sequencing

4. **Architecture & System Design**
   - High-level system architecture
   - Key components and responsibilities
   - Integration points (payments, email, QR validation)
   - Non-functional considerations
   - **Technical implementation diagrams** (minimum one sequence or component diagram; PlantUML preferred)  
   *→ Delegate to architect agent if available. Diagram ownership is with Solution Architect.*

5. **QA & Test Plan**
   - Overall test strategy
   - Functional, Integration, Regression, UAT
   - Traceability: user stories → test coverage  
   *→ Delegate to qa-lead agent if available*

6. **Development & Delivery Roadmap**
   - MVP roadmap, Post-MVP roadmap
   - High-level timeline and milestones
   - Roles accountable per phase

---

## Output Format & Style

- Use **Markdown** format
- Label each section clearly
- Attribute outputs to roles where appropriate (e.g., *[SolArch]* ...)
- Keep artifacts **realistic and build-ready**
- Avoid theoretical explanations—focus on execution
- Treat outputs as handoff to engineering and leadership

---

## Role Ownership Summary

| Role | Primary Outputs |
|------|-----------------|
| PO | Scope, prioritization, user stories |
| PM | Timeline, risks, milestones, roadmap |
| SolArch | Architecture, integrations, NFRs, technical implementation diagrams |
| BA | BRD, business plan, financials, personas |
| System Analyst | Traceability, requirements mapping |
| Eng Lead | Technical stories, dependencies |
| QA Lead | Test strategy, test plan |
| SM | Backlog structure, sprint readiness |
| SD | Implementation input (tech constraints) |
| Senior DevOps | CI/CD, deployment, observability, self-healing, test automation pipeline |

---

## Product Context (Default vs DAR)

**Default** (when not working on DAR): Multi-sided event marketplace and ticketing platform (public/private events, ticket purchase, QR validation, Organizer Console, tiered organizers).

**DAR** (when working on DAR): Use this context instead. Full doc: [docs/project/dar-system-context.md](../../docs/project/dar-system-context.md).

- **Product**: DAR — input interface for **farm supervisors** to track **daily work** (accomplishment reports).
- **Operations**: Fruit care, bagging, gouging, harvest, fertilization, chemical mixing, survey, utility, MPS, erad, sigatoka, popcount, etc.
- **Repos**: **DAR_Middleware** (Node/Express API, MSSQL, Knex) + **main_dar_app** (frontend input UI). Branch: **MDAG-339** (not yet merged to DEV).
- **Personas**: Farm supervisor (primary), operations/back office, system/integration.
- **Discovery**: [docs/project/dar-discovery-summary.md](../../docs/project/dar-discovery-summary.md) (DISCOVER → DESIGN → DECIDE for DAR Middleware and main_dar_app).

Override with user-provided context when given; when the user says they are working on DAR or taking over the DAR project, use the DAR context above.

---

## Reference

- Full sprint structure and templates: [reference.md](reference.md)
