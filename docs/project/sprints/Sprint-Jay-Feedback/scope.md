# Sprint-Jay-Feedback — Scope

**Source**: [jay-feedback-intake.md](../../jay-feedback-intake.md)

---

## Epics

| Epic | Description |
|------|-------------|
| **Epic-Jay** | Jay Feedback II — Filters, submission fixes, missing fields, Gang/Group timekeeping and incentives |
| **Epic-Server** | Server selection before login (base URL for session) |

---

## User stories (backlog items)

| ID | Title | Acceptance criteria | Deps |
|----|--------|---------------------|------|
| US-J1 | General Services costcenter/incentive filter | Costcenter: tblActivity Active=1. Incentive: Doctype != '03','01'. Available to all applicable operations. | — |
| US-J2 | PD Survey send to server | Material max 5; header submission succeeds; full flow sends. | — |
| US-J3 | Gouging send to server | Incentive Opcode '25'; header submission succeeds. | — |
| US-J4 | Sucker Pruning Cycle/Sqno and incentive | Cycle no., Seq no. in app; save to tblDarHdr.Cycle, tblDarHdr.Sqno; incentive Icode 'RP13'. | — |
| US-J5 | Chemical Mixing operation code | Saved operation code matches mobile; fix tbl_MixingOprtn/costcenter mapping. | — |
| US-J6 | Weedspray timekeeping | One accomplishment per report or time per accomplishment (product decision). | PO |
| US-J7 | CPMS decimal | Material quantity accepts decimal. | — |
| US-J8 | Fertilizer incentive and remove Operations | Add incentive (Opcode '25'); remove Operations field. | — |
| US-J9 | General Services Group timekeeping | Costcenter and incentive in timekeeping per employee; same filters. | — |
| US-J10 | PD Eradication incentive and accomplishments | Incentive filter; ERAD accomplishments saved to server. | — |
| US-J11 | Planting/Replanting header costcenter | Costcenter selection in header (two options). | — |
| US-J12 | Popcount HAS and incentive | HAS = layout actual area; remove incentive field. | — |

---

## Technical stories

| ID | Title | Acceptance criteria |
|----|--------|---------------------|
| TS-SERVER | Server selection before login | Pre-login screen: dropdown of servers → selected URL = base URL for session. |

---

## Dependencies

- Backend filter/sync endpoints must expose correct queries (tblActivity, tblincentives) for US-J1, J2, J3, J4, J8, J9, J10.
- Frontend forms: Cycle/Sqno (J4), costcenter in timekeeping (J9), header costcenter (J11), HAS from layout (J12), decimal CPMS (J7).
- ERAD and Survey/Gouging header submission: backend investigation (J2, J3, J10).
