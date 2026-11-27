---
name: react-routing
description: Use when defining or reviewing routes in a React app, paths, params, search params, guards, lazy loading, nested layouts, 404, redirects, scroll restoration, error boundaries or URL-as-source-of-truth. Load this before adding or changing a route.
---

# React Routing

Owns **how routes are defined** in a React application. Load this before adding or changing a route.

## When to use

- Adding or reviewing a route, param, search param or redirect.
- Setting up guards, layouts, 404 or error boundaries.
- Deciding where routing files live.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared pages, read `react-shared`. For the library stack, read `react-stack`. For forms, read `react-forms`. For effects, read `react-effects`. For testing, read `react-testing`.

## Conventions

### Paths
- kebab-case, plural nouns: `/users`, `/user-details`, `/orders/:orderId`.
- Route params in camelCase with `Id` suffix: `:userId`, `:orderId`.

### Files
- `src/routes/app.router.tsx` composes feature routes.
- `src/routes/protected.router.tsx` centralizes guards.
- Feature routes: `features/<f>/routes/<f>.routes.tsx` exporting `<f>Routes` (mandatory when the feature exposes pages; omit the folder otherwise).

## Search params (URL as source of truth)
- Filters, pagination and search live in the URL.
- Use `nuqs` for type-safe, serializable search params.
- Keep state shareable and persistent across reloads.

## Guards
- Reusable components: `RequireAuth`, `RequireRole`, located in `shared/components/routes/`.
- Criteria (auth, roles, both, or none) is **decided by the project**; guards must be composable to cover each case.
- Centralize in `protected.router.tsx`.
- Redirect to login or access-denied page accordingly.

## 404
- `NotFound` page lives in `shared/pages/`.
- Add a catch-all route (`path: '*'`) rendering `NotFound` at the end of the route tree.

## Nested layouts
- `RootLayout` (app) plus feature layouts nested via `<Outlet />`.
- Feature layouts live in `features/<f>/layouts/`.

## Lazy loading
- Lazy-load pages in the feature router file.

## Scroll restoration
- Enable `ScrollRestoration` from React Router.

## Error boundaries
- `ErrorBoundary` per route/feature to isolate failures.

## Accessibility (route-level)
- Skip link "Saltar al contenido principal" in the root layout (see `react-a11y`).
- Move focus to `main` on every navigation.
- Top-level landmarks (header, nav, main, footer) live in the root layout, not per feature.

## Redirects
- Use the `redirect()` helper and named routes; avoid magic strings.

## Data fetching
- Client-side with React Query. Do not mix React Router loaders with React Query without a clear reason.

## Template (`features/<feature>/routes/<feature>.routes.tsx`)

```tsx
import { lazy } from 'react';
import type { RouteObject } from 'react-router';

const FeatureLayout = lazy(() => import('../layouts/FeatureLayout'));
const ListPage = lazy(() => import('../pages/ListPage'));
const DetailPage = lazy(() => import('../pages/DetailPage'));

export const featureRoutes: RouteObject[] = [
  {
    path: 'features',
    element: <FeatureLayout />,
    errorElement: <RouteErrorBoundary />,
    children: [
      { index: true, element: <ListPage /> },
      { path: ':featureId', element: <DetailPage /> },
    ],
  },
];
```

Notes:
- Pages are lazy-loaded; the feature layout owns `<Suspense>`.
- `errorElement` isolates failures per route (see Error boundaries). `RouteErrorBoundary` is a shared component (e.g. `shared/components/RouteErrorBoundary.tsx`).
- Guards (auth/roles) are applied in `protected.router.tsx`, not here, unless the feature needs a local guard.

## Anti-patterns (do not)

- Magic strings for paths (use constants/named routes).
- State for filters/pagination in memory when it should be in the URL.
- Guards hardcoded to a single criterion.
- Fetching in route loaders without a clear reason.
- Missing 404 or error boundary.

## Checklist

- [ ] Paths kebab-case, plural; params camelCase with `Id`.
- [ ] Search params typed with `nuqs` and reflected in the URL.
- [ ] Guards centralized, criterion decided by the project.
- [ ] 404 in `shared/pages/NotFound`.
- [ ] Nested layouts with `<Outlet />`.
- [ ] Every feature that exposes pages has exactly one `routes/<feature>.routes.tsx`; features without routes have no `routes/` folder.
- [ ] Pages lazy-loaded.
- [ ] `ScrollRestoration` enabled.
- [ ] `ErrorBoundary` per route/feature.
- [ ] Skip link + focus-to-`main` on navigation (see `react-a11y`).
- [ ] Redirects use helper + named routes.
- [ ] Data fetching with React Query.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared pages and components | `react-shared` |
| Library stack and versions | `react-stack` |
| Forms (RHF + Zod) | `react-forms` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Testing strategy and coverage | `react-testing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |