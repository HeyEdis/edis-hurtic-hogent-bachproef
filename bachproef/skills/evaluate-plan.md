---
name: evaluate-plan
description: Evaluate generated Playwright E2E test output for one route against the user story, plan.json, progress log, and E2E testing guidelines. Use after a route's E2E tests have been generated and executed.
---

# Evaluate Plan

Evaluate the generated E2E test output for one route. The goal is not to evaluate the functional quality of the application itself, but to assess whether the AI-supported E2E workflow produced usable, guideline-compliant, executable Playwright test output.

The evaluation answers this central question:

> Did the workflow correctly translate the user story, acceptance criteria, happy flow, and coverage structure into executable Playwright test output that follows the given E2E testing guidelines?

## 1. Evaluation Unit

The evaluation unit is the generated E2E test output for one route.

The Playwright tests and the supporting test files are the primary objects being evaluated. The user story, `plan.json`, and `progress.md` files are reference artefacts used to judge whether the generated output reflects the intended flow, acceptance criteria, coverage structure, and execution status.

Do not evaluate the user story as a standalone writing artefact. Use it as the source of truth for:

- the route under test;
- the actor and role;
- the happy flow;
- the acceptance criteria;
- relevant preconditions;
- expected end state;
- relevant extended scenarios.

Do not evaluate `plan.json` as an isolated planning artefact. Use it as the source of truth for:

- planned tasks;
- task priorities;
- coverage hierarchy;
- task IDs;
- `passes` status;
- acceptance criteria per task.

Use `progress.md` as evidence for:

- what the agent implemented;
- what was verified;
- what failed or was blocked;
- what was corrected or placed out of scope;
- whether execution status is traceable.

## 2. Required Inputs

Read all relevant files before scoring.

Required inputs:

1. User story file: `apps/client/playwright/user-stories/<role>/<route>.md`
2. Plan file: `apps/client/playwright/plans/<route>.json`
3. Progress log: `apps/client/playwright/plans/<route>.progress.md`
4. Generated spec file(s): `apps/client/playwright/e2e/<route>.spec.ts`
5. Matching fixture file(s): `apps/client/playwright/fixtures/*.fixtures.ts`
6. Matching page object file(s): `apps/client/playwright/support/pages/*Page.ts`
7. Route constants: `apps/client/playwright/support/data/routes.ts`
8. E2E guidelines: `apps/client/playwright/skill/e2e-testing-guidelines.md`

Optional but useful inputs:

- helper files in `apps/client/playwright/support/helpers/`;
- shared data files in `apps/client/playwright/support/data/`;
- generated evaluation files from earlier runs in `apps/client/playwright/evaluations/<route>.md`;
- Playwright command output, if available;
- CI output, if available.

If a required input is missing, evaluate what can still be evaluated and explicitly list the missing artefact under "Missing evidence". Missing required evidence should lower the relevant criterion score.

## 3. Scoring Model

Each criterion receives a score from 0 to 2.

| Score | Meaning |
|---:|---|
| 0 | Not satisfied: the criterion is missing or conflicts with the guidelines. |
| 1 | Partially satisfied: the criterion is present but incomplete, inconsistent, or insufficiently evidenced. |
| 2 | Satisfied: the criterion is clearly present and follows the guidelines. |

For every score, write a short finding. For scores `0` or `1`, explicitly state:

- what is missing or incorrect;
- which guideline, functional requirement, non-functional requirement, or coverage rule is affected;
- what should be changed;
- if relevant, why the guideline could not be followed.

For each evaluation dimension, calculate:

```text
dimension_percentage = achieved_points / maximum_points * 100
```

Calculate an overall percentage from all scored criteria unless a different weighting is explicitly requested.

The preliminary quality level is based on the overall percentage:

