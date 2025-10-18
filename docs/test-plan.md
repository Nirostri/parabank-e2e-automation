# 🧩 Test Plan – Parabank E2E Automation

## 🎯 Objective
This document defines the planned test coverage, suite structure, and grouping strategy for the Parabank End-to-End Automation Framework.

---

## 🧱 Test Coverage by Layer

| Layer | Main Scenarios | Data Source | Expected Outcome |
|--------|----------------|--------------|------------------|
| **UI** | Login (positive/negative), Transfer Funds (valid/invalid), Accounts Overview display | `users.xlsx`, `transfers.xlsx` | Correct login behavior, correct balances and transfer confirmation messages |
| **API** | GET /accounts, GET /balance, POST /transfer (positive & negative) | Static test data / JSON templates | Correct HTTP codes and response validation |
| **DB** | Validate new transfer entries, verify balances updated correctly | H2 in-memory DB | Matching data between UI/API and DB tables |
| **E2E** | Login → Transfer → Validate via API → Confirm via DB | Excel + API + DB | Full functional verification of system flow |

---

## 🧮 Test ID Table (Examples)

| ID | Test Name | Priority | Layer | Data Source | Expected Result |
|----|------------|-----------|--------|--------------|----------------|
| UI_LOGIN_POS_01 | Valid Login | P1 | UI | users.xlsx | SUCCESS message |
| UI_LOGIN_NEG_01 | Invalid Password | P2 | UI | users.xlsx | Error shown |
| API_ACCOUNTS_01 | GET /accounts returns 200 | P1 | API | – | Response OK |
| DB_TRANSFER_01 | Validate Transfer Record Exists | P1 | DB | transfers.xlsx | DB entry added |
| E2E_TRANSFER_01 | Full Transfer Flow | P0 | E2E | Excel + API + DB | Flow completes successfully |

---

## 🏷️ Test Groups (TestNG)

| Group Name | Purpose | Used In |
|-------------|----------|----------|
| `ui` | All Selenium-based UI tests | Login, Transfer, Accounts Overview |
| `api` | All REST API tests | Accounts, Balance, Transfers |
| `db` | Database validation tests | Transfers repository verification |
| `e2e` | End-to-End integrated flows | UI→API→DB Transfer |
| `sanity` | Quick health-check of all layers | 1 short test per layer |
| `smoke` | Basic UI navigation only | Login + homepage |
| `negative` | Negative / validation scenarios | Login invalid, bad transfers |
| `regression` | Full coverage of system | Complete runs nightly |
| `critical` | Critical core functionality | E2E Transfer, main login |
| `data-driven` | Tests using Excel input | Login & Transfer tests |
| `fast` | Quick tests (<1min each) | sanity, smoke, api basic |
| `slow` | Heavy E2E / DB operations | full regression, nightly |

---

## 🧩 Class-to-Groups Mapping

| Test Class | Groups |
|-------------|--------|
| `LoginTest` | ui, sanity, regression, data-driven |
| `TransferFundsUiTest` | ui, regression, negative, data-driven |
| `AccountsOverviewTest` | ui, regression, fast |
| `AccountsApiTest` | api, sanity, regression, fast |
| `TransfersApiTest` | api, regression, negative, fast |
| `TransfersDbTest` | db, regression, fast |
| `UiApiDb_E2E_TransferFlow_Test` | e2e, critical, regression, slow |

---

## 🧰 Suites Definition (TestNG Suites)

| Suite Name | Groups Included | Purpose / When to Run |
|-------------|-----------------|------------------------|
| **Sanity Suite** | sanity | Morning build verification, quick health check |
| **UI Suite** | ui | Validate UI only (after front-end changes) |
| **API Suite** | api | Validate backend / contracts only |
| **DB Suite** | db | Validate data consistency in DB |
| **E2E Suite** | e2e | Full end-to-end system validation |
| **Regression Suite** | regression | Complete functional coverage |
| **Negative Suite** | negative | Validation & error-handling tests only |

