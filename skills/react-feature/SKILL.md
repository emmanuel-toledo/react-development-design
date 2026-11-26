---
name: react-feature
description: Use when creating, restructuring or reviewing a feature in a React app, deciding where a file belongs inside a feature, or changing a feature public API. Covers the canonical feature tree, naming, lazy routes, Zustand stores, React Query data layer and the shared promotion gate. Load this before creating a feature or adding a folder.
---

# React Feature

Owns **the anatomy of a feature** in a React application. Load this before creating a feature, adding a folder, moving a component or changing a feature's `index.ts`.

## When to use

- Adding a new feature under `src/features/`.
- Adding a page, layout, router, store, interface, type, hook, component or API module to a feature.
- Deciding whether a component belongs to a feature or to `src/shared`.
- Changing what `@/features/<feature>` exports.
- Reviewing a change that crosses a feature boundary.

For tests, read the testing skill; this skill only states where tests do **not** go. For the shared folder, read `react-shared`. For the library stack, read `react-stack`.

## Canonical tree

```text
src/features/<feature>/
  index.ts
  apis/
    services/
    queries/
    mutations/
  components/
  pages/
    <Name>Page.tsx
  hooks/
    use<Name>.ts
  interfaces/
    <name>.interface.ts
  types/
    <name>.type.ts
  utils/
  layouts/
    <Feature>Layout.tsx
  routes/
    <feature>.routes.tsx
  stores/
    <feature>.store.ts
```

The tree is exhaustive. A new top-level folder requires justification. **No empty folders**: create a folder only when it has a responsibility.

## Folder rules

| Folder        | Put here                                                                                       | Not here                                    |
| ------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `apis/services/`   | HTTP clients, Axios calls, connection to external services, mappers                             | UI, React, component state                  |
| `apis/queries/`    | React Query `useQuery` wrappers and query keys                                                 | Mutations, UI                              |
| `apis/mutations/`  | React Query `useMutation` wrappers                                                             | Queries, UI                                |
| `components/`       | Feature-only UI; local props in the same file                                                  | Data access, cross-feature contracts        |
| `pages/`           | Route screens named `<Name>Page.tsx`; compose components and hooks                             | Business logic, direct API access          |
| `stores/`          | Zustand stores; files end in `.store.ts`                                                       | Server state duplicated from React Query   |
| `interfaces/`      | Object contracts shared inside the feature or exported publicly; `.interface.ts`               | Local component props                      |
| `types/`           | Unions, aliases, inferred form values, utility types; `.type.ts`                               | Runtime values                             |
| `utils/`           | Pure functions, helpers, classes                                                               | React imports, API access                  |
| `layouts/`         | Feature layout; `.tsx` with `Layout` suffix; owns the feature's `<Suspense>`                   | Extra layouts without justification        |
| `routes/`          | Single `<feature>.routes.tsx`; lazy pages; exports `<feature>Routes`                           | Route components defined inline            |
| `hooks/`           | Real hooks named `use<Name>.ts`; orchestration and selectors                                   | Domain calculations that belong in `utils/` |
| `index.ts`         | The only public entry point and the only barrel                                                | Internal helpers                           |

No `providers/`. No `schemas/` (Zod schemas live in `utils/` and form types are inferred with `z.infer`). No internal `index.ts` barrels.

## Public API

1. `index.ts` is the contract. Export only what the app or another feature actually needs.
2. App imports only `@/features/<feature>`. Never `@/features/<feature>/pages/...` or any deep path.
3. Export pages, routes and public components. Keep stores, services, schemas and error helpers internal unless explicitly needed.
4. Only the feature root is a barrel. No internal `index.ts`.
5. No cycles. A shared need is extracted to `src/shared` (see `react-shared`) with approval.
6. Changing the public API is a contract change: update the spec first.

```ts
// src/features/users/index.ts
export { usersRoutes } from './routes/users.routes';
export { UsersPage } from './pages/UsersPage';
export { UserCard } from './components/UserCard';

export type { User } from './interfaces/user.interface';
export type { UserStatus } from './types/user.type';
```

