# Sprint Execution — Reference

**Skill**: [SKILL.md](SKILL.md)  
**Orchestrator**: [delivery-orchestrator](../../agents/delivery-orchestrator.md)  
**Playbooks**: [sprint-planning-playbook](../../../docs/development/sprint-planning-playbook.md) | [release-playbook](../../../docs/development/release-playbook.md) | [responsibility-boundaries](../../../docs/development/responsibility-boundaries.md)

---

## Invocation

- *"Run sprint execution for docs/project/sprints/Sprint-01"*
- *"Execute the sprint in docs/project/sprints/Sprint-02"*

Input: path to sprint folder. Output: Sprint Package (all mandatory artifacts), UAT checklist, Sprint Review, Audit, release path.

---

## Execution order (summary)

1. PM → goal, scope, timeline  
2. PO → scope vs roadmap  
3. Business → refined stories, traceability  
4. Architect → validate vs approved design  
5. Design Authority → design scope (if UI)  
6. **QA Lead → test strategy and cases first (TDD)**  
7. Dev squads → unit test matrix from QA, then implementation  
8. DevOps → CI/CD impact, deployment plan  
9. SM → Definition of Done, artifact check  
10. Orchestrator → compile Sprint Package; UAT, Review, Audit, Release

---

## Mandatory Sprint Package (sprint folder)

- sprint-plan.md, scope.md, tasks-by-agent.md, deliverables.md, references.md (from planning)
- test-matrix.md, cicd-impact.md, deployment-plan.md, risk-log.md, carryover.md
- uat-checklist.md (stakeholder manual testing), sprint-review.md, audit.md
- Release: tag + docs/project/releases.md (when release is cut)

---

## Governance (no exceptions)

- No dev without QA test strategy first  
- No architecture redesign without Architect + Design Authority  
- No DB change without migration plan  
- No deployment without DevOps approval  
- No story complete without QA sign-off  