---

## ⚡ Sanity Suite Composition

| Layer | Representative Test | Description |
|--------|---------------------|-------------|
| UI | `LoginTest` (positive login) | Verifies user can log in successfully |
| API | `AccountsApiTest` | Ensures account list API returns 200 |
| DB | `TransfersDbTest` | Checks DB connection & record count |
| E2E | `UiApiDb_E2E_TransferFlow_Test` | Performs one complete transfer flow |

---

## ⚖️ Priorities

- **P0 – Critical:** E2E Transfer Flow
- **P1 – Core:** Positive Login, Transfer Success, API Balance
- **P2 – Negative:** Validation, error messages, input edge cases

---

## 📋 Summary
- Each test is tagged with at least one core group (`ui`, `api`, `db`, or `e2e`).
- Suites are configured to run by group for flexible CI pipelines.
- Sanity suite ensures daily quick health check.
- Regression suite provides full coverage for nightly runs.

---

# 📋 Basic Test Matrix (MVP)

> Scope: UI, API, DB, E2E — high-level list (no steps).

| ID | Title | Priority | Data Source | Expected |
|---|---|---|---|---|
| **UI_LOGIN_POS_01** | UI – Valid login navigates to Accounts Overview | **P1** | `users.xlsx` | SUCCESS |
| **UI_LOGIN_NEG_01** | UI – Invalid password shows error | **P2** | `users.xlsx` | ERROR |
| **UI_LOGIN_NEG_02** | UI – Unknown user shows error | **P2** | `users.xlsx` | ERROR |
| **UI_ACCOUNTS_POS_01** | UI – Accounts Overview displays accounts & balances | **P2** | — | VISIBLE |
| **UI_TRANSFER_POS_01** | UI – Transfer valid amount shows success message | **P1** | `transfers.xlsx` | SUCCESS |
| **UI_TRANSFER_NEG_01** | UI – Amount = 0 shows validation error | **P2** | `transfers.xlsx` | ERROR |
| **UI_TRANSFER_NEG_02** | UI – Negative amount shows validation error | **P2** | `transfers.xlsx` | ERROR |
| **UI_TRANSFER_NEG_03** | UI – Insufficient funds shows error | **P2** | `transfers.xlsx` | ERROR |
| **API_ACCOUNTS_POS_01** | API – GET /accounts returns 200 & non-empty list | **P1** | — | 200 |
| **API_BALANCE_POS_01** | API – GET balance by account returns 200 & numeric balance | **P1** | — | 200 |
| **API_BALANCE_NEG_01** | API – GET balance for unknown account returns 404/error | **P2** | — | 404/ERR |
| **API_TRANSFER_POS_01** | API – POST transfer valid payload returns 200/201 | **P1** | `transfers.xlsx` | 200/201 |
| **API_TRANSFER_NEG_01** | API – POST transfer invalid amount returns 4xx/error body | **P2** | `transfers.xlsx` | 4XX |
| **DB_TRANSFER_POS_01** | DB – After success, TRANSFERS count increased by 1 | **P1** | `transfers.xlsx` | +1 |
| **DB_TRANSFER_NEG_01** | DB – After invalid transfer, no new row is inserted | **P2** | `transfers.xlsx` | NO-CHANGE |
| **DB_TRANSFER_POS_02** | DB – Last transfer row matches from/to/amount/currency | **P2** | `transfers.xlsx` | MATCH |
| **E2E_TRANSFER_P0_01** | E2E – Full transfer flow UI→API→DB | **P0** | `transfers.xlsx` | PASS |

---

### Notes
- **P0**: Critical business flow (E2E transfer).
- **P1**: Core success paths (valid login, valid transfer, core API calls, DB count).
- **P2**: Negative/edge validations and supportive checks.
- `users.xlsx` sheet: `login` • `transfers.xlsx` sheet: `cases`.
- “Expected” is intentionally short; detailed assertions will be documented in test specs.

