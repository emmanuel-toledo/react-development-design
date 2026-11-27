---
name: react-shared
description: Use when adding, moving or reviewing code in src/shared, deciding whether something belongs in shared, or using shared components, hooks, stores, types or shadcn/ui. Covers the shared tree, promotion gate and naming. Load this before touching src/shared.
---

# React Shared

Owns **the `src/shared` folder**: generic infrastructure reused across features. Load this before adding, moving or reviewing anything in `src/shared`.

## When to use

- Adding or moving a component, hook, store, type, util, constant or API module to `src/shared`.
- Deciding whether code is generic enough for `shared`.
- Using `shadcn/ui` components.
- Reviewing a change that touches `src/shared`.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For the library stack, read `react-stack`.

## Canonical tree

```text
src/shared/
  components/
    ui/
    layout/
    feedback/
    forms/
    routes/         # route guards (RequireAuth, RequireRole), only if needed
    shadcn/
      ui/
  pages/
  stores/
  hooks/
  interfaces/
  types/
  utils/          # only if needed
  config/         # only if needed (env.ts)
  apis/           # only if needed
    services/
    queries/
    mutations/
    <client>/     # transport clients (client.ts), e.g. axios, firebase
  constants/      # only if needed
```

**No empty folders.** Create a subfolder only when it has at least one file. `utils/`, `apis/` and `constants/` are created on demand.

## Folder rules

| Folder | Put here | Not here |
| --- | --- | --- |
| `components/ui/` | Generic UI primitives (buttons, inputs, cards) | Domain-specific components |
| `components/layout/` | Shared layouts, page wrappers | Feature layouts |
| `components/feedback/` | Toasts, alerts, spinners, empty states | Feature-specific feedback |
| `components/forms/` | Generic form inputs and wrappers | Domain forms |
| `components/routes/` | Route guards (`RequireAuth`, `RequireRole`) | Domain route logic |
| `components/shadcn/ui/` | shadcn/ui generated components | Hand-written primitives (use `components/ui/`) |
| `pages/` | Generic pages: `NotFound`, `Error`, `Loading`, `Maintenance` | Feature pages |
| `stores/` | Global Zustand stores (theme, session, UI state) | Feature-specific stores |
| `hooks/` | Generic hooks reused by 2+ features | Feature hooks |
| `interfaces/` | Shared object contracts | Domain interfaces |
| `types/` | Shared unions, aliases, utility types | Domain types |
| `utils/` | Generic pure functions and helpers | Domain logic |
| `config/` | Environment config (env.ts) — read `import.meta.env` here only | Domain config |
| `apis/` | Transport clients (`<client>/client.ts`, e.g. axios, firebase) always shared; promoted data modules (`services/`, `queries/`, `mutations/`) only via promotion gate | Feature APIs (stay in `features/<f>/apis/` unless promoted) |
| `constants/` | Shared constants | Feature constants |

## shadcn/ui rule

- Prefer `shadcn/ui` components first. Only fall back to Radix UI (or a custom primitive) when shadcn does not cover the need.
- shadcn components live in `src/shared/components/shadcn/ui/`.
- If shadcn components are found in any other location while reviewing code, **report it to the user** so they can decide whether to move them to the canonical path or leave them where they are. Do not move them automatically.
- Do not edit generated shadcn files beyond theming; wrap them in `components/ui/` or `components/forms/` when domain-specific behavior is needed.

## State

- Use Zustand for shared global state. No React Context in `shared` (Context is not used for shared state).
- Stores live in `src/shared/stores/`.

## Promotion gate

Before moving anything to `src/shared`, it must be:

1. Used by at least 2 features (or by the app and 1 feature).
2. Domain-agnostic (no domain types such as `User`, `Product`).
3. Imported by no feature internals.

Document the promotion: component, current consumers, why it is domain-agnostic, which features will use it, proposed public API, and confirmation that it imports no feature or domain type. Without approval, the component stays in its feature.

## Naming

- Files: `kebab-case` with role suffix where applicable (`.store.ts`, `.interface.ts`, `.type.ts`).
- Components: `PascalCase` named exports.
- Hooks: `useSomething`, file `useSomething.ts`.
- Types and interfaces: `PascalCase`, no `I` prefix.
- Constants: `UPPER_SNAKE_CASE`.
- Tailwind classes only.

## Imports

- `shared` never imports `app` or `features`.
- Features import shared via `@/shared/...`.
- Use `import type` for types.

## Anti-patterns (do not)

- Domain types or feature imports in `shared`.
- Empty folders.
- Internal barrels inside `shared` subfolders.
- Duplicating a feature component in `shared` without promotion approval.
- Editing shadcn generated files for domain logic.

## Checklist

- [ ] The item is used by 2+ features (or app + 1 feature).
- [ ] It is domain-agnostic and imports no feature or domain type.
- [ ] Promotion is documented and approved.
- [ ] No empty folders were created.
- [ ] shadcn components live under `components/shadcn/ui/`.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |