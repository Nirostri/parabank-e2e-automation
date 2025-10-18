r# 🧭 Architecture – Parabank E2E Automation

This document describes the testing architecture, layers, planned flows, and logical API/DB mapping used by the framework.

---

## 1) High-Level View

**Goal:** Validate Parabank using **UI + API + DB** with **Excel data** and **TestNG** groups/suites.

**Layers:**
- **UI (Selenium + POM):** Login, Accounts Overview, Transfer Funds.
- **API (RestAssured):** Accounts, Balance, Transfer actions and validations.
- **DB (H2/JDBC):** Post-action verification (records & balances).
- **Data (Excel):** Users & Transfers input sheets drive parameterized tests.
- **Orchestration (TestNG):** Groups (`ui`, `api`, `db`, `e2e`, …) and suites (Sanity/UI/API/DB/E2E/Regression).

---

## 2) Pages Under Test (UI scope)

- **Login Page**
    - Inputs: username, password
    - Actions: submit
    - Expected: success message / landing on Accounts Overview; error messages for invalid input

- **Accounts Overview Page**
    - Displays: account list, balances, links to details
    - Expected: at least one active account; total balance visible

- **Transfer Funds Page**
    - Inputs: from-account, to-account, amount (and currency if shown)
    - Actions: submit transfer
    - Expected: success message, updated balances

> Notes: Exact locators & labels are defined in Page Objects (not here). This file is conceptual.

---

## 3) Planned API (Logical endpoints)

> These are **logical** endpoints you will map to the actual Parabank endpoints during implementation.  
> Keep names stable even if URL paths differ; the client methods will reflect this contract.

- **Accounts**
    - **GET `GET_ACCOUNTS`**  
      Purpose: Get list of user accounts  
      Success: `200`, array of accounts with `id`, `number`, `type`, `balance`

    - **GET `GET_BALANCE_BY_ACCOUNT_ID`**  
      Purpose: Get balance for a specific account  
      Path Var: `accountId`  
      Success: `200`, body contains `balance` (numeric)

- **Transfers**
    - **POST `CREATE_TRANSFER`**  
      Purpose: Transfer funds between accounts  
      Body: `fromAccount`, `toAccount`, `amount`, `currency` (optional)  
      Success: `200/201`, body contains `transferId` or confirmation fields

- **Negative/Validation**
    - **POST `CREATE_TRANSFER` (invalid amount)** → expect `4xx` and error payload
    - **POST `CREATE_TRANSFER` (insufficient funds)** → expect `4xx` and explicit error
    - **GET `GET_BALANCE` (unknown account)** → expect `404` or error schema

**Common Response Fields (logical)**
- `status` (optional)
- `error.code`, `error.message` (for 4xx)
- `account.id`, `account.number`, `account.balance`
- `transfer.id`, `transfer.createdAt`

> During implementation, record the actual base URI and concrete paths in `api/BaseApi` and update this doc if necessary.

---

## 4) Planned DB (H2 logical schema)

We use H2 in-memory as a **verification DB** (seeded or mirrored data).  
Logical tables used for assertions:

- **ACCOUNTS**
    - `ID` (PK)
    - `ACCOUNT_NUMBER` (unique)
    - `TYPE` (CHECKING/SAVINGS)
    - `BALANCE` (DECIMAL(15,2))
    - `UPDATED_AT` (TIMESTAMP)

- **TRANSFERS**
    - `ID` (PK)
    - `FROM_ACCOUNT` (FK → ACCOUNTS.ACCOUNT_NUMBER)
    - `TO_ACCOUNT` (FK → ACCOUNTS.ACCOUNT_NUMBER)
    - `AMOUNT` (DECIMAL(15,2))
    - `CURRENCY` (VARCHAR)
    - `CREATED_AT` (TIMESTAMP)

**Primary DB Validations**
- After a successful transfer:
    - `TRANSFERS` row is inserted with correct fields.
    - `ACCOUNTS.BALANCE` adjusted for both `FROM_ACCOUNT` (−amount) and `TO_ACCOUNT` (+amount).
