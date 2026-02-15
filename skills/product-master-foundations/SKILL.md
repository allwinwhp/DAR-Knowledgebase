---
name: product-master-foundations
description: Establishes specification-driven delivery for new products using Markdown as single source of truth. Use when defining product specs, repo structure, backlog, QA docs, roadmaps, or delivery governance. Covers business/product/functional specs, user stories, backlog refinement, test strategy, project plans, and role responsibilities.
---

# Product Master Foundations

Specification-driven delivery system for large-scale, multi-role software platforms. Markdown (.md) is the single source of truth.

## Role

Act as a senior product leader, program manager, and solution architect. Provide pragmatic, scalable guidance—no fluff. Call out trade-offs and optional approaches.

---

## 1. Repository Structure

When proposing or creating repo structure, use the canonical layout from [reference.md](reference.md). Key folders:

| Folder | Purpose |
|--------|---------|
| `docs/specs/business/` | Business objectives, KPIs, stakeholders |
| `docs/specs/product/` | Vision, user journeys, NFRs |
| `docs/specs/functional/` | Architecture, data models, APIs |
| `docs/backlog/` | Epics, stories, refinement |
| `docs/qa/` | Test strategy, plans, cases |
| `docs/project/` | Plans, sprints, risks, releases |
| `docs/roadmap/` | MVP, post-MVP, strategic |

Principles: intuitive for PMs, POs, engineers, QA, leadership; supports long-term iteration; preserves traceability as docs evolve.

---

## 2. Lifecycle Phases

| Phase | Goals | Core Activities | Key Artifacts |
|-------|-------|-----------------|---------------|
| **Design** | Align stakeholders, define scope | Discovery, spec writing | Business/Product specs |
| **Development** | Build incrementally | Sprints, refinement | Stories, functional specs |
| **Deployment** | Ship safely | Release planning, ops | Release plans, runbooks |

For each phase: define goals, outcomes, activities, artifacts (mapped to repo folders), and accountable roles. See reference for phase-to-artifact mapping.

---

## 3. Roles & Governance

| Role | Decision Ownership |
|------|--------------------|
| PM | Timeline, resources, risk, status |
| PO | Scope, priority, acceptance |
| SolArch | Architecture, tech stack, integrations |
| SM | Process, ceremonies, blockers |
| Engineering Lead | Technical approach, code quality |
| SD | Implementation |
| QA Lead | Test strategy, coverage, sign-off |
| BA | Requirements, analysis |
| System Analyst | Technical analysis, traceability |

Scope/priority: PO owns; PM aligns with timeline. Technical decisions: SolArch + Engineering Lead. Approvals flow from spec type (business → product → functional).

---

## 4. Specification Layers

**A. Business specs**: Objectives, KPIs, stakeholders, segments, processes, constraints.  
**B. Product specs**: Vision, success criteria, user journeys, features, NFRs.  
**C. Functional/Technical specs**: Architecture, integrations, data models, APIs, security, performance, scalability.

Traceability: Business → Product → Functional → Stories.

---

## 5. Backlog & Story Management

- **Location**: `docs/backlog/` (epics, stories, refinement logs).
- **Hierarchy**: Feature → Epic → Story → Task.
- **Story types**: User stories (with acceptance criteria), technical stories (architecture, infra, refactoring).
- **Generation**: Derive from product and functional specs; document source in each story.
- **Refinement**: Maintain refinement logs and prioritization decisions in backlog folder.

---

## 6. QA & Quality

- **Structure**: `docs/qa/` (strategy, test plans, cases, regression, defect flow).
- **Ownership**: QA Lead owns strategy and plans; test cases tied to stories.
- **Traceability**: Test cases → user/technical stories → specs.

---

## 7. Project Plans & Delivery Tracking

- **Location**: `docs/project/`.
- **Artifacts**: Project plan, sprint plans, milestones, dependencies, risk/issue logs, release plans.
- **Ownership**: PM maintains; leadership visibility via milestone and status docs.

---

## 8. Product Roadmap

- **Structure**: MVP roadmap → Post-MVP → Long-term strategic.
- **Requirements**: Versioned, time-phased; aligned to business/product specs; shows dependencies and release goals.

---

## Output Conventions

When producing deliverables:

1. Use clear headings and bullet points.
2. Avoid academic fluff and buzzwords.
3. State constraints, assumptions, and trade-offs.
4. Map artifacts to repo folders.
5. Use templates from [reference.md](reference.md) for consistency.

---

## Quick Reference

- **Repo tree and templates**: [reference.md](reference.md)
- **Phase-to-artifact mapping**: [reference.md](reference.md)
- **Role matrix**: [reference.md](reference.md)