| Percentage | Quality level | Meaning |
|---:|---|---|
| 0-49% | Needs rework | The test output does not sufficiently meet the guidelines and requires rework. |
| 50-74% | Acceptable | The test output is usable as a basis, but has clear deviations or missing parts. |
| 75-89% | Good | The test output mostly follows the guidelines and has only limited deviations. |
| 90-100% | Excellent | The test output almost fully follows the guidelines and shows strong functional, technical, and traceable quality. |

After calculating the preliminary quality level, apply the minimum criteria in section 6. Minimum criteria can cap the final quality level. If this happens, state the reason clearly.

Example:

> Preliminary quality level based on score: Good. Final quality level after minimum criteria: Acceptable. Reason: the output does not contain one continuous full happy-flow test.

## 4. Evaluation Dimensions

Evaluate the output across four dimensions:

1. Functional coverage
2. Coverage and guideline conformity
3. Executability
4. Traceability of execution and status

Coverage and guideline conformity is the main dimension because it directly evaluates whether the generated tests follow the predefined E2E testing rules.

## 5. Dimension Criteria

### 5.1 Functional Coverage

This dimension checks whether the generated tests validate the intended behaviour from the user story and `plan.json`.

| Criterion | Score guidance |
|---|---|
| Smoke test present | Score 2 if there is one smoke test that opens the route and validates the main page elements. Score 1 if the route is opened but key page elements are not clearly validated. Score 0 if no smoke test exists. |
| Happy-flow steps tested | Score 2 if every happy-flow step from the user story is covered by generated tests. Score 1 if only some steps are covered. Score 0 if the happy flow is mostly absent. If steps are missing, list the missing steps. |
| Full happy-flow test present | Score 2 if one continuous test executes the full primary happy flow. Score 1 if a partial continuous flow exists but does not cover the complete happy flow. Score 0 if no continuous full happy-flow test exists. |
| Acceptance criteria covered | Score 2 if the relevant acceptance criteria from the user story or plan are tested. Score 1 if only some criteria are covered. Score 0 if acceptance criteria are not reflected in the tests. |
| Expected end state checked | Score 2 if the generated tests assert the expected end state from the user story or task description. Score 1 if the end state is only partially or indirectly checked. Score 0 if no end-state assertion exists. |

The expected end state must be derived from the happy flow, acceptance criteria, or task description. Examples include correct navigation, a visible result component, persisted `E2E_` test data, an active filter state, or wizard completion.

### 5.2 Coverage and Guideline Conformity

This dimension checks whether the generated tests follow the coverage hierarchy and E2E testing guidelines.

#### Coverage Hierarchy

| Criterion | Score guidance |
|---|---|
| Smoke test comes first | Score 2 if the smoke test is the first test for the route. Score 1 if a smoke test exists but is not clearly first. Score 0 if no smoke test exists. |
| Incremental happy-flow build-up | Score 2 if the happy flow is built up through incremental tests before the full happy-flow test. Score 1 if incremental tests exist but are incomplete or unordered. Score 0 if the output jumps directly to a full flow or lacks step tests. |
| Full happy-flow test follows step tests | Score 2 if the full happy-flow test comes after the incremental step tests. Score 1 if the full happy-flow test exists but the order is unclear. Score 0 if the full happy-flow test is missing. |
| Extended tests follow happy flow | Score 2 if validation, alternative-state, empty-state, error-path, or edge-case tests appear after the happy-flow tests. Score 1 if extended tests exist but their position or relation to the flow is unclear. Score 0 if relevant extended tests are expected but absent. If no extended scenario exists in the user story or plan, mark the criterion as not applicable and explain why. |

#### E2E Guidelines