- For negative transfers:
    - No new row in `TRANSFERS`.
    - No change in `ACCOUNTS.BALANCE`.

> In early MVP, you may assert **counts** (e.g., `count(TRANSFERS)`) and **last inserted row**; later you can extend to balance diffs.

---

## 5) Data Sources (Excel)

- **`testdata/users.xlsx`** (Sheet: `login`)
    - Columns: `Username`, `Password`, `ExpectedOutcome` (`SUCCESS` / `INVALID_PASSWORD` / `USER_NOT_FOUND`)
    - Used in: UI Login (positive/negative), optional API login/authorization if applicable

- **`testdata/transfers.xlsx`** (Sheet: `cases`)
    - Columns: `FromAccount`, `ToAccount`, `Amount`, `Currency`, `ExpectedMessage`, `ShouldAffectBalance` (`YES`/`NO`)
    - Used in: UI Transfer, API Transfer, DB validation, E2E transfer flow

---

## 6) Core Flows (E2E + Layered)

### 6.1 E2E – Transfer Flow (Flagship)
1. **UI:** Login with valid user (from Excel).
2. **UI:** Submit transfer (from Excel row).
3. **API:** Fetch balance(s) for involved accounts; verify change reflects UI action.
4. **DB:** Verify `TRANSFERS` has a new record and (optionally) balances reflect the operation.

**Pass Criteria:**
- Success message on UI.
- API balances updated as expected.
- `TRANSFERS` new row exists (`ShouldAffectBalance = YES`).

### 6.2 UI Flows
- **Login Positive/Negative:** validate success path and error messages.
- **Accounts Overview:** account list and balances visible.
- **Transfer Funds:** success message for valid amounts; error for invalid/insufficient funds.

### 6.3 API Flows
- **GET Accounts:** status `200`, non-empty list, required fields exist.
- **GET Balance by Account:** status `200`, numeric balance; `404` for unknown id.
- **POST Transfer:** success path returns confirmation; negative cases return meaningful `4xx`.

### 6.4 DB Checks
- **Transfers Count:** increases by 1 on success.
- **Last Transfer Row:** matches Excel inputs (from/to/amount/currency).
- **Balances Consistency (optional for MVP):** from-account decreased, to-account increased.

---

## 7) Traceability (Flow ↔ Data ↔ Tests)

| Flow | Data Source | UI Tests | API Tests | DB Tests | E2E Test |
|------|-------------|----------|-----------|----------|----------|
| Login (pos/neg) | `users.xlsx` | `LoginTest` | – | – | – |
| Transfer (pos/neg) | `transfers.xlsx` | `TransferFundsUiTest` | `TransfersApiTest` | `TransfersDbTest` | `UiApiDb_E2E_TransferFlow_Test` |
| Accounts Overview | – | `AccountsOverviewTest` | `AccountsApiTest` | – | – |
| Balance Validation | `transfers.xlsx` (account ids) | – | `AccountsApiTest` | – | `UiApiDb_E2E_TransferFlow_Test` |

---

## 8) Negative & Edge Strategy

- **Invalid Credentials:** expect UI error, no session.
- **Invalid Amounts (0 / negative / too large):** UI/API error; DB unchanged.
- **Unknown Accounts:** API `404` / error body; UI prevents submission.
- **Concurrency/Repeat (optional):** Same transfer twice → idempotency/consistent state.

---

## 9) Non-Functional Assumptions (MVP)

- **Timeouts:** UI explicit waits up to 10–20s; API timeouts up to 10s.
- **Data Independence:** Each test row in Excel considered independent; no cross-row dependency.
- **Environment:** `dev` profile local; `stage` reserved for later.

---

## 10) Open Items for v2 (Future Work)

- Real DB seeding (Flyway/Liquibase).
- Contract validation (JSON Schema).
- Dockerized local grid + pipelines artifacts hosting.
- Visual assertions on key screens.
