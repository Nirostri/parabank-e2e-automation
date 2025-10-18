# Parabank E2E Automation Project

## Overview
This is a Java-based automation framework that tests Parabank (https://parabank.parasoft.com/)
using Selenium, RestAssured, TestNG, MySQL and Excel data-driven testing.

## Project Goals
- Demonstrate UI + API + DB integration.
- Practice Java automation architecture.
- Produce professional Allure reports.

## Technologies
Java 17, Selenium, RestAssured, TestNG, Maven, Apache POI, MySQL Database, Allure Reports.

## Structure
(src/main/java - framework code)
(src/test/java - test cases)
(config - environment setup)
(testdata - Excel files)
(docs - documentation)

## 🧾 Test Data
The framework uses Excel-based data for parameterized tests.

**1️⃣ login data (`testdata/users.xlsx`)**
- Sheet name: `login`
- Columns: Username, Password, ExpectedOutcome
- Used in: Login tests (UI & API)

**2️⃣ transfer data (`testdata/transfers.xlsx`)**
- Sheet name: `cases`
- Columns: FromAccount, ToAccount, Amount, Currency, ExpectedMessage, ShouldAffectBalance
- Used in: Transfer funds tests (UI, API, DB integration)

## 🧪 TestNG Suites & Groups

This project follows a clear grouping and suite structure using **TestNG** annotations.

### 🔖 Groups
Each test method is assigned one or more groups to allow selective execution:
- `ui` – All Selenium UI tests
- `api` – REST API tests
- `db` – Database validation tests
- `e2e` – End-to-End flows (UI→API→DB)
- `sanity` – Quick smoke from each layer
- `regression` – Full coverage runs
- `negative` – Negative/validation scenarios
- `critical` – Core functionality
- `data-driven` – Tests using Excel inputs

Example (in code, later):
```java
@Test(groups = {"ui", "sanity"})
public void validLoginTest() { ... }
```

## 🧭 Architecture Overview

This framework validates Parabank end-to-end across **UI, API, and DB**:

- **UI (Selenium/POM):** Login, Accounts Overview, Transfer Funds.
- **API (RestAssured):** Logical endpoints – GET accounts, GET balance by account, POST transfer (positive & negative).
- **DB (H2/JDBC):** Post-action verification in logical tables `ACCOUNTS` and `TRANSFERS`.
- **Data (Excel):** `users.xlsx` (login), `transfers.xlsx` (transfer scenarios).

### Core E2E Flow
1) Login via UI → 2) Make a transfer via UI → 3) Validate balances via API → 4) Verify transfer record in DB.  
   This flow is tagged as **`e2e`** and represents the **P0 critical** path of the system.

> Exact API paths will be mapped during implementation; method names in the API client follow the logical contract listed above.

## 🌍 Environments

The framework supports multiple environments, configured via `.properties` files located in the `config/` directory.

### Available Environment Files
- **config.dev.properties** → Used for local development and test runs.
- **config.stage.properties** → Reserved for staging/pre-production testing.

### Common Keys (Properties)
| Key | Description | Example Value |
|------|-------------|----------------|
| `baseUrl` | Base URL of the Parabank web application | `https://parabank.parasoft.com` |
| `browser` | Browser type for Selenium execution | `chrome` |
| `headless` | Run browser in headless mode | `false` |
| `explicitTimeoutSeconds` | Maximum wait time for element visibility | `15` |
| `api.baseUri` | Root URI for API endpoints | `https://parabank.parasoft.com/parabank/services` |
| `db.url` | JDBC connection string (H2/MySQL) | `jdbc:h2:mem:parabank` |
| `db.user` | Database username | `sa` |
| `db.pass` | Database password | *(empty)* |
| `data.users` | Path to Excel file containing login test data | `testdata/users.xlsx` |
| `data.transfers` | Path to Excel file containing transfer test data | `testdata/transfers.xlsx` |

### How Environment Selection Works
By default, the framework uses the `dev` environment.  
Later, when CI/CD is integrated, you will be able to run with a specific profile:

```bash
mvn clean test -Denv=stage

