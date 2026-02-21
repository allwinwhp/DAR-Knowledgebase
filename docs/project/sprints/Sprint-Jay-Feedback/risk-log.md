# Sprint-Jay-Feedback — Risk Log

**Owner**: SM / PM

---

## Risks and mitigations

| ID | Risk | Impact | Mitigation | Status |
|----|------|--------|------------|--------|
| R1 | Survey/Gouging header submission failure (batch saves, header fails) | Data not sent to server | Backend investigation (BE-2, BE-3); fix payload/validation | Open |
| R2 | ERAD accomplishments not saved | Incomplete data on server | Fix accomplishment submission (BE-7) | Open |
| R3 | Chemical Mixing operation code mismatch (tbl_MixingOprtn) | Wrong OpCode in DB | Align mapping (BE-5) | Open |
| R4 | US-J6 Weedspray: one time per report vs per accomplishment | Product decision pending | PO decision; then FE-4 implementation | Open |
| R5 | Server selection: base URL used everywhere | Inconsistent API target | Single source of truth (Config.baseURL); all services use it | Mitigated by design |

---

## Blockers

| ID | Blocker | Owner | Resolution |
|----|---------|--------|------------|
| — | None at sprint start | — | — |

---

## Escalations

| From | To | When | Outcome |
|------|-----|------|---------|
| — | — | — | — |
