# Sprint-Jay-Feedback — Code Deep Dive

**Purpose**: Point implementation to the right areas in **main_dar_app** (Flutter) and **DAR_Middleware** (Node/Express) for Jay feedback items and server selection.

---

## 1. Base URL and session (server selection feature)

### Flutter (main_dar_app)

- **Where base URL is set**: Look for a global config or environment used by the HTTP client, e.g.:
  - A `Config` or `AppConfig` class (e.g. `lib/core/` or `lib/config/`) holding `baseUrl` / `baseURL`.
  - Build-time or runtime assignment (e.g. `lib/main.dart` or a config loader).
- **Where to add server selection**:
  - **Pre-login screen**: The screen shown before login (splash → login). Add a **dropdown** here that lists available servers (e.g. from a static list or a small config asset).
  - On selection: set the chosen URL as the app base URL and **persist for the session** (in-memory is enough; optional: save to local storage so next launch can default to last server).
  - Ensure **all API calls** use this base URL (single HTTP client or interceptors that read from the same config).
- **Files to locate** (names may vary):
  - Splash / welcome / login screen: `lib/features/auth/` or `lib/screens/` or `lib/presentation/`.
  - API client / Dio or http: `lib/core/network/` or `lib/data/` or `lib/services/`.
  - Any `config.dart`, `env.dart`, `constants.dart` that hold base URL.

### Backend (DAR_Middleware)

- No change required for server selection: the app only switches which server (base URL) it calls. Each server instance uses its own `knex_config` / env.

---

## 2. Jay feedback — Backend (DAR_Middleware)

| Item | Where to look | Action |
|------|----------------|--------|
| **Costcenter filter** (tblActivity Active=1) | Sync or controller routes that return costcenter/list; e.g. `routes/`, `controllers/`, `/api/` activity or costcenter. | Use query `SELECT Cstctr, Cstctrdesc FROM tblActivity WHERE Active = 1`. |
| **Incentive filters** (Doctype, Opcode, Icode) | Routes that return incentives; e.g. `tblincentives` or incentive lookup. | Add/use filters: Doctype != '03','01'; Opcode = '25'; Icode = 'RP13' per operation. |
| **PD Survey header** (send failure) | `sendDARForm` for Survey/PD Survey: batch + header creation. Check `sendDARForm/survey/` or PDCM equivalent. | Debug why header fails after batch; fix payload or validation. |
| **Gouging header** (send failure) | `sendDARForm` for Gouging/PPM: header creation. | Same as above; ensure incentive and payload match backend expectations. |
| **Sucker Pruning Cycle/Sqno** | `tblDARhdr` insert/update for PPM/Sucker Pruning; ensure Cycle and Sqno columns exist and are written. | Accept Cycle, Sqno in request; persist to tblDarHdr. |
| **Chemical Mixing operation code** | `tbl_MixingOprtn` usage; mapping from costcenter/operation to OpCode. | Align OpCode sent with mobile selection; fix lookup or mapping. |
| **ERAD accomplishments** | `sendDARForm` for Eradication: accomplishment insert. | Ensure accomplishment payload is sent and inserted; fix route or validation. |
| **Popcount HAS** | Accomplishment insert for Popcount; HAS value. | Set HAS from layout actual area (ActArea) when provided by client or resolve from layout in backend. |

---

## 3. Jay feedback — Frontend (main_dar_app)

| Item | Where to look | Action |
|------|----------------|--------|
| **PD Survey materials** | Survey/PD Survey form: material list max count and header submit payload. | Max 5 materials; ensure header payload matches backend. |
| **Gouging / Sucker Pruning incentive** | Gouging and Sucker Pruning forms: incentive dropdown and payload. | Bind to filtered incentive API; send correct codes. |
| **Sucker Pruning Cycle/Sqno** | Sucker Pruning form and report model: add Cycle no., Seq no. | Add fields; include in hdr payload. |
| **Weedspray** | Weedspray form: timekeeping vs accomplishments (one time per accomplishment vs one per report). | Implement chosen design (one accomplishment per report or per-row time). |
| **CPMS decimal** | CPMS form: material quantity input. | Allow decimal (e.g. `keyboardType: TextInputType.numberWithOptions(decimal: true)`). |
| **Fertilizer** | Fertilizer form: incentive field; Operations field. | Add incentive; remove Operations field. |
| **General Services Group** | Group General Services: timekeeping rows. | Add costcenter and incentive **per row** (per employee). |
| **Planting/Replanting** | Header of Planting/Replanting form. | Add costcenter/operation dropdown (two options). |
| **Popcount** | Popcount form: incentive; HAS from layout. | Remove incentive; set HAS from selected layout’s ActArea. |

---

## 4. Suggested search patterns (Flutter)

- `baseUrl`, `baseURL`, `Config`, `apiUrl`, `BASE_URL`.
- `login`, `splash`, `auth`, `LoginScreen`, `SplashScreen`.
- `Dio`, `http`, `HttpClient`, `ApiClient`.
- Form/widget names: `Survey`, `Gouging`, `SuckerPruning`, `Weedspray`, `CPMS`, `Fertilizer`, `Popcount`, `ChemicalMixing`, `GeneralServices`, `PlantingReplanting`, `Eradication`.

---

## 5. Suggested search patterns (DAR_Middleware)

- `tblActivity`, `tblincentives`, `Cstctr`, `Doctype`, `Opcode`, `Icode`.
- `sendDARForm`, `survey`, `gouging`, `sucker`, `erad`, `popcount`, `tbl_MixingOprtn`.
- `tblDARhdr`, `Cycle`, `Sqno`, `Hdr_id`.