## Routes

- One router file per feature **when the feature exposes pages/routes**: `routes/<feature>.routes.tsx`.
- If a feature exposes no routes, do **not** create the `routes/` folder (no empty folders; see canonical tree).
- Lazy-load pages with `lazy(() => import('../pages/<Name>Page'))`.
- Export `<feature>Routes` as `RouteObject[]`.
- The feature layout provides `<Suspense>` and, optionally, a local section landmark (`section`/`article`). Top-level landmarks (header, nav, main, footer, skip link) belong to the app/root layout (see `react-a11y`).
- The app composes the feature routes; it does not import pages directly.

## Naming

- Files: `kebab-case` for folders; components `PascalCase.tsx`; hooks `use<Name>.ts`; other files `camelCase` with role suffix (`.service.ts`, `.queries.ts`, `.mutations.ts`, `.store.ts`, `.interface.ts`, `.type.ts`, `.routes.tsx`, `.layout.tsx`).
- Components: `PascalCase` named exports.
- Hooks: `useSomething`, file `use-something.ts` → `useSomething.ts` (camelCase with `use` prefix).
- Types and interfaces: `PascalCase`, no `I` prefix.
- Constants: `UPPER_SNAKE_CASE`.
- Tailwind classes only; no CSS files per component.

## Imports

- Use the `@/` alias pointing to `src/`.
- No deep imports into another feature.
- Order: external → `@/shared` → feature internals → types.
- Use `import type` for types.

## State and data

- Feature-wide state lives in a Zustand store under `stores/<feature>.store.ts`.
- Server state uses React Query; queries in `apis/queries/`, mutations in `apis/mutations/`, HTTP in `apis/services/`.
- Hooks in `hooks/` orchestrate queries, mutations and store access.
- Components never call APIs directly; they consume hooks.
- Keep state close to where it is used; lift only when needed.

## Styling

- Tailwind CSS. No per-component style files.
- Shared UI primitives come from `src/shared/components` (see `react-shared`).

## Interfaces vs types

- `interfaces/`: object contracts (`.interface.ts`).
- `types/`: unions, aliases, inferred form values, utility types (`.type.ts`).

## Anti-patterns (do not)

- `providers/`, `schemas/`, empty folders, internal barrels.
- Deep imports across features.
- Business logic or API access in components.
- Duplicating server state in Zustand.
- Tests under `src/` (tests live in `tests/`).
- Importing any external SDK directly in components.

## Template

```text
src/features/users/
  index.ts
  apis/
    services/
      user.service.ts
    queries/
      user.queries.ts
    mutations/
      user.mutations.ts
  components/
    UserCard.tsx
  pages/
    UsersPage.tsx
    UserDetailPage.tsx
  hooks/
    useUsers.ts
  interfaces/
    user.interface.ts
  types/
    user.type.ts
  utils/
    formatUserName.ts
  layouts/
    UsersLayout.tsx
  routes/
    users.routes.tsx
  stores/
    users.store.ts
```

## Checklist

- [ ] Feature boundary and public API are defined.
- [ ] Tree matches the canonical layout; no extra top-level folders.
- [ ] No empty folders.
- [ ] `index.ts` exports the minimal contract and nothing internal.
- [ ] No `providers/`, no `schemas/`, no internal barrels.
- [ ] If the feature exposes routes, there is exactly one router file `routes/<feature>.routes.tsx`; otherwise no `routes/` folder exists.
- [ ] Pages are lazy-loaded from the single router file.
- [ ] One layout owns `<Suspense>`.
- [ ] API access only in `apis/`; components consume hooks.
- [ ] Shared needs went through the promotion gate (see `react-shared`).
- [ ] Tests live under `tests/features/<feature>/`, not next to source.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Layer rules, imports, feature graph, public API | `react-architecture` |
| Shared folder structure and promotion | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |