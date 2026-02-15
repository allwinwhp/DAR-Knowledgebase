# Product Master Foundations — Reference

## Repository Tree Structure

```
docs/
├── specs/
│   ├── business/
│   │   ├── objectives.md
│   │   ├── stakeholders.md
│   │   └── processes-and-constraints.md
│   ├── product/
│   │   ├── vision-and-success.md
│   │   ├── user-journeys.md
│   │   └── non-functional-requirements.md
│   └── functional/
│       ├── architecture.md
│       ├── data-models.md
│       ├── apis.md
│       └── security-performance.md
├── backlog/
│   ├── epics.md
│   ├── stories/
│   │   ├── US-001-example.md
│   │   └── TS-001-example.md
│   ├── refinement-log.md
│   └── prioritization.md
├── qa/
│   ├── test-strategy.md
│   ├── test-plans/
│   │   └── release-1.0.md
│   ├── test-cases/
│   └── regression-strategy.md
├── project/
│   ├── project-plan.md
│   ├── sprints/
│   ├── milestones-and-dependencies.md
│   ├── risks-and-issues.md
│   └── release-plans/
├── roadmap/
│   ├── mvp-roadmap.md
│   ├── post-mvp.md
│   └── strategic-roadmap.md
└── governance/
    └── roles-and-responsibilities.md
```

---

## Phase-to-Artifact Mapping

| Phase | Artifacts | Primary Roles |
|-------|-----------|---------------|
| Design | Business specs, Product specs, User journeys | BA, PO, SolArch |
| Development | Functional specs, Epics, Stories, Test plans | PO, SolArch, Eng Lead, QA |
| Deployment | Release plans, Runbooks, Go-live checklist | PM, Eng Lead, QA |

---

## Role and Responsibility Matrix

| Role | Scope | Priority | Technical | Approvals |
|------|-------|----------|-----------|-----------|
| PM | Aligns | Aligns timeline | — | Project/release |
| PO | Owns | Owns | Input | Product scope |
| SolArch | Input | — | Owns | Architecture |
| SM | Process | — | — | — |
| Engineering Lead | Input | — | Owns approach | Technical approach |
| QA Lead | Input | Input | Test design | Quality sign-off |
| BA | Owns analysis | Input | — | Requirements |

---

## Markdown Templates

### Business Specification

```markdown
# [Product/Initiative] — Business Specification

## Business Objectives
- [Objective 1]
- [Objective 2]

## KPIs
| KPI | Target | Measurement |
|-----|--------|-------------|
|     |        |             |

## Stakeholders
| Role/Group | Interest | Influence |
|------------|----------|-----------|

## User Segments
- [Segment 1]: [Description]
- [Segment 2]: [Description]

## Business Processes
[High-level processes affected]

## Constraints
- [Constraint 1]
```

### Product Specification

```markdown
# [Product] — Product Specification

## Vision
[One-paragraph product vision]

## Success Criteria
- [Criterion 1]
- [Criterion 2]

## User Journeys
### Journey: [Name]
1. [Step]
2. [Step]
3. [Step]

## Feature Definitions
### [Feature name]
- **Description**: 
- **Source**: [Spec/Story reference]
- **Acceptance**: 

## Non-Functional Requirements
| Category | Requirement |
|----------|-------------|
| Performance | |
| Security | |
| Scalability | |
```

### Functional / Technical Specification

```markdown
# [Component] — Functional Specification

## Overview
[Brief description]

## Architecture
[High-level architecture]

## Integrations
| System | Purpose | Protocol |
|--------|---------|----------|

## Data Models
[Key entities and relationships]

## APIs
| Endpoint | Method | Purpose |
|----------|--------|---------|

## Security
[Auth, data protection, compliance]

## Performance & Scalability
[Targets, scaling approach]
```

### User Story

```markdown
# US-[ID]: [Title]

## User Story
As a [role], I want [capability] so that [benefit].

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Source
- Spec: [Link/reference]
- Epic: [Link/reference]

## Notes
[Optional]
```

### Technical Story

```markdown
# TS-[ID]: [Title]

## Description
[Technical work: architecture, infra, refactoring]

## Acceptance Criteria
- [ ] [Criterion 1]

## Dependencies
- [Link]

## Notes
```

### Backlog Refinement Log

```markdown
# Refinement Log — [Date]

## Stories Refined
| ID | Title | Status |
|----|-------|--------|
| US-001 | ... | Ready |
| TS-001 | ... | Blocked |

## Decisions
- [Decision 1]

## Deferred
- [Item] — Reason
```

### Test Plan

```markdown
# Test Plan — [Release/V Sprint]

## Scope
[What is in scope for testing]

## Out of Scope
[Explicit exclusions]

## Test Strategy
- Unit: 
- Integration: 
- E2E: 
- UAT: 

## Test Cases
| ID | Scenario | Type | Status |
|----|----------|------|--------|
| TC-001 | | | |
```

### Project Plan (High-Level)

```markdown
# Project Plan — [Project Name]

## Milestones
| Milestone | Target | Owner | Status |
|-----------|--------|-------|--------|
| M1 | Date | | |
| M2 | Date | | |

## Dependencies
| Dependency | Type | Owner |

## Risks
| Risk | Impact | Mitigation |

## Issues
| Issue | Owner | Status |
```

### Roadmap

```markdown
# [Product] Roadmap

## MVP
| Release | Target | Goals |
|---------|--------|-------|
| R1 | Qx | [Goals] |

## Post-MVP
| Release | Target | Goals |

## Strategic (Long-term)
[High-level themes and direction]

## Dependencies
[Cross-release dependencies]
```

---

## Constraints and Style

- Assume long-lived, production product.
- Pragmatic over academic.
- Trade-offs: call them out when relevant.
- Optional approaches: present as alternatives when applicable.