| Criterion | Score guidance |
|---|---|
| `data-testid` locators | Score 2 if own components are located with `page.getByTestId()`. Score 1 if most locators follow this rule but a few exceptions are present. Score 0 if the test relies heavily on other locator strategies. |
| No banned selectors | Score 2 if the output avoids `getByText`, `getByLabel`, `getByPlaceholder`, `getByRole` with `name`, CSS selectors, and XPath for own components. Score 1 for limited non-critical violations. Score 0 for repeated or core-flow violations. |
| No text assertions | Score 2 if the output avoids `toHaveText` and `toContainText`. Score 1 for limited non-critical use. Score 0 if text assertions are used for core validation. |
| No arbitrary waits | Score 2 if `waitForTimeout` is absent. Score 0 if any arbitrary wait is present. |
| Routes from `routes.ts` | Score 2 if routes are imported from route constants. Score 1 if most route usage follows this but isolated hardcoded paths remain. Score 0 if routes are mainly hardcoded. |
| Correct authentication | Score 2 if authenticated feature tests rely on `storageState` and do not perform manual login. Score 1 if there is ambiguous or mixed auth handling. Score 0 if feature tests implement manual login incorrectly. |
| Real APIs | Score 2 if the test uses real API behaviour and no internal mocks/intercepts. Score 1 if there are limited justified exceptions. Score 0 if internal mocks are used for E2E behaviour. |
| `E2E_` test data | Score 2 if persistent test-created data uses the `E2E_` prefix. Score 1 if this is inconsistent. Score 0 if persistent test data is created without the prefix. If the test is read-only, score 2 and state that no persistent data is created. |
| Project structure | Score 2 if the output follows the project structure for specs, page objects, fixtures, helpers, and support data. Score 1 if the structure is mostly correct but has limited misplaced logic. Score 0 if the test is mostly implemented as an unstructured spec. |

### 5.3 Executability

This dimension checks whether the generated tests can actually run.

| Criterion | Score guidance |
|---|---|
| Local execution | Score 2 if the test runs locally with Playwright. Score 1 if it can run after minor non-structural fixes. Score 0 if it cannot run. |
| Clear pass/fail output | Score 2 if the test output clearly shows which tests pass or fail. Score 1 if output exists but is incomplete or ambiguous. Score 0 if no useful execution output exists. |
| CI/CD suitability | Score 2 if the test can run in the existing CI/CD pipeline without structural changes. Score 1 if minor configuration work remains. Score 0 if the test is structurally incompatible with CI/CD. |
| No structural execution blocker | Score 2 if no missing route, missing test data, missing auth, or missing configuration fundamentally blocks execution. Score 1 if a blocker exists but is clearly documented and limited. Score 0 if execution is blocked and not sufficiently explained. |

### 5.4 Traceability of Execution and Status

This dimension checks whether it is objectively possible to reconstruct what the agent did, what passed, what was blocked, and which coverage level was reached.

| Criterion | Score guidance |
|---|---|
| `progress.md` exists | Score 2 if a progress file exists for the route. Score 0 if it is missing. |
| Progress per task | Score 2 if executed tasks are clearly described in `progress.md`. Score 1 if progress is present but incomplete or vague. Score 0 if task progress cannot be reconstructed. |
| Changes and verification documented | Score 2 if `progress.md` describes changed files and verification commands. Score 1 if only one of these is clear. Score 0 if neither is clear. |
| Blockers or scope corrections documented | Score 2 if blockers, failed assumptions, or scope corrections are documented when relevant. Score 1 if they are only partially documented. Score 0 if known blockers are absent from the log. If no blocker occurred, score 2 and state that none were found. |
| `passes` matches test output | Score 2 if tasks marked `passes: true` correspond to present and passing test output. Score 1 if the relation is partially clear. Score 0 if `passes` is inconsistent with test output. |
| Coverage level is inferable | Score 2 if smoke, happy-flow, and extended coverage status can be inferred from tests, plan, and progress. Score 1 if only partial coverage status is inferable. Score 0 if coverage status cannot be determined. |

## 6. Minimum Criteria by Quality Level

### 6.1 Acceptable

Generated E2E test output for one route can only be `Acceptable` if all of these minimum criteria are met:

