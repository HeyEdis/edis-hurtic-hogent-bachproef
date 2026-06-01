---
name: generate-user-stories
description: Generate user story files for every route in the app. Reads RouteConfig.tsx, groups related routes, creates structured user story markdown files in playwright/user-stories/. Skips files that already exist. Use when you want to generate user stories for e2e test planning.
---

# Generate User Stories from Routes

This skill reads every route in the application and generates user story files that serve as input for a future e2e test-generation loop. Do NOT interview the user — autonomously explore the codebase and generate.

---

## 1 — Source of Truth

- **Routes:** `apps/client/src/RouteConfig.tsx` — the single source of truth for all routes.
- **E2E guidelines:** `apps/client/playwright/skill/SKILL.md` — reference for auth fixtures and test conventions (used lightly, only in Preconditions).
- **Route constants:** `apps/client/playwright/support/data/routes.ts` — existing route constants for tests.
- **Output directory:** `apps/client/playwright/user-stories/`

---

## 2 — Folder Structure

Organise user story files by role:

```markdown
playwright/user-stories/
├── public/
│   ├── signin.md
│   ├── signup.md
│   └── ...
├── employee/
│   ├── me.md
│   ├── me-activities.md
│   └── ...
└── admin/
    ├── people.md
    ├── people-detail.md          ← parent
    ├── people-detail-jobs.md     ← child of people-detail
    ├── jobs.md
    ├── jobs-detail.md            ← parent
    ├── jobs-detail-people.md     ← child of jobs-detail
    └── ...
```

### Naming rules

- Role folders: `public/`, `employee/`, `admin/`
- File names are derived from the route path with `/` replaced by `-`, stripping the role prefix.
- Dynamic segments (`:id`, `:personId`) are replaced with `detail` for parent routes or dropped for child routes.
- Examples:
  - `/signin` → `public/signin.md`
  - `/employee/me` → `employee/me.md`
  - `/employee/me/activities` → `employee/me-activities.md`
  - `/hr/jobs/:id` → `admin/jobs-detail.md`
  - `/hr/jobs/:id/people` → `admin/jobs-detail-people.md`
  - `/hr/people/:id` → `admin/people-detail.md`
  - `/hr/settings/branding` → `admin/settings-branding.md`

---

## 3 — Route Grouping Rules

### 3.1 Parent/child pattern (default)

When a route has sub-routes under a dynamic segment (e.g. `/hr/jobs/:id/people`, `/hr/jobs/:id/history`):

- The **parent** route (e.g. `/hr/jobs/:id`) gets its own file with shared context and preconditions.
- Each **child** route gets its own file with a `parent` field in frontmatter pointing to the parent file.
- Each child file has its own focused acceptance criteria, happy flow, and key interactions.

### 3.2 Wizard exception

Sequential multi-step flows (e.g. `/hr/development-plans/create/:personId/role-selection`, `step-2`, `step-3`, `step-4`) are treated as a **single file**:

- All step routes listed in frontmatter.
- Acceptance criteria organised as sub-sections per step.
- One continuous happy flow covering the full sequence.
- File name: `admin/development-plan-wizard.md`

### 3.3 Identifying wizards vs. tabs

- If child routes are **sequential steps** of a single flow (user must go through them in order) → wizard pattern → single file.
- If child routes are **independent views/tabs** of a resource (user can visit any in any order) → parent/child pattern → separate files.

---

so that [benefit].

## Acceptance Criteria
- [ ] Given [context], when [action], then [result].

## Happy Flow
1. User navigates to ...
2. User sees ...

## Key Interactions
- List of interactive elements on THIS page (not children).

## Preconditions
- Auth: `adminAuth` fixture
- Seed data: "at least one job profile must exist"
- Note: child route files inherit these preconditions.
```

### 4.3 Child route (has parent)

```markdown
---## 4 — File Templates

### 4.1 Standard route (no parent, no children)

```markdown
---
route: /signin
role: public
page: LoginPage
---

# Sign In

## Description
Brief description of what this page does and why it exists.

## User Stories
1. As a [actor], I want to [action], so that [benefit].

## Acceptance Criteria
- [ ] Given [context], when [action], then [result].

## Happy Flow
1. User navigates to ...
2. User sees ...
3. User does ...
4. System responds with ...

## Key Interactions
- List of interactive elements (buttons, forms, modals, navigation links, etc.)

## Preconditions
- Auth fixture required (e.g. `none — public route`, `adminAuth`, `employeeAuth`)
- Seed data needed (e.g. "existing user account with valid credentials")
```

### 4.2 Parent route (has children)

```markdown
---
route: /hr/jobs/:id
role: admin
page: JobProfilePage
children:
  - admin/jobs-detail-people.md
  - admin/jobs-detail-history.md
  - admin/jobs-detail-activities.md
  - admin/jobs-detail-compare.md
  - admin/jobs-detail-sources.md
---

# Job Detail

## Description
Brief description of the job detail page and its role as a hub for sub-views.

## User Stories
1. As an [actor], I want to [action],
route: /hr/jobs/:id/people
role: admin
page: JobMatchesPage
parent: admin/jobs-detail.md
---

# Job Detail — People Matches

## Description
Brief description of this specific sub-view.

## User Stories
1. As an [actor], I want to [action], so that [benefit].

## Acceptance Criteria
- [ ] Given [context], when [action], then [result].

## Happy Flow
1. User is on the job detail page (see parent).
2. User navigates to the people tab ...
3. User sees ...

## Key Interactions
- List of interactive elements specific to this sub-view.

## Preconditions
- Inherits from parent: admin/jobs-detail.md
- Additional: "job must have at least one person match"
```

### 4.4 Wizard flow (single file, multiple steps)

```markdown
---
routes:
  - /hr/development-plans/create/:personId/role-selection
  - /hr/development-plans/create/:personId/step-2
  - /hr/development-plans/create/:personId/step-3
  - /hr/development-plans/create/:personId/step-4
role: admin
pages:
  - WizardStep1
  - WizardStep2
  - WizardStep3
  - WizardStep4
type: wizard
---

# Development Plan Wizard

## Description
Brief description of the entire wizard flow.

## User Stories
1. As an [actor], I want to [action], so that [benefit].

## Acceptance Criteria

### Step 1 — Role Selection
- [ ] Given [context], when [action], then [result].

### Step 2 — ...
- [ ] Given [context], when [action], then [result].

### Step 3 — ...
- [ ] Given [context], when [action], then [result].

### Step 4 — ...
- [ ] Given [context], when [action], then [result].

## Happy Flow
1. HR admin navigates to create development plan for a person.
2. Step 1: ...
3. Step 2: ...
4. Step 3: ...
5. Step 4: ...
6. Wizard completes and redirects to ...

## Key Interactions
- Step 1: ...
- Step 2: ...
- Step 3: ...
- Step 4: ...

## Preconditions
- Auth: `adminAuth` fixture
- Seed data: "person must exist with a profile"
```

---

## 5 — Exploration Depth

For each route, **deeply explore the page component tree**:

1. Read the top-level page component imported in `RouteConfig.tsx`.
2. Follow its child components — read each imported component.
3. Identify all interactive elements: buttons, forms, modals, dropdowns, links, tabs, toggles.
4. Identify data dependencies: what GraphQL queries/mutations are used, what data must exist.
5. Identify conditional UI: what changes based on state, permissions, or data presence.

Use this understanding to write **specific, meaningful** user stories and acceptance criteria — not generic ones.

### What makes a good user story

**Good:** "As an HR admin, I want to see a ranked list of people matching a job's required skills, so that I can identify the best internal candidates."

**Bad:** "As a user, I want to view the people page, so that I can see people."

### What makes good acceptance criteria

**Good:** "Given a job with 3 matching people, when the HR admin navigates to the people tab, then all 3 matches are displayed with their match percentage."

**Bad:** "Given a user, when they visit the page, then it loads."

---

## 6 — Skip Logic

Before generating a file, check if it already exists in `playwright/user-stories/`:

- If the file exists → **skip it entirely**. Do not overwrite, do not merge.
- If the file does not exist → generate it.

---

## 7 — Scoping

By default, the skill processes **all routes** from `RouteConfig.tsx`.

If the user provides context narrowing the scope:

- A specific role (e.g. "generate for admin routes") → only process routes in that role group.
- A specific route (e.g. "generate for /hr/jobs/:id") → only generate that file (and its children if it's a parent).

---

## 8 — Process

1. **Read `RouteConfig.tsx`** — extract all routes, their page components, and role groupings.
2. **Build the route map** — determine which routes are parents, children, wizards, or standalone.
3. **Check existing files** — scan `playwright/user-stories/` for already-generated files.
4. **For each route group that needs generation:**
   a. Read the page component(s) deeply — follow the component tree.
   b. Identify user-visible features, interactions, data dependencies, and conditional UI.
   c. Reference `SKILL.md` lightly for auth fixture names and seed data conventions.
   d. Write the user story file using the appropriate template (standard / parent / child / wizard).
   e. Save to `playwright/user-stories/<role>/<filename>.md`.
5. **Print the generation summary.**

---

## 9 — Generation Summary

After processing all routes, output a summary:

```markdown
## Generation Summary

### Generated (X files)
- ✅ public/signin.md
- ✅ admin/jobs-detail.md (parent)
- ✅ admin/jobs-detail-people.md (child of jobs-detail)
- ...

### Skipped (Y files already exist)
- ⏭️ employee/me.md
- ...

### Route Map
Total routes: N | Grouped into: M files | Generated: X | Skipped: Y
```
