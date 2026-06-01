---
name: user-story-to-plan-json
description: Turn a user story into a plan.json file for the Ralph autonomous e2e testing loop. Breaks the user story into prioritized test tasks with acceptance criteria in JSON format. Output goes in playwright/plans/.
allowed_tools: Read, Write, Edit, Glob, Grep, Bash
---

# User Story to plan.json

Convert a user story into a `plan.json` file suitable for the Ralph autonomous e2e testing loop. Each task in the plan maps to a single `test()` block that Ralph will implement.

## Process

### 1. Confirm the user story is in context

The user story should already be in the conversation. If it isn't, ask the user to paste it or point you to the file. Check `apps/client/playwright/user-stories/<role>/*.md` for existing user stories. User stories follow the format defined by the `generate-user-stories` skill — they contain route, role, page metadata, acceptance criteria, happy flows, and key interactions.

### 2. Explore the codebase

If you have not already explored the codebase, do so to understand:

- The page components referenced in the user story's `page` frontmatter field
- Existing POM pages in `apps/client/playwright/support/pages/`
- Existing fixtures in `apps/client/playwright/fixtures/`
- Existing route constants in `apps/client/playwright/support/data/routes.ts`
- Existing `data-testid` attributes on the components under test
- Existing spec files in `apps/client/playwright/e2e/` to avoid duplication

This informs how to break down the work and what infrastructure already exists.

### 3. Draft tasks

Break the user story into concrete e2e test tasks. Each task maps to a single `test()` block — one observable behaviour per task.

<task-rules>
- Each task describes ONE test that asserts ONE observable user behaviour
- A completed task means a single `test()` block passes
- Prefer many small tests over few large ones
- Later tasks can depend on earlier ones but should be as independent as possible

**Task hierarchy — every plan MUST follow this order:**

1. **Smoke test (always priority 1):** Navigate to the route and assert the page renders without errors. Verify key layout elements are visible via `data-testid` (main sections, tabs, containers). For parent routes with children, each child route also gets its own smoke test. The smoke test proves the route is reachable, authenticated correctly, and the page shell renders.
2. **Incremental step tests (priority 2–N):** Break the user story's happy flow into individual steps. Each step gets its own test that covers one interaction or state transition from the happy flow. Order them sequentially — step 2 tests what happens after step 1's action, step 3 after step 2, etc. These build up confidence in each piece of the flow independently.
3. **Full happy path test (high priority, after step tests):** One end-to-end test that executes the entire happy flow from the user story — from initial navigation through to the final outcome. This test walks through all the steps in sequence and asserts the final result. Redundancy with step tests is intentional — this proves the full flow works as an integrated sequence.
4. **Edge cases and validation tests (after happy path):** Tests for error states, boundary conditions, empty states, and validation rules from the user story's acceptance criteria that are NOT part of the happy flow.

- Acceptance criteria must be specific and testable — tied to what the user sees, not implementation details
- Each task description MUST embed these e2e conventions so Ralph follows them:
  - Use `data-testid` locators only — never locate by visible text (this is a multilingual app with lingui/i18n)
  - Use `page.getByTestId()` exclusively — banned: `getByText`, `getByRole` with name, `getByLabel`, `getByPlaceholder`, CSS selectors, XPath
  - Assert on visibility, enabled state, checked state, and input values — never assert on text content
  - Safe assertions: `toBeVisible()`, `not.toBeVisible()`, `toBeEnabled()`, `toBeChecked()`, `toHaveValue()`, `toHaveURL()`
  - Unsafe assertions (never use): `toHaveText()`, `toContainText()`
  - Routes come from `apps/client/playwright/support/data/routes.ts` — if a route is missing, flag it; do not invent routes
  - Test data persisted to the database must be prefixed with `E2E_`
  - Use `date-fns` for any date manipulation — never raw `Date` arithmetic
  - Tests hit the real staging API — no mocks except for third-party external services
  - Each test must be independently runnable — no shared state, no execution-order dependency
  - POM pages go in `support/pages/`, fixtures in `fixtures/`, helpers in `support/helpers/`
  - Add `data-testid` attributes to React components if they are missing — include this in the task description
  - Sole locator exception: `page.getByRole('dialog')` or `page.getByRole('alert')` for third-party components you cannot modify
</task-rules>

### 4. Quiz the user

Present the proposed task breakdown as a numbered list. For each task show:

- **Title**: short descriptive name
- **What it covers**: brief description of the single behaviour being tested
- **Priority**: suggested order

Ask the user:

- Does the granularity feel right?
- Should any tasks be merged or split?
- Is the priority order correct?

Iterate until the user approves.

### 5. Write plan.json

Write the plan to `apps/client/playwright/plans/<user-story-filename>.json`.

For example, if the user story is `playwright/user-stories/admin/jobs-detail.md`, write to `playwright/plans/jobs-detail.json`. Create the `plans/` directory if it doesn't exist.

Use this structure:

```json
{
  "source": {
    "user_story": "playwright/user-stories/admin/jobs-detail.md",
    "route": "/hr/jobs/:id",
    "role": "admin",
    "page": "JobProfilePageWithState"
  },
  "tasks": [
    {
      "id": "short-kebab-case-id",
      "title": "Short descriptive title",
      "description": "What single test() block to write. Include: the behaviour being tested, which page object methods to use or create, which data-testid attributes to target (or add if missing), which fixture data is needed. Be specific enough that an AI agent can implement this without further clarification. Embed the e2e conventions: data-testid locators only, no text assertions, routes from routes.ts, E2E_ prefix for test data.",
      "acceptance_criteria": [
        "Specific, testable criterion — e.g. 'test passes: page.getByTestId(\"feature-section\").toBeVisible()'",
        "Specific, testable criterion — e.g. 'data-testid=\"feature-save\" exists on the save button component'"
      ],
      "priority": 1,
      "passes": false
    }
  ]
}
```

Rules for the JSON:

- `source.user_story`: relative path to the user story file from the workspace root
- `source.route`, `source.role`, `source.page`: copied from the user story frontmatter
- `id`: unique kebab-case identifier
- `priority`: integer, 1 is highest priority (do first)
- `passes`: always `false` — Ralph marks these `true` as it completes them
- `description`: detailed enough for an autonomous agent to write a single `test()` block without ambiguity. Must include which `data-testid` attributes to use and the e2e conventions to follow.
- `acceptance_criteria`: concrete and verifiable — tied to assertions (`toBeVisible`, `toBeChecked`, `toHaveValue`, `toHaveURL`), not vague ("works correctly")
- For tasks that verify data-fetching UI controls (dropdowns, selects, lists, tables), acceptance criteria MUST include end-to-end data verification — e.g., "open the dropdown and verify it shows options fetched from the API via `toBeVisible()` on `getByTestId('feature-option-<id>')`". It is not sufficient to assert that the control renders — the data flow must be proven to work.
- Tasks MUST be ordered following the task hierarchy: smoke test → incremental steps → full happy path → edge cases/validation
- **NEVER edit existing tasks** — when a `plan.json` already exists, only append new tasks. Never modify tasks that already exist — not their `passes` field, not their description, not their acceptance criteria. This includes tasks marked `"passes": true` that turn out to be incomplete or broken: do NOT flip them back to `false`. Instead, create a new task that addresses the gap. Set the `priority` of new tasks to continue from the highest existing priority number.