1. The test runs locally with Playwright.
2. The test gives a clear pass/fail result.
3. There is no structural blocker that makes CI execution impossible.
4. One smoke test is present.
5. The smoke test opens the route.
6. The smoke test validates the main page elements.
7. Every happy-flow step from the user story is tested.
8. The happy-flow steps may be spread across separate tests.
9. A single continuous full happy-flow test is not required for `Acceptable`.
10. The test does not validate behaviour outside the user story or `plan.json`.
11. The test mainly uses `data-testid` locators.
12. `plan.json` and `progress.md` show which tasks were executed and which tasks passed or were blocked.

If not every happy-flow step from the user story is tested, the output cannot be `Acceptable`. The evaluation must list the missing happy-flow step or steps.

`Acceptable` may still miss:

- one continuous full happy-flow test;
- extended coverage;
- full locator conformity;
- perfect project structure;
- full reusability.

### 6.2 Good

Generated E2E test output for one route can only be `Good` if all of these minimum criteria are met:

1. The output satisfies all `Acceptable` minimum criteria.
2. One continuous full happy-flow test is present.
3. That test executes the complete primary user flow from the user story.
4. That test checks the expected end state from the user story or `plan.json`.
5. The test order follows: smoke test, incremental happy-flow steps, full happy-flow test.
6. Extended tests, if present, follow the happy-flow test.
7. The test mostly follows locator rules.
8. There are no severe guideline violations, such as text selectors for core interactions, CSS or XPath selectors for own components, manual login in feature tests, or internal mocks.
9. Routes, test data, and authentication mostly follow the E2E guidelines.
10. `passes` status, test output, and `progress.md` are consistent with each other.
11. Important blockers or scope corrections are documented.

`Good` may still miss:

- extended coverage;
- full locator conformity;
- full reusability;
- perfect page object or fixture structure;
- all possible edge cases.

If the full happy-flow test is missing, the output can be at most `Acceptable`.

### 6.3 Excellent

Generated E2E test output for one route can only be `Excellent` if all of these minimum criteria are met:

1. The output satisfies all `Good` minimum criteria.
2. At least one relevant extended test is present when the user story or plan contains validation, alternative-state, empty-state, error-path, or edge-case scenarios.
3. The extended test clearly maps to an acceptance criterion, validation rule, or edge case from the user story or plan.
4. If the route has no relevant extended scenario, the evaluation explicitly explains this.
5. The test uses `data-testid` locators for all own components.
6. The test uses no banned selectors or text assertions.
7. The test uses no arbitrary waits.
8. Routes come from `routes.ts`.
9. Authenticated feature tests use `storageState`.
10. Persistent test data uses the `E2E_` prefix.
11. The test uses no internal mocks for E2E behaviour.
12. Interaction logic is placed in page objects.
13. Test data and setup are placed in fixtures or support files.
14. Reusable steps are not unnecessarily duplicated in specs.
15. `progress.md` describes executed tasks, verification, and relevant decisions.
16. `passes: true` corresponds to passing test execution.
17. The reached coverage level is clearly inferable.

`Excellent` may only contain small non-blocking improvement points, such as slightly inconsistent naming, additional possible reuse, or extra edge cases outside the user story or plan.

If extended coverage is missing while relevant extended scenarios exist in the user story or plan, the output can be at most `Good`.

## 7. Final Quality Level

Determine the final quality level in two steps:

1. Calculate the preliminary quality level from the overall score percentage.
2. Apply the minimum criteria for that quality level.

If minimum criteria are missing, lower the quality level to the highest level whose minimum criteria are satisfied.

The evaluation must explicitly state:

- preliminary quality level;
- final quality level;
- whether minimum criteria capped the level;
- which missing criteria caused the cap.

Example:

> Preliminary quality level based on score: Good. Final quality level after minimum criteria: Acceptable. Reason: the output does not contain one continuous full happy-flow test.

## 8. Required Output Format

Create an evaluation file at `apps/client/playwright/evaluations/<route>.md`. Use this structure:

```markdown
# Evaluation: <route>

## Artefacts

- User story: `apps/client/playwright/user-stories/<role>/<route>.md`
- Plan file: `apps/client/playwright/plans/<route>.json`
- Progress file: `apps/client/playwright/plans/<route>.progress.md`
- Test files: `apps/client/playwright/e2e/<route>.spec.ts`
- Fixture files: `apps/client/playwright/fixtures/<route>.fixtures.ts`
- Page object files: `apps/client/playwright/support/pages/<route>Page.ts`
- E2E guidelines: `apps/client/playwright/skill/e2e-testing-guidelines.md`

## Missing Evidence

- <List missing artefacts, or write "None">

## Dimension Scores

| Dimension | Score | Percentage |
|---|---:|---:|
| Functional coverage | X/Y | Z% |
| Coverage and guideline conformity | X/Y | Z% |
| Executability | X/Y | Z% |
| Traceability of execution and status | X/Y | Z% |

## Detailed Findings

### 1. Functional Coverage

| Criterion | Score | Finding |
|---|---:|---|
| Smoke test present | 2 | ... |
| Happy-flow steps tested | 1 | ... |
| Full happy-flow test present | 0 | ... |
| Acceptance criteria covered | 1 | ... |
| Expected end state checked | 1 | ... |

Missing or weak points:
- <Concrete issue>

Required improvements:
- <Concrete improvement>

### 2. Coverage and Guideline Conformity

| Criterion | Score | Finding |
|---|---:|---|
| Smoke test comes first | 2 | ... |
| Incremental happy-flow build-up | 1 | ... |
| Full happy-flow test follows step tests | 0 | ... |
| Extended tests follow happy flow | 1 | ... |
| `data-testid` locators | 2 | ... |
| No banned selectors | 2 | ... |
| No text assertions | 2 | ... |
| No arbitrary waits | 2 | ... |
| Routes from `routes.ts` | 1 | ... |
| Correct authentication | 2 | ... |
| Real APIs | 2 | ... |
| `E2E_` test data | 2 | ... |
| Project structure | 1 | ... |

Missing or weak points:
- <Concrete issue>

Required improvements:
- <Concrete improvement>

### 3. Executability

| Criterion | Score | Finding |
|---|---:|---|
| Local execution | 2 | ... |
| Clear pass/fail output | 2 | ... |
| CI/CD suitability | 1 | ... |
| No structural execution blocker | 2 | ... |

Missing or weak points:
- <Concrete issue>

Required improvements:
- <Concrete improvement>

### 4. Traceability of Execution and Status

| Criterion | Score | Finding |
|---|---:|---|
| `progress.md` exists | 2 | ... |
| Progress per task | 2 | ... |
| Changes and verification documented | 1 | ... |
| Blockers or scope corrections documented | 2 | ... |
| `passes` matches test output | 2 | ... |
| Coverage level is inferable | 1 | ... |

Missing or weak points:
- <Concrete issue>

Required improvements:
- <Concrete improvement>

## Quality Level

Preliminary quality level based on score: <Needs rework | Acceptable | Good | Excellent>

Final quality level after minimum criteria: <Needs rework | Acceptable | Good | Excellent>

Minimum criteria cap:
- <None, or explain which missing criterion capped the level>

## Improvement Plan

- <Concrete improvement needed to reach the next quality level>
- <Concrete improvement needed to reach the next quality level>

## Conclusion

<Short paragraph explaining whether the generated E2E test output is usable, which quality level it reached, and what must be improved next.>
```

## 9. Evaluation Rules

- Be strict but fair.
- Do not award points for behaviour that is only implied but not visible in the test output.
- Use the user story and `plan.json` as references for the intended flow.
- Use the E2E guidelines as the source of truth for technical conventions.
- If a criterion is not applicable, explain why and exclude it from the maximum score for that dimension.
- Never hide missing evidence. If an artefact is absent, state it clearly.
- When a happy-flow step is missing, quote or paraphrase the missing step from the user story.
- When a score is `0` or `1`, provide a concrete improvement.
- If a minimum criterion caps the quality level, state this explicitly.
