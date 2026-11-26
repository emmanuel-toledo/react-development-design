---
name: react-testing
description: Use when writing, moving or reviewing tests in a React app, configuring Vitest or Playwright, or deciding where a test belongs. Covers the tests/ tree mirroring src/, naming, mocking with MSW, accessibility checks and coverage. Load this before adding or changing a test.
---

# React Testing

Owns **how and where tests are written** in a React application. Load this before adding, moving or reviewing a test.

## When to use

- Adding or moving a test under `tests/`.
- Configuring Vitest, Testing Library or Playwright.
- Deciding whether a test is unit, integration or e2e.
- Reviewing a change that touches tests.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared code, read `react-shared`. For the library stack, read `react-stack`.

## Structure

```text
tests/
  app/                      # composition, providers, router
  shared/                   # mirrors src/shared
  features/
    <feature>/              # mirrors src/features/<feature>
  e2e/                     # Playwright end-to-end tests
  factories/                # (or fixtures/) reusable test data
  setup.ts                  # global Vitest setup
```

`tests/` mirrors `src/`. **No `*.test.*` files under `src/`.**

## Naming

- Files: `*.test.ts(x)` (e.g. `UserCard.test.tsx`, `useUsers.test.ts`).
- Describe behavior, not implementation.
- One concern per test; clear `describe`/`it` titles.

## Frameworks (per `react-stack`)

- Unit / integration: Vitest + Testing Library.
- End-to-end: Playwright in `tests/e2e/`.
- Accessibility: `jest-axe` / `axe-core` in component tests.

## What to test (mandatory)

- Components: render + interaction.
- Hooks.
- Utils (pure functions).
- Stores (Zustand).

Cover the states: loading, empty, error, success.

## Queries and assertions

- Use Testing Library queries by role, label or visible text.
- Test behavior, not implementation details.
- Do not rely on class names or internal structure.

## Mocking

- APIs are mocked with MSW (Mock Service Worker) at the network level.
- Fall back to `vi.mock` of services/Axios only when MSW does not apply.
- Mock `@/shared` config in app-level tests when needed.

## Test data

- Reusable factories/fixtures live in `tests/factories/` (or `tests/fixtures/`).
- Mock data factories for MSW live in `mocks/factories/` (root) and are the single source shared by handlers and tests (see `react-data`). `tests/factories/` is reserved for test-only data that is not an API mock (component props, store fixtures). Do not duplicate a mock factory in both places.

## React Query and Zustand in tests

- Wrap components in `QueryClientProvider` with a test client (`retry: false`, no cache).
- Reset Zustand stores between tests.

## Accessibility

- Run `axe` checks on critical components.
- Assert no violations.

## Execution rules

- **All tests must pass (100% success).** No merge with failing tests.
- **Coverage threshold: 90%** (lines, functions, branches, statements). The run fails if coverage drops below 90%.
- Tests of a feature may import its own internals.
- Integration tests across features must use the public API (`@/features/<feature>`).

## Anti-patterns (do not)

- Tests under `src/`.
- Tests that assert implementation details.
- `console.log` in tests.
- Snapshots as the only assertion.
- Skipping or commenting out failing tests to make the suite green.

## Checklist

- [ ] Test lives in `tests/` mirroring `src/`.
- [ ] File named `*.test.ts(x)`.
- [ ] Covers loading, empty, error and success states.
- [ ] Uses MSW for APIs.
- [ ] Accessibility check on critical components.
- [ ] All tests pass (100%).
- [ ] Coverage ≥ 90%.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared folder and components | `react-shared` |
| Library stack and versions | `react-stack` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |