# Server Selection Feature — Before Login

**Story**: TS-SERVER  
**Goal**: On the page before login, the user can choose from a dropdown a list of servers; the selected value is set as the **base URL** of the Flutter app and used for the **entire session**.

---

## 1. User flow

1. User opens the app.
2. **Before** the login form (e.g. on splash or a dedicated “server selection” screen), a **Server** dropdown is shown.
3. User selects a server from the list (e.g. "UAT", "Production", "Local", or explicit URLs).
4. The selected URL is set as the app’s **base URL** (e.g. `Config.baseURL` or equivalent).
5. This base URL is used for **all** API calls for the **rest of the session** (until app is closed or user changes server, if re-selection is allowed).
6. User proceeds to login; login and all subsequent requests use the chosen base URL.

---

## 2. Functional requirements

| # | Requirement |
|---|-------------|
| 1 | Pre-login screen (or splash) shows a **dropdown** of servers. |
| 2 | Server list is configurable (e.g. static list in app, or small config file/asset). |
| 3 | Selecting a server sets the **base URL** used by the HTTP client. |
| 4 | Base URL is used for **entire session** (all API calls: login, sync, send report, etc.). |
| 5 | Optional: persist last selected server (e.g. local storage) so next app open can default to it. |
| 6 | No login possible until a server is selected (or default is applied). |

---

## 3. UI/UX (Design Authority)

- **Placement**: Same screen as login, or a step before it (e.g. splash → server selection → login).
- **Control**: Dropdown (or list picker) with readable labels (e.g. "UAT", "Production", "Local (10.0.2.2:3000)").
- **Persistence**: Session-only required; optional “Remember this server” for next launch.
- **Validation**: Ensure URL is well-formed before allowing “Continue” or “Login”.

---

## 4. Technical notes

- **Flutter**: Single source of truth for base URL (e.g. `Config.baseURL` or a service). HTTP client (Dio/http) must read from this source for every request.
- **Backend**: No change; each environment (UAT, prod, local) is a separate deployment. App only points to one at a time.
- **List of servers**: Can be hardcoded, or loaded from asset (e.g. `config/servers.json`) with labels and URLs.

---

## 5. Acceptance criteria (checklist)

- [ ] Dropdown visible before login (or on same screen above login).
- [ ] Selecting a server sets base URL for the app.
- [ ] Login request uses selected base URL.
- [ ] At least one other flow (e.g. sync or send report) uses the same base URL.
- [ ] Session uses one base URL until app is closed or server is changed (if supported).
