---
name: e2e-testing-guidelines
description: Apply Playwright e2e test guidelines: fixtures via test.extend, POM pages with inline locators, real API only, auth via storageState, data-testid locators, helpers and typed data in support/.
---

# E2E Testing with Playwright

This skill governs how to write, structure, and maintain end-to-end tests in `apps/client/playwright/`.
Read the full file before generating any test code.

---

## 1 — Philosophy

### 1.1 Research first

Before implementing, **read the relevant Playwright documentation**:

- [Best practices](https://playwright.dev/docs/best-practices)
- [Fixtures](https://playwright.dev/docs/test-fixtures)
- [Page Object Model](https://playwright.dev/docs/pom)
- [Locators](https://playwright.dev/docs/locators)
- [Assertions](https://playwright.dev/docs/test-assertions)
- [Authentication](https://playwright.dev/docs/auth)
- [Parallelism & sharding](https://playwright.dev/docs/test-parallel)

If the Playwright docs and this skill file conflict, **this skill file wins** — it encodes
project-specific decisions made on top of Playwright defaults.

### 1.2 Core principles

| Principle | What it means |
|---|---|
| **Test user-visible behaviour** | Click what users click, assert what users see. Never test implementation details. |
| **Isolate every test** | Each test runs independently — no shared state, no execution-order dependency. |
| **Real API, no mocks** | E2E tests hit the real staging API. Only mock external third-party services. |
| **`data-testid` everywhere** | This is a multilingual app (lingui / i18n). Never locate by visible text. |
| **One behaviour per test** | A test should verify one thing. Multiple assertions are fine if they support a single behaviour. |
| **Routes** | Routes come from the routes.ts file. If they aren't present don't invent the route. Prompt the user to give you the correct route(s). |

### 1.3 Good tests vs bad tests

**Good test — tests observable behaviour:**

```ts
test('save persists the new job and shows it in the list', async ({ page, myData }) => {
  const careerPage = new CareerPathPage(page)
  await careerPage.goToCareerPath(myData.appUrl)
  await careerPage.startEditingCurrentJob()
  await careerPage.addJobViaSearch('E2E_Test Job')
  await careerPage.clickSave()
  await careerPage.assertJobVisible('E2E_Test Job')
})
```

**Bad test — tests implementation details:**

```ts
// ❌ Checks internal state instead of what the user sees
test('save updates sessionStorage', async ({ page }) => {
  // ...
  const raw = await page.evaluate(() => sessionStorage.getItem('career'))
  expect(JSON.parse(raw).jobs).toHaveLength(2)
})

// ❌ Locates by translated text — breaks in other languages
test('shows heading', async ({ page }) => {
  await expect(page.getByText('Career Path')).toBeVisible()
})

// ❌ Depends on CSS class — breaks when designers refactor
test('button is styled', async ({ page }) => {
  await expect(page.locator('.btn-primary')).toBeVisible()
})
```

---

## 2 — Workflow

**Do not write any test code until steps 2.1–2.4 are complete.**

### 2.1 Identify critical user journeys

Ask the user: *"Which user journeys are most critical to test?"*

You can't test everything. Get the user to confirm which flows matter most.

### 2.2 Map pages and interactions

For each journey, list:

- The pages visited.
- The interactions performed (click, fill, navigate, assert).

Present this to the user. They confirm, add, or remove items.

### 2.3 Build the test plan

Present a structured plan following this **mandatory hierarchy**:

```markdown
## Test Plan: <Feature Name>

### 1. Smoke test
- [ ] Route renders — navigate to the route, assert key layout elements are visible via data-testid

### 2. Incremental happy flow steps
- [ ] Step 1 — first interaction/state from the happy flow
- [ ] Step 2 — next interaction/state from the happy flow
- [ ] ...

### 3. Full happy path (end-to-end)
- [ ] Complete happy flow — all steps in sequence, assert final outcome

### 4. Validation & edge cases
- [ ] Test name — description
```

**Hierarchy rules:**
- The **first test is always a smoke test** — navigate to the route and verify the page renders with key sections visible. For parent routes with child tabs, each child route also gets a smoke test.
- **Incremental step tests** break the user story's happy flow into individual interactions. Each step is its own test, ordered sequentially.
- The **full happy path** executes the entire happy flow end-to-end in a single test. Redundancy with step tests is intentional — it proves the integrated sequence works.
- **Edge cases and validation** come last — error states, boundary conditions, and acceptance criteria not covered by the happy flow.

### 2.4 Get user approval

**Wait for explicit approval.** The user may add, remove, or reorder tests.
Only after approval, proceed to implementation in the listed order.

### 2.5 Tracer bullet — first test must pass before continuing

Implement the first (highest-priority) task:

1. Add `data-testid` attributes to the React component if missing.
2. Create or update the POM page in `support/pages/`.
3. Create or update the fixture in `fixtures/`.
4. Add any missing routes to `support/data/routes.ts`.
5. Write the test in `e2e/<feature>.spec.ts`.
6. Run the spec file: `cd apps/client && npx playwright test e2e/<spec-file>.spec.ts`

Apply the **RED → GREEN → REFACTOR** loop:
- **RED** — the test fails (expected — nothing is implemented yet).
- **GREEN** — write the minimum code to make it pass. Do not over-engineer.
- **REFACTOR** — once green, clean up any duplication or rough edges. Re-run after each refactor step to stay green.

**Never refactor while RED.** Get to GREEN first, then clean up.

This is your tracer bullet — it proves the path works end-to-end. **Do not implement remaining tasks until the first test is green.**

Commit when green.

### 2.6 Implement remaining tasks

For each remaining task, in priority order, apply the same RED → GREEN → REFACTOR loop:

1. Add `data-testid` attributes to the React component if missing.
2. Create or update the POM page and fixture as needed.
3. Write or update the test in `e2e/<feature>.spec.ts`.
4. **RED** — confirm the test fails before implementing.
5. **GREEN** — write the minimum code to make it pass.
6. **REFACTOR** — clean up once green, re-run to confirm still green.
7. Commit each task separately.

### 2.7 Verify

Once all tasks are implemented, run the full spec file:

```bash
cd apps/client && npx playwright test e2e/<spec-file>.spec.ts
```

Fix any failures and re-run until it is green. Only then run the full suite:

```bash
cd apps/client && npm run test:e2e
```

### 2.8 Cycle checklist

After each test:

- [ ] Test describes behaviour, not implementation.
- [ ] Code is minimal for this test — no speculative features.
- [ ] All elements use `data-testid` locators.
- [ ] No hardcoded credentials, URLs, or route strings.
- [ ] Routes come from `routes.ts`.

---

## 3 — Project Structure

```markdown
playwright/
├── .auth/                    # Generated storageState (gitignored)
├── e2e/                      # Spec files — *.spec.ts
├── fixtures/                 # Fixture definitions — *.fixtures.ts
├── skill/                    # This skill file
└── support/
    ├── auth.setup.ts         # One-time login, saves storageState
    ├── data/
    │   ├── constants.ts      # Environment variables, static values
    │   ├── routes.ts         # Centralised route constants
    │   └── sharedTypes.ts    # Shared TypeScript types
    ├── helpers/              # Reusable utility functions
    └── pages/                # Page Object classes
```

### Where each piece goes

| What | Where | Example |
|---|---|---|
| Test spec | `e2e/<feature>.spec.ts` | `e2e/careerpath.spec.ts` |
| Fixture | `fixtures/<feature>.fixtures.ts` | `fixtures/careerPath.fixtures.ts` |
| Page object | `support/pages/<feature>Page.ts` | `support/pages/careerPathPage.ts` |
| Helper function | `support/helpers/<name>.ts` | `support/helpers/loginHelpers.ts` |
| Route constants | `support/data/routes.ts` | Single file, always |
| Static constants | `support/data/constants.ts` | Single file, always |
| Shared types | `support/data/sharedTypes.ts` | Single file, always |

**Rule:** Spec files must not contain fixture setup, locator definitions, route strings, or reusable
logic. Move those to the appropriate location above.

---

## 4 — Locators: `data-testid` Only

### 4.1 The rule

**This is a multilingual app. All visible text is translated. Never locate elements by text.**

The only allowed locator method is:

```ts
page.getByTestId('feature-element')
page.getByTestId(`feature-item-${dynamicId}`)
```

### 4.2 Adding `data-testid` to components

When writing a test, you MUST also add `data-testid` to the React component if it's missing.

**Naming convention:**

```markdown
data-testid="<feature>-<element>"              // static
data-testid="<feature>-<element>-<dynamicId>"   // list item
```

**Example from the codebase:**

```tsx
// Static
<button data-testid="career-save">...</button>
<button data-testid="career-cancel">...</button>
<div data-testid="career-past-section">...</div>

// Dynamic — append the unique ID
<tr data-testid={`people-row-${person.id}`}>...</tr>
<input data-testid={`coach-settings-level-checkbox-${level}`} />
```

### 4.3 Rules

1. Always use `page.getByTestId()`.
2. Add `data-testid` to the component if missing — never skip this step.
3. Place `data-testid` on the actual rendered DOM element, not a wrapper that gets stripped.
4. Use `data-testid` strings directly inline in page objects — no separate constants file for test IDs.
5. IDs must be unique per page for single elements, or follow the `<feature>-<element>-<id>` prefix
   pattern for lists.

### 4.4 Banned locators

| Method | Why |
|---|---|
| `getByText('...')` | Text is translated per locale |
| `getByRole('...', { name: '...' })` | `name` matches visible text — same problem |
| `getByLabel('...')` | Label text is translated |
| `getByPlaceholder('...')` | Placeholder text is translated |
| `locator('.css-class')` | Breaks when markup changes |
| XPath | Fragile and unreadable |

**Sole exception:** `page.getByRole('dialog')` or `page.getByRole('alert')` (structural role without
text) for third-party components you cannot modify.

**If you're reaching for anything other than `getByTestId`, stop and add a `data-testid` first.**

---

## 5 — Authentication

All feature tests inherit authentication automatically via `storageState`.

### How it works

`support/auth.setup.ts` runs once via the `setup` project. It logs in and saves browser state to
`.auth/user.json`. The `chromium` project depends on `setup` and loads that state for every test.

```ts
// playwright.config.ts (relevant excerpt)
{ name: 'setup', testMatch: 'support/auth.setup.ts' },
{
  name: 'chromium',
  testMatch: 'e2e/**/*.spec.ts',
  testIgnore: 'e2e/login.spec.ts',
  dependencies: ['setup'],
  use: { storageState: 'playwright/.auth/user.json' },
},
{ name: 'chromium-public', testMatch: 'e2e/login.spec.ts' },
```

### Rules

- **Feature tests** (everything except `login.spec.ts`): Do NOT call login functions. The page is
  already authenticated.
- **`loginHelpers.ts`** exists only for `login.spec.ts` — testing the login/logout flow itself.
- **`auth.fixtures.ts`** exists only for `login.spec.ts`.
- Credentials come from `.env.test` via `constants.ts` — never hardcode them.

---

## 6 — Fixtures

Fixtures provide typed test data and per-test setup/teardown.

### When to create a fixture

- The test needs feature-specific data (IDs, URLs, computed dates).
- Setup or teardown is needed before/after each test.
- The data will be reused across multiple tests in the same file.

### Convention

File: `fixtures/<feature>.fixtures.ts`

1. Define a TypeScript type for the fixture data.
2. Extend Playwright's `test` using `test.extend<T>()`.
3. Export the extended `test` and re-export `expect`.

### Example — from `careerPath.fixtures.ts` (generalised)

```ts
import { test as base } from '@playwright/test'
import { APP_URL } from '../support/data/constants'

export type FeatureData = {
  appUrl: string
}

type FeatureFixtures = {
  featureData: FeatureData
  _featureSetup: void
}

export const test = base.extend<FeatureFixtures>({
  featureData: async ({}, use) => {
    await use({ appUrl: APP_URL })
  },

  // Auto-fixture: runs before each test without being explicitly requested
  _featureSetup: [
    async ({ page }, use) => {
      // Setup: normalise browser state before the test
      await page.addInitScript(() => {
        localStorage.setItem('sectionExpanded', 'true')
      })
      await use()
      // Teardown: clean up after the test (optional)
    },
    { auto: true },
  ],
})

export { expect } from '@playwright/test'
```

### Auto-fixtures vs `beforeEach`

Use `{ auto: true }` fixtures for automatic setup/teardown. **Do not use `test.beforeEach()` at module
level in fixture files.**

### Composing fixtures

When a test needs data from two feature fixtures:

```ts
import { mergeTests } from '@playwright/test'
import { test as featureATest } from './featureA.fixtures'
import { test as featureBTest } from './featureB.fixtures'

export const test = mergeTests(featureATest, featureBTest)
export { expect } from '@playwright/test'
```

---

## 7 — Page Objects (POM)

Page objects encapsulate all interactions with a page or feature area.

### Convention

- File: `support/pages/<feature>Page.ts`
- One class per feature page or feature flow.
- Constructor receives `page: Page`.
- Locators are inline in methods (not in a separate file) — most are parameterised by dynamic IDs.
- Create a new instance per test — never share across tests.

### Example — from `careerPathPage.ts` (generalised)

```ts
import { Page, expect } from '@playwright/test'
import { ROUTES } from '../data/routes'

export class FeaturePage {
  constructor(private page: Page) {}

  // Navigation — go to the feature page
  async goTo(appUrl: string): Promise<void> {
    await this.page.goto(`${appUrl}${ROUTES.HR_PEOPLE}`)
    await this.page.getByTestId('people-row-first').click()
    await this.page.getByTestId('feature-tab').click()
    await expect(this.page.getByTestId('feature-section')).toBeVisible()
  }

  // Assertion guard — confirm we landed on the right route
  async assertOnRoute(): Promise<void> {
    await expect(this.page).toHaveURL(/\/feature/)
  }

  // Interaction — click a button
  async clickSave(): Promise<void> {
    const btn = this.page.getByTestId('feature-save')
    await expect(btn).toBeEnabled()
    await btn.click()
  }

  // Interaction with dynamic ID
  async toggleItem(itemId: string): Promise<void> {
    const checkbox = this.page.getByTestId(`feature-item-checkbox-${itemId}`)
    await expect(checkbox).toBeVisible()
    await checkbox.click()
  }

  // Assertion guard — check element visibility
  async assertItemChecked(itemId: string): Promise<void> {
    await expect(this.page.getByTestId(`feature-item-checkbox-${itemId}`)).toBeChecked()
  }
}
```

### What belongs in a POM method

| ✅ Include | ❌ Exclude |
|---|---|
| Navigation (`page.goto`, tab clicks) | Business assertions (`expect(value).toBe(...)`) |
| Click, fill, select interactions | Data setup / API calls |
| Navigation guards (`toBeVisible` after nav) | Test-specific branching logic |
| State reads (`sessionStorage`, `localStorage`) | Unrelated page interactions |

---

## 8 — Helpers

Helpers are standalone utility functions for actions reused across multiple page objects or specs.

### Convention

- File: `support/helpers/<name>.ts`
- Functions must be stateless — no module-level variables.
- Use a helper when the same multi-step action appears in multiple page classes.

### Example:

```ts
// support/helpers/loginHelpers.ts
import { Page } from '@playwright/test'

export async function goToSigninPage(page: Page, appUrl: string): Promise<void> {
  await page.goto(`${appUrl}/signin`)
  await page.getByTestId('cookie-consent-accept').click()
}
```

---

## 9 — Data (`support/data/`)

### 9.1 Routes — `routes.ts`

**All route paths MUST come from `support/data/routes.ts`.**

```ts
import { ROUTES, ROUTE_GLOBS } from '../support/data/routes'

// ✅ Use the constant
await page.goto(`${appUrl}${ROUTES.HR_PEOPLE}`)
await page.waitForURL(ROUTE_GLOBS.HR_PEOPLE)

// ❌ Never hardcode
await page.goto(`${appUrl}/hr/people`)
```

**Rules:**

1. Check `routes.ts` before writing any navigation. If the route exists, use it.
2. If the route does **not** exist, **ask the user** for the correct path. Do not guess.
3. If a route from `routes.ts` doesn't work (404, redirect), **stop and ask the user**.
4. Dynamic segments are assembled at the call site: `` `${ROUTES.HR_PEOPLE}/${personId}/career-path` ``
5. `ROUTE_GLOBS` are for `waitForURL` only — never pass to `page.goto`.

### 9.2 Constants — `constants.ts`

Environment variables and static values shared across fixtures:

```ts
import { APP_URL, VALID_EMAIL, ALL_LEVELS } from '../support/data/constants'
```

Never read `process.env` directly in specs or page objects — access it through `constants.ts`.

### 9.3 Shared types — `sharedTypes.ts`

TypeScript types used across multiple files:

```ts
import { ExpertiseLevel } from '../support/data/sharedTypes'
```

### 9.4 Dates — use `date-fns`

Always use `date-fns` for date manipulation. Never use raw `Date` arithmetic.

```ts
// ✅ Correct
import { addDays, format, startOfToday } from 'date-fns'
const endDate = addDays(startOfToday(), 14)

// ❌ Wrong
const endDate = new Date(Date.now() + 14 * 86400000)
```

### 9.5 Test-created data — prefix with `E2E_`

Any data persisted to the database must start with `E2E_` so it can be identified and cleaned up.

```ts
// ✅ Identifiable
await page.getByTestId('plan-title-input').fill('E2E_Test Career Plan')

// ❌ Indistinguishable from real data
await page.getByTestId('plan-title-input').fill('Career Plan')
```

---

## 10 — API Strategy

### Real API only

| Test level | API | Why |
|---|---|---|
| Unit / Component | Mock | Speed, isolation |
| **E2E (this skill)** | **Real staging API** | Validates end-to-end data flow, auth, business logic |

- Do not use `graphqlMock.ts` or `buildDevPlanWizardHandlers` in new tests.
- Tests run against the staging environment configured in `.env.test`.
- Use real entity IDs from the staging database. Store them in `constants.ts` or fixtures.

### Exception — third-party services

Mock external services you don't control (analytics, payments) to avoid flakiness:

```ts
await page.route('**/analytics.thirdparty.com/**', route => route.fulfill({ status: 200 }))
```

---

## 11 — Writing Tests

### File naming

`e2e/<featureName>.spec.ts` — name after the feature, not the component.

### Structure

```ts
import { test, expect } from '../fixtures/careerPath.fixtures'
import { CareerPathPage } from '../support/pages/careerPathPage'

test.describe('Career Path', () => {
  test.describe('Page render', () => {
    test('loads the career path page via the Career tab', async ({ page, careerPath }) => {
      const careerPage = new CareerPathPage(page)
      await careerPage.goToCareerPath(careerPath.appUrl)
      await careerPage.assertOnCareerPathRoute()
    })
  })

  test.describe('Current job section', () => {
    test('enters edit mode and shows the sticky footer', async ({ page, careerPath }) => {
      const careerPage = new CareerPathPage(page)
      await careerPage.goToCareerPath(careerPath.appUrl)
      await careerPage.startEditingCurrentJob()
      await careerPage.assertStickyFooterVisible()
    })
  })
})
```

### Test isolation

- Each test is independently runnable.
- No shared state between tests via module-level variables.
- No reliance on test execution order.
- Clean up any data your test creates (use fixture teardown or `E2E_` prefix for manual cleanup).
- When a test mutates shared staging records, make setup idempotent: remove or reset prior test-created
  residue before the mutation, then verify teardown returns the record to its baseline.

### Assertions — visibility, state, and values only

**Never assert on text content** — this is a multilingual app. Text changes by language. Assert on visibility, enabled state, checked state, and input values instead.

```ts
// ✅ Safe — visibility and state (multilingual-compatible)
await expect(page.getByTestId('career-save')).toBeVisible()
await expect(page.getByTestId('career-save')).not.toBeVisible()  // After cancel, footer hides
await expect(page.getByTestId('career-save')).toBeEnabled()
await expect(page.getByTestId('coach-settings-level-checkbox-EXPERT')).toBeChecked()
await expect(page.getByTestId('coach-settings-level-checkbox-BEGINNER')).not.toBeChecked()
await expect(page.getByTestId('career-past-start-month')).toHaveValue('3')
await expect(page).toHaveURL(/\/career-path/)

// ❌ Breaks in other languages — never do this
await expect(page.getByTestId('result')).toHaveText('Success')
await expect(page.getByTestId('message')).toContainText('Saved successfully')
```

**Safe assertions in your codebase:**

- `toBeVisible()` / `not.toBeVisible()` — element visible or hidden on screen
- `toBeEnabled()` / `not.toBeEnabled()` — button/input clickable or disabled
- `toBeChecked()` / `not.toBeChecked()` — checkbox checked or unchecked
- `toHaveValue(...)` — input/select value
- `toHaveURL(...)` — current page URL

**Why the negatives matter:**
Test that UI elements disappear after actions, not just that they appear before. For example:

- After cancel → footer should be **not.toBeVisible()**
- After uncheck → checkbox should be **not.toBeChecked()**
- Before form is valid → button should be **not.toBeEnabled()**

**Unsafe assertions (never use):**

- `toHaveText()` — relies on visible text (language-dependent)
- `toContainText()` — substring in visible text (language-dependent)
- `getByText()` / `getByLabel()` / `getByPlaceholder()` — all text-based locators

---

## 12 — Environment & Config

### `.env.test`

| Variable | Purpose |
|---|---|
| `VITE_APP_URL` | Base URL for the staging app |
| `VALID_USERNAME` | Test user email |
| `VALID_PASSWORD` | Test user password |
| `DEV_PLAN_PERSON_ID` | Staging person ID for dev plan tests |

Never hardcode these in test files. Access them through `constants.ts`.

### `playwright.config.ts` — key settings

- `testDir: './playwright'`
- `fullyParallel: true`
- `workers: 2` (local) / `1` (CI)
- `retries: 2` (CI) / `0` (local)
- `trace: 'retain-on-failure-and-retries'`
- `expect.timeout: 15000`

### Projects

| Project | Matches | Auth | Purpose |
|---|---|---|---|
| `setup` | `support/auth.setup.ts` | — | One-time login, saves storageState |
| `chromium` | `e2e/**/*.spec.ts` (excl. login) | `storageState` | All feature tests |
| `chromium-public` | `e2e/login.spec.ts` | — | Login/logout/auth tests |
