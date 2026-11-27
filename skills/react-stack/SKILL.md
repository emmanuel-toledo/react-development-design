---
name: react-stack
description: Use when choosing, adding or reviewing dependencies in a React app. Defines the standard library stack, version policy (most stable) and UI component preference (shadcn/ui first, Radix for custom). Load this before adding a dependency.
---

# React Stack

Owns **the standard library stack** for React applications. Load this before adding, changing or reviewing a dependency.

## Version policy

- Always use the **most stable** version of each library. No pinned/locked versions are mandated by this skill; prefer the latest stable release.
- Do not adopt alpha/beta/RC unless explicitly approved.

## Core

| Concern | Library |
| --- | --- |
| UI framework | React |
| Language | TypeScript |
| Build / dev server | Vite |
| Styling | Tailwind CSS |

## Application

| Concern | Library |
| --- | --- |
| Routing | React Router |
| Type-safe search params | nuqs |
| Global state | Zustand |
| Server state / data fetching | TanStack Query (React Query) |
| HTTP client | Axios |
| Forms | react-hook-form |
| Form resolver (Zod) | @hookform/resolvers |
| Validation / schemas | Zod |
| Internationalization | react-i18next / i18next |
| Icons | Lucide React |
| Toasts / notifications | Sonner |

## UI components

- Prefer **shadcn/ui** first. Its components live in `src/shared/components/shadcn/ui/` (see `react-shared`).
- Fall back to **Radix UI** only when shadcn does not cover the need (custom/complex components).
- Do not introduce other component libraries (MUI, Chakra, Ant Design, etc.) without approval.

## Testing

| Concern | Library |
| --- | --- |
| Unit / integration | Vitest + Testing Library |
| API mocking in tests | MSW (Mock Service Worker) |
| Accessibility in tests | axe-core / jest-axe |
| End-to-end | Playwright |

## Dev tooling

| Concern | Library |
| --- | --- |
| Linting | ESLint |
| A11y linting (dev, recommended) | eslint-plugin-jsx-a11y |
| List virtualization (recommended, requires approval) | @tanstack/react-virtual |
| Formatting | Prettier |
| Types | TypeScript |

## Optional dependencies (require approval)

These are mentioned by other skills but are not part of the base stack. Add only with justification and approval (see `react-architecture`).

| Concern | Library | Used by |
| --- | --- | --- |
| Test data factories | @faker-js/faker | `react-data` |
| Re-render debugging (dev) | why-did-you-render | `react-performance` |
| Bundle analysis (dev) | rollup-plugin-visualizer / vite-bundle-visualizer | `react-performance` |
| Commit message linting (dev) | commitlint + husky | `react-conventions` |
| Pre-commit linting (dev) | lint-staged | `react-conventions` |
| Image optimization (build) | vite-imagetools | `react-performance` |
| List virtualization | @tanstack/react-virtual | `react-performance` |
| A11y linting (dev) | eslint-plugin-jsx-a11y | `react-a11y` |

## Rules

- Do not add a dependency that duplicates an existing stack concern.
- New dependencies require justification and approval (see `react-architecture`).
- Keep the stack aligned with `react-feature` (Tailwind, Zustand, React Query, Axios in `apis/`) and `react-shared` (shadcn/ui).
- Forms use `react-hook-form` + `Zod` with `zodResolver`; derive form types with `z.infer`.
- Data access goes through the shared transport client (`shared/apis/<client>/`); services live in `apis/services/` and React Query in `apis/queries/` and `apis/mutations/`.
- Internationalization uses `react-i18next`/`i18next`; there is no dedicated skill yet — follow the library docs and keep translation keys in `locales/` (or `public/locales/`).

## References

| Situation | Read |
| --- | --- |
| Feature structure and data layer | `react-feature` |
| Shared components and shadcn location | `react-shared` |
| Layer rules and dependency approval | `react-architecture` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |