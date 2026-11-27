---
name: react-conventions
description: Use when committing, branching, opening a PR, or reviewing code in a React app. Covers Conventional Commits, Git Flow, branch naming, PR template, merge strategy and a unified code review checklist. Load this before committing or opening a PR.
---

# React Conventions

Owns **team workflow conventions** in a React application: commits, branching, pull requests, merge strategy and code review. Load this before committing or opening a PR.

## When to use

- Writing a commit message or creating a branch.
- Opening or reviewing a pull request.
- Deciding merge strategy or review requirements.
- Setting up commit/PR tooling.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared code, read `react-shared`. For the library stack, read `react-stack`. For testing, read `react-testing`. For effects, read `react-effects`. For forms, read `react-forms`. For routing, read `react-routing`. For data, read `react-data`. For accessibility, read `react-a11y`. For performance, read `react-performance`.

## Commits (Conventional Commits)

Format: `<type>(<scope>): <subject>`

- `type`: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `style`, `ci`, `build`, `revert`.
- `scope` (optional): affected area, e.g. `feat(auth): add login form`.
- `subject`: imperative mood, lowercase, no trailing period, ≤ 72 characters.
- Body (optional): explain the *why*, not the *what*.
- Footer (optional): `BREAKING CHANGE: ...` or `Closes #123`.

### Atomic commits

- One logical change per commit.
- Do not mix refactors, features and fixes in the same commit.
- Each commit should leave the app in a working state.

### Commit template

```text
<type>(<scope>): <subject>

<body>

<footer>
```

Example:

```text
feat(users): add user detail page

Adds a detail view reachable from the users list, with loading and
error states.

Closes #42
```

## Branching (Git Flow)

- Main branches: `main` (production) and `develop` (integration).
- Working branches:
  - `feature/*` (or `feat/*`) from `develop`.
  - `release/*` from `develop` toward `main`.
  - `hotfix/*` from `main`.
- Never commit directly to `main` or `develop`.
- Delete branches after merge.

> The branching strategy is a project decision (like routing guards). Git Flow is the recommended default; adapt if the project uses GitHub Flow or trunk-based.

## Branch naming

Format: `type/short-description` in kebab-case.

- Types aligned with commits: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`.
- Short, descriptive, no spaces or special characters.
- Examples: `feat/user-auth`, `fix/login-bug`, `chore/update-deps`.

## Pull Requests

- Template at `.github/PULL_REQUEST_TEMPLATE.md` (see template below).
- Title follows Conventional Commits.
- One PR per logical change; keep PRs small and focused.
- Link the issue: `Closes #123`.
- At least **1 approval** required before merge.
- CI must pass: lint, typecheck, tests, coverage ≥ 90%.

### PR template

```markdown
## Description

<!-- What does this PR do and why? -->

## Changes

- ...

## How to test

<!-- Steps to verify the change -->

## Screenshots (if UI)

<!-- Before/after -->

## Checklist

- [ ] Lint and typecheck pass
- [ ] Tests added/updated; all pass (100%)
- [ ] Coverage ≥ 90%
- [ ] No `console.log` or dead code
- [ ] No secrets or sensitive data
- [ ] Feature structure respected (no empty folders, no `providers/`/`schemas/`)
- [ ] API access only in `apis/`; components consume hooks
- [ ] Public API (`index.ts`) minimal; no deep imports
- [ ] Effects only approved and documented (`docs/decisions/effects.md` + `// effect: EFF-XXX`)
- [ ] Forms reuse shared components; schema in `utils/`; inferred type; unsaved-changes guard
- [ ] Routing: kebab-case plural paths, camelCase+Id params, guards, 404 in shared, lazy pages
- [ ] Data: clients in `shared/apis/<client>/`; React Query invalidation; mocks updated
- [ ] A11y: semantics, labels, keyboard, contrast, skip link, axe without violations
- [ ] Performance: no premature memoization; lists > 100 virtualized; images optimized
- [ ] Shared: promotion approved; shadcn in `shared/components/shadcn/ui/`
- [ ] No unrelated changes in this PR
```

## Versioning and releases

- Use Semantic Versioning (semver): `MAJOR.MINOR.PATCH`.
- `MAJOR`: breaking changes; `MINOR`: new features (backward compatible); `PATCH`: bug fixes.
- Tag releases on `main` (e.g. `v1.2.3`).
- `CODEOWNERS` file (optional) to auto-assign reviewers per path.
- Release process: create `release/*` from `develop`, bump version, merge to `main` and back-merge to `develop`, tag.

## Merge strategy

- **Squash and merge** by default (one commit per PR).
- Edit the squash message to follow Conventional Commits.
- No direct pushes to protected branches.

## Code review checklist

Use the checklist in the PR template above. Reviewers verify all items; authors self-check before requesting review.

## Anti-patterns (do not)

- Giant commits or commits mixing unrelated changes.
- PRs without description or tests.
- Merging without review.
- Force-pushing to protected branches.
- Secrets in commits.
- Long-lived branches not synced with `develop`.

## Tooling (recommended, requires approval)

- `commitlint` + `husky` to validate commit messages.
- `lint-staged` to lint on pre-commit.

New dev dependencies require justification and approval per `react-architecture`.

## Checklist

- [ ] Commits follow Conventional Commits.
- [ ] Branches follow Git Flow and naming convention.
- [ ] PRs use the template and require approval.
- [ ] Merge strategy: squash and merge.
- [ ] Code review checklist complete.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared folder and promotion | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |