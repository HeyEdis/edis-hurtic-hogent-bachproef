---
name: introspect
description: End-of-session self-improvement — reviews friction (retries, rework, backtracking), audits skills and guidance files, and applies targeted improvements. Use at the end of a coding session, after completing a Ralph loop, or when prompted to reflect and improve tooling.
---

# Introspect

End-of-session self-improvement. Run this while you have rich context about what was just implemented and how it went. The goal is to make the _next_ session better by capturing what this session taught you.

## Step 1: Friction Log

Scan the session for signals that something could be improved:

- **Retried commands** — wrong flags, missing deps, incorrect paths, typos
- **Reworked code** — changes that were reverted or significantly rewritten
- **Test struggles** — specs that took multiple attempts to pass
- **Pattern mismatches** — code written one way then corrected to match conventions
- **Backtracking** — decisions that had to be unwound
- **Missing knowledge** — things you had to discover that should have been documented

Compile a brief friction log (bullet list). If the session was smooth, say so and skip to Step 4 for the documentation pass.

## Step 2: Skill Audit

Identify which skills were used (or _should_ have been used). Check these locations:

- Project-specific skills: `.cursor/skills/*/SKILL.md` or `.cursor/rules/*.md` — project-level skills/rules
- Global skills: `~/.agents/skills/*/SKILL.md` — global agent skills (global claude skills (`~/.claude/skills/*/SKILL.md`) are symlinked to this directory)

For each relevant skill, ask:

1. Was the guidance sufficient, or did I deviate from it?
2. Is there missing guidance that _caused_ friction?
3. Would an addition here prevent this class of mistake across projects?

## Step 3: Prompt Audit

Review every prompt file that was used or referenced during this session — skill prompts, iteration prompts, system prompts, command prompts, or any `.md` file that instructed the model.

For each prompt, ask:

1. Did the prompt's instructions lead to correct behaviour, or did I have to deviate?
2. Did ambiguous or missing instructions _cause_ friction logged in Step 1?
3. Could clearer wording, additional examples, or reordered steps prevent the same friction next time?

## Step 4: Apply Improvements

Work through each layer. The key rule: **scope determines destination**.

### 4a. Global skills (`~/.agents/skills/` symlinked to `~/.claude/skills/`)

Update ONLY if the improvement applies **across codebases**, not just the current one.

Good candidates:

- A workflow skill missing a step that caused retries
- Ambiguous instructions that led to wrong interpretation
- A common tool flag or pattern that should be documented

Bad candidates (don't add):

- Project-specific conventions (Angular prefix, Tailwind config)
- Framework-specific patterns only this repo uses

### 4b. CLAUDE.md / AGENTS.md (repo root)

Update ONLY with **cross-cutting, non-feature-specific** guidance that applies repo-wide.

Good candidates:

- Missing coding conventions that apply across all features
- Command patterns that are commonly needed but weren't documented
- Architecture rules that apply repo-wide
- Inject function preferences, testing patterns, import conventions

Bad candidates (don't add):

- Feature-specific decisions or implementation details
- Temporary workarounds for a single feature

### 4c. Feature-specific docs (`docs/`)

Everything specific to the current feature goes here:

- Architectural decisions made during implementation
- Patterns established for this feature
- Gotchas and edge cases discovered
- Integration notes and API quirks

Save as `docs/<feature-slug>-learnings.md` or append to an existing feature doc if one exists.

### 4d. Prompt improvements

Apply improvements identified in Step 3 to the prompt files. Follow the same scoping rules:

- **Global prompts** (in `~/.agents/`) — update only if the improvement applies across codebases
- **Repo prompts** — update if the improvement is specific to this repository

Focus on:

- Clarifying ambiguous instructions that caused wrong interpretation
- Adding missing steps or constraints that would have prevented friction
- Improving examples or adding edge case guidance
- Removing instructions that actively misled or conflicted with reality

### 4e. Ralph prompt sync

If you updated any Ralph iteration prompts (`prompts/ralph-iteration*.md`), ensure all three variants stay consistent where they share the same sections:

- `prompts/ralph-iteration.md` (base)
- `prompts/ralph-iteration-afk.md` (autonomous)
- `prompts/ralph-iteration-hitl.md` (human-in-the-loop)

Only the mode-specific sections (autonomous decision making, HITL review phase, introspect phase) should differ.

## Step 5: Summary

Output a concise summary of what was improved and why:

```
## Introspect Results

### Friction log
- <what went wrong and why>

### Changes made
- `path/to/file` — <what was added/changed and why>

### Skipped (feature-specific, already documented)
- <items captured in docs/ instead>
```

## Rules

- **Surgical edits only** — add or modify what's needed, never rewrite whole files
- **Be specific and actionable** — not "write better code" but "use `injectX()` instead of `inject(XService)`"
- **Include a why** — every addition must explain its purpose
- **No duplicates** — read existing content before adding; update rather than duplicate
- **Earn your place** — every rule added must prevent a _recurring class_ of mistakes, not a one-off
- **Keep improvements feature-agnostic** — global skills, prompts, CLAUDE.md, and memory entries must never reference specific features, entities, routes, or domain concepts. Extract the general principle. If an example is needed, use a generic placeholder.
- **Never add secrets or credentials** to any guidance file