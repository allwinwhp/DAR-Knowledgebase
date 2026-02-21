# Sprint-Jay-Feedback — Baseline & Expected Results for Dropdown Testing

**Purpose**: Compare app dropdown contents and API responses against this baseline.  
**Source**: Jay feedback SQL ([jay-feedback-intake.md](../../jay-feedback-intake.md)); baseline captured from **localhost:3000** (see [baseline-api-responses.json](baseline-api-responses.json)).

---

## 1. How to use

1. Run backend on localhost:3000 (or same DB as baseline).
2. Call the API (or sync in app) and compare response to **Expected** below.
3. In the app, open each form’s dropdown and confirm options match the **Expected** set (count and key codes).

---

## 2. Expected results table (for comparison)

| # | Form / Area | Dropdown | API request | SQL to check (Jay) | Expected (compare to) |
|---|-------------|----------|-------------|--------------------|------------------------|
| 1 | **Costcenter** (all applicable) | Costcenter | `GET /api/activity` | `SELECT Cstctr, Cstctrdesc FROM tblActivity WHERE Active = 1` | Same rows as baseline; only Active=1. Baseline count: **317** rows. |
| 2 | **General Services** (no op) | Incentive | `GET /api/incentive` | `SELECT * FROM tblincentives WHERE Doctype != '03' AND Doctype != '01'` | No Doctype '03' or '01'. Baseline count: **5** rows. |
| 3 | **Gouging** | Incentive | `GET /api/incentive?opcode=25` | Doctype filter + `Opcode = '25'` | **1** row: S800 Chemical Handler. |
| 4 | **Sucker Pruning** | Incentive | `GET /api/incentive?icode=RP13` | Doctype filter + `Icode = 'RP13'` | Per Jay: **RP13 only**. Baseline DB returned 0; if DB has RP13, expect 1 row. |
| 5 | **Fertilizer** | Incentive | `GET /api/incentive?opcode=25` | Same as Gouging | **1** row: S800 (Opcode 25). |
| 6 | **PD Eradication** | Incentive | `GET /api/incentive?opcode=25` | Same as Gouging | **1** row: S800 (Opcode 25). |
| 7 | **CPMS** | Incentive | `GET /api/incentive?opcode=25` | Same as Gouging | **1** row: S800 (Opcode 25). |
| 8 | **Chemical Mixing** | Incentive | `GET /api/incentive?opcode=25` | Same as Gouging | **1** row: S800 (Opcode 25). |
| 9 | **Weedspray** | Incentive | `GET /api/incentive?opcode=30` | Doctype filter + `Opcode = '30'` | Rows with Opcode 30 (e.g. P14 WEEDSPRAY). |
| 10 | **Gen Services** (with op) | Incentive | `GET /api/incentive?opcode=<selected>` | Doctype filter + Opcode from selection | Match selected operation’s Opcode. |
| 11 | **Popcount** | Incentive | — | Per Jay: **remove incentive field** | No incentive dropdown. |

---

## 3. Baseline payload location

- **Activity (costcenter)**: Full response is large; use `GET http://localhost:3000/api/activity` and count rows / check all have `Active = 1` and columns `Cstctr`, `Cstctrdesc`.
- **Incentive responses**: See [baseline-api-responses.json](baseline-api-responses.json) for request URLs, SQL equivalent, and response count/sample.

---

## 4. Quick verification commands

```bash
# Costcenter (activity)
curl -s http://localhost:3000/api/activity | jq length

# Incentive (no filter)
curl -s http://localhost:3000/api/incentive | jq length

# Incentive Opcode 25 (Gouging, Fertilizer, ERAD, CPMS, Chemical Mixing)
curl -s "http://localhost:3000/api/incentive?opcode=25" | jq length

# Incentive Icode RP13 (Sucker Pruning)
curl -s "http://localhost:3000/api/incentive?icode=RP13" | jq length
```

Compare `length` and key fields (Icode, Idesc, Opcode) to the table above.
