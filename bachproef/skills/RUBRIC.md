# Evaluation Rubric

Detailed scoring rules for each check. Apply partial credit as specified.

---

## A — Coverage structure (25 pts)

Verifies the spec follows the mandatory test hierarchy from `e2e-testing-guidelines.md`: smoke → incremental steps → full happy path → edge cases.

| # | Check | Points | How to evaluate |
|---|---|---|---|
| A1 | Smoke test exists | 5 | The first `test()` block (lowest priority task) navigates to the route and asserts the page renders. Must be a standalone render/navigation test, not part of a larger flow. |
| A2 | Building-block tests exist | 5 | Tasks between smoke and happy path break the flow into incremental steps. Award 5 if ≥2 intermediate step tests exist. Award 3 if only 1 exists. Award 0 if the spec jumps from smoke directly to full happy path. |
| A3 | Full happy-path test exists | 10 | A single `test()` that executes the complete user journey end-to-end. Award 10 if present and covers the flow described in the plan's `source.user_story`. Award 5 if a partial happy path exists. Award 0 if absent. **If the plan type does not warrant a happy path (e.g. a static read-only page), document why in the reasoning — award full points if justification is valid.** |
| A4 | Edge-case tests exist | 5 | At least one test covers error states, validation, disabled states, empty states, or boundary conditions. Award 5 if ≥1 edge-case test exists. Award 0 if none. |

---

## B — Task execution fidelity (30 pts)

Verifies that all plan tasks were implemented and are traceable in the spec.

| # | Check | Points | How to evaluate |
|---|---|---|---|
| B1 | All tasks pass | 15 | Every task in `plan.json` has `"passes": true`. Award proportionally: `(passing_count / total_count) × 15`, rounded to nearest integer. |
| B2 | Test count matches | 8 | Count `test(` calls in the spec file. Must be ≥ number of tasks in `plan.json`. Award 8 if equal or greater. Deduct 2 per missing test, minimum 0. |
| B3 | Task IDs traceable | 7 | Each task `id` from the plan appears in the spec — as a test name, `describe` block name, or comment. Award proportionally: `(found / total) × 7`, rounded to nearest integer. |

---

## C — Locator & selector discipline (20 pts)

Verifies compliance with the `data-testid`-only locator rule from the guidelines.

| # | Check | Points | How to evaluate |
|---|---|---|---|
| C1 | No banned text locators | 7 | Spec and POM must not use `getByText(`, `getByLabel(`, `getByPlaceholder(`, or `getByRole(` with a `name:` option. **Allowed exception:** `getByRole('dialog')` and `getByRole('alert')` without `name:`. Deduct 2 per violation, minimum 0. |
| C2 | No raw CSS/XPath locators | 5 | Spec and POM must not use `locator('` with CSS selectors, `querySelector`, `$()`, `$$()`, or XPath (`//`). Deduct 2 per violation, minimum 0. |
| C3 | No arbitrary waits | 5 | No `waitForTimeout(` calls anywhere in spec or POM. Award 5 if absent. Award 0 if any found. |
| C4 | Every test has assertions | 3 | Every `test(` block contains at least one `expect(` call (directly or via POM assert methods). Deduct 1 per test without assertions, minimum 0. |

---

## D — Architecture compliance (15 pts)

Verifies the generated code follows the project structure from the guidelines.

| # | Check | Points | How to evaluate |
|---|---|---|---|
| D1 | Spec file location | 2 | File exists at `e2e/{plan-name}.spec.ts`. Award 2 if correct, 0 if missing or wrong path. |
| D2 | POM file exists | 3 | A Page Object class exists in `support/pages/` for this feature. Award 3 if present with at least navigation and assertion methods. Award 1 if the file exists but is minimal/empty. Award 0 if missing. |
| D3 | Fixture file exists | 3 | A fixture file exists in `fixtures/` using `test.extend<T>()`. Award 3 if present and exports `test`. Award 1 if the file exists but doesn't follow the `test.extend` pattern. Award 0 if missing. |
| D4 | Routes from routes.ts | 4 | Route paths in POM and spec are imported from `support/data/routes.ts` — no hardcoded path strings like `'/hr/people'` or `'/employee/'`. Award 4 if fully compliant. Deduct 1 per hardcoded route, minimum 0. |
| D5 | Auth via storageState | 3 | Feature tests do not call login functions or import `loginHelpers`. Auth is inherited from the Playwright config's `storageState` setup. Award 3 if compliant. Award 0 if manual login logic is present. |

---

## E — Data & mock integrity (10 pts)

Verifies real API usage, no mocks, and proper test data conventions.

| # | Check | Points | How to evaluate |
|---|---|---|---|
| E1 | No mock/intercept calls | 4 | Spec and POM must not use `page.route(`, `page.unroute(`, or any request interception. Award 4 if absent. Award 0 if any found. |
| E2 | No hardcoded credentials | 3 | No password strings, email literals, or credential values in spec or fixture. Credentials must come from `constants.ts` or environment variables. Award 3 if clean. Deduct 1 per violation, minimum 0. |
| E3 | E2E_ data prefix | 3 | Persistent test data (created entities, titles, names) uses the `E2E_` prefix. Award 3 if all test data follows the convention. Award 1 if partially followed. Award 0 if no `E2E_` prefix is used despite creating data. **If the test is read-only and creates no data, award full points.** |
