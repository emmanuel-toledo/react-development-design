---
name: react-effects
description: Use when writing or reviewing React code that may use useEffect, useLayoutEffect, useInsertionEffect or useEffectEvent, or when deciding whether an effect is needed. Effects are prohibited by default and require explicit approval. Load this before adding any effect.
---

# React Effects

Owns **the policy for React effects**. Load this before writing or reviewing any `useEffect`, `useLayoutEffect`, `useInsertionEffect` or `useEffectEvent`.

## Rule

`useEffect`, `useLayoutEffect`, `useInsertionEffect` and `useEffectEvent` are **prohibited by default**.

An effect is allowed only when all of these are true:

1. It synchronizes with an external system that React does not control.
2. The alternatives listed below do not solve the problem.
3. The project owner approved it before the code was written.
4. It is recorded in `docs/decisions/effects.md` with an ID, reason, cleanup and test.
5. The call site has an inline `// effect: EFF-XXX` comment.

If approval is missing, **stop and ask**. Do not add the effect, do not add a row to `docs/decisions/effects.md`, and do not bypass the rule.

## Do not use an effect for

- Deriving state from props or state.
- Synchronizing two state values.
- Fetching data.
- Responding to a user event.
- Resetting state when a prop changes.
- One-time initialization that can be done during render or with a ref callback.

## Alternatives (use these instead)

1. **Derive during render.** Compute the value from props/state instead of mirroring it in `useState` + effect. *Why*: avoids extra renders and duplicated state; the value stays in sync automatically. *When*: filters, totals, formatting, derived flags.

2. **Event handlers.** Respond to user interaction in the handler, not in an effect reacting to state changes. *Why*: logic runs at the moment of the event; predictable and testable. *When*: submit, click, change handlers.

3. **`key` to reset state.** Change the `key` so React recreates the component and resets its state. *Why*: no effect needed; React handles teardown/setup. *When*: switching entity/user and clearing local form state.

4. **`useSyncExternalStore`.** Subscribe to an external store safely (SSR/concurrent). *Why*: React manages subscription and snapshot. *When*: external stores, `window`/media queries, browser APIs.

5. **Ref callbacks.** Run code when a DOM node mounts/unmounts. *Why*: no effect; tied to the node lifecycle. *When*: measuring dimensions, focusing inputs, initializing third-party widgets.

6. **Route loaders (React Router).** Load data before render. *Why*: data is ready on render; no loading state in components. *When*: initial data for a route. Note: the default in this stack is client-side data fetching with React Query (see `react-data`/`react-routing`). Use a route loader only for initial route data with a documented reason; do not mix loaders and React Query without a clear reason.

7. **React Query (TanStack Query).** Server state with cache, deduplication and revalidation. *Why*: no fetch-in-effect; handles loading/error/retry. *When*: any API read or mutation.

8. **`useMemo` / `useCallback`** only for measured cost or required stable identity. *Why*: avoids unnecessary recomputation; do not use by default. *When*: expensive calculations, child memoization.

9. **`useEffectEvent` (React 19+)** to read non-reactive values inside an approved effect. Requires React 19 or later (stack uses the most stable React). *Why*: extracts logic that must not re-trigger the effect, avoiding stale closures. *When*: only inside an already-approved effect; it does not remove the approval requirement.

## Allowed exceptions (with approval)

- Synchronization with an external system React does not control: WebSocket, DOM APIs, third-party widgets, analytics, non-declarative timers.
- Manual subscriptions not covered by `useSyncExternalStore`.

## Documentation

Approved effects must be recorded in `docs/decisions/effects.md` with: ID, reason, cleanup and test. The call site must include `// effect: EFF-XXX`.

Example row:

```md
| EFF-001 | Sync WebSocket connection | cleanup closes socket | WebSocket.test.tsx |
```

## Checklist

- [ ] No `useEffect`/`useLayoutEffect`/`useInsertionEffect`/`useEffectEvent` added without approval.
- [ ] Alternatives were considered and ruled out.
- [ ] Approval obtained before writing code.
- [ ] Row added to `docs/decisions/effects.md` with ID, reason, cleanup and test.
- [ ] Inline `// effect: EFF-XXX` comment present.
- [ ] Cleanup implemented where needed.
- [ ] Tests cover the effect behavior.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared folder and components | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy and coverage | `react-testing` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |