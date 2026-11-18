# SendItCycles-UTA-API-Tests

API tests for the SendIt Cycles demo backend, written in the HCL DevOps Test Hub test DSL (`.dtx.yaml` / `.dts.yaml`).

Component folder: `SendItCycles-API/`. Default target: `http://10.134.60.215:3000/api`. Change the `BASE_URL` var in a test
(or pass it in from a suite) to point at another instance, for example `http://localhost:5000/api`.

## Layout

| Folder / file | Contents |
|---|---|
| `1 Atomic/` | Single read-only calls: health, categories, products, product by id, 404 cases, image |
| `2 Auth/` | Register, login (valid / wrong password / unknown / missing), profile, token errors, admin access control |
| `3 Workflows/` | Chained flows: register-login-profile, catalog consistency, order totals and stock, two-product order, order isolation between users, stock and cart validation |
| `Known Issues/` | Tests that assert the *correct* behaviour of defects currently on the demo host, so they FAIL until fixed |
| `modules/Register And Login.dtx.yaml` | Registers a throwaway user with generated credentials and logs in; returns `TOKEN`, `USER_ID`, `EMAIL`, `PASSWORD` |
| `schemas/senditcycles-openapi.yaml` | OpenAPI 3 description of all 25 operations (has `operationId`s) |
| `_Atomic Suite`, `_Auth Suite`, `_Workflow Suite` | Runs each group |
| `_All API Tests Suite` | Atomic + Auth + Workflow (excludes Known Issues) |
| `_Known Issues Suite` | The three expected-to-fail tests |

## What the tests write

- `_Atomic Suite`: nothing.
- `_Auth Suite` and `_Workflow Suite`: throwaway users (`apitest+<timestamp>-<n>@example.com`, random generated password).
- `_Workflow Suite`: orders. Each run reduces stock of products 1 (by 2 and 1), 2 (by 2) and 3 (by 1). The demo data has thousands of
  units, but do not point these tests at an environment where stock matters.

Tests that compare stock before and after an order assume nothing else is ordering the same product at that moment,
so do not run the workflow suite concurrently with other order traffic.

## Assumptions

- Seed data: products 1, 2, 3 and 5 exist, product 5 has far fewer than 1,000,000 units, and product 1 and 2 have enough stock.
- Passwords are generated per run and never stored; admin routes are only exercised with a non-admin token or no token.
- Admin write operations (create/update/delete products, categories, order status, bulk) are described in the OpenAPI file
  but have no tests here because they need an admin token.

## Known issues covered by `Known Issues/`

| Test | Expected | Demo host currently returns |
|---|---|---|
| `GET Products By Category` | 200 with the category's products | 500 `Database error` (SQL alias bug in `getProductsByCategory`) |
| `POST Register Duplicate Email` | 409 `Email already exists` | 500 `Database error` (handler only maps the Postgres duplicate code; host runs SQLite) |
| `POST Login Malformed JSON` | 400 | 500 `Internal server error` |
