---
name: react-architecture
description: Use when adding, moving, or reviewing modules, features, imports, environment configuration, tests placement, or public APIs in a React app. Owns where code lives and who may import it. Load this before structural changes.
---

# React Architecture

Owns **where code lives and who may import it**. Load this before structural changes or when reviewing a change that touches modules, features, layers, environment configuration, or public APIs.

## Source of truth

1. The active spec in `docs/specs/`.
2. `AGENTS.md`.
3. This skill.
4. The executable checks (`lint`, `typecheck`).

If two sources conflict, stop and report the conflict. Do not silently choose one.

## Layer model

```text
app → features → shared
```

- `src/app` is the composition root: app-level constants, providers, layouts and styles. It imports features only through `@/features/<feature>` and may import `shared`. It never contains domain logic or reaches into feature internals.
- `src/features/<feature>` is an isolated domain module. Only its root `index.ts` is public. It may import `shared`. It must not import `app`, another feature's internals, or create cycles.
- `src/shared` is generic infrastructure. It never imports `app` or `features` and never knows domain types. See `react-shared` for its structure and promotion rules.
- `src/routes` (`app.router.tsx`, `protected.router.tsx`) is part of the app/composition layer: it imports feature routes through their public API.

## Current feature graph

```text
app/routes → features
features → shared
all layers → shared
```

By default there are **no cross-feature dependencies**. A new cross-feature dependency requires an approved spec and an update to the graph. Do not add it just because an import compiles.

## Where does this code belong?

| If the code... | It belongs in... |
| --- | --- |
| Composes the application: routes, providers, layouts, app-level constants | `src/app` / `src/routes` |
| Is specific to one domain | `src/features/<feature>` |
| Is generic and used by two features, or by app and a feature | `src/shared` (see `react-shared`) |
| Reads `import.meta.env` | `src/shared/config/env.ts` only |
| Queries or writes a feature's external service | The owning feature's `apis/` |

If the answer is ambiguous, stop and ask before moving code.

## Public API rules

1. Every feature has a root `index.ts`. It is the only public entry point.
2. Cross-feature and app imports use the exact path `@/features/<feature>`. Never use `@/features/<feature>/components/...`, `.../hooks/...`, `.../index` or a relative path that crosses a feature boundary.
3. The public API is minimal: export only the components, hooks, types and contracts another layer actually needs. Pages and route modules may be exported when the app needs them; internal API clients stay internal.
4. If a consumer needs something that is not exported, first decide whether it is generic enough for `shared`; otherwise export it explicitly from the owning feature.
5. Only the feature root `index.ts` is a barrel. Do not add internal `index.ts` barrels.
6. No cycles between features. If two features need each other, extract the common part to `shared` or redesign the split.
7. Feature tests may import their own internals. Integration tests between features must use the public API.

## Environment

- `import.meta.env` is read only in `src/shared/config/env.ts`; the rest of the code imports the validated object.

## Tests and naming

- Application tests live under `tests/`, mirroring `src/`: `tests/app`, `tests/shared`, `tests/features/<feature>`.
- Do not put `*.test.ts(x)` under `src/`.
- A feature test may import its own internals; integration tests between features use the public API.
- Files are `kebab-case`; component exports are `PascalCase`; hooks are `useSomething`; types are `PascalCase` without an `I` prefix; constants are `UPPER_SNAKE_CASE`.
- Feature files use role suffixes: `.service.ts`, `.queries.ts`, `.mutations.ts`, `.store.ts`, `.interface.ts`, `.type.ts`, `.layout.tsx`, `.routes.tsx`.
- Use `import type` for types and the `@/` alias for cross-layer imports.

## New feature

Read `react-feature` for the canonical tree. At minimum:

1. Approve the feature and its public API in a spec.
2. Create `src/features/<feature>/index.ts`.
3. Add only the internal folders the feature actually needs; no `providers/` or `schemas/`.
4. Put HTTP access in `apis/services/`, React Query in `apis/queries/` and `apis/mutations/`, Zustand stores in `stores/`, orchestration in `hooks/`, domain logic in `utils/`, and UI in `components/` or `pages/`.
5. Add critical tests under `tests/features/<feature>/` and update the spec and feature graph.
6. Run lint and typecheck.

## Validation

```bash
pnpm lint
pnpm typecheck
```

## Stop and ask

Ask before:

- Adding or changing a cross-feature dependency.
- Changing a feature's public API.
- Moving code between `app`, a feature and `shared` when ownership is ambiguous.
- Adding a dependency.
- Changing the layer structure, boundary rules or the feature graph.

## References

| Situation | Read |
| --- | --- |
| Canonical feature tree, naming, routes, stores | `react-feature` |
| Shared folder structure and promotion gate | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |