---
name: react-performance
description: Use when optimizing or reviewing React performance code splitting, lazy loading, memoization, list virtualization, concurrency, image optimization, Web Vitals, profiling or bundle budgets. Load this before adding heavy components, large lists or expensive computations.
---

# React Performance

Owns **performance** in a React application: code splitting, lazy loading, memoization, virtualization, concurrency, images, Web Vitals, budgets and profiling. Load this before adding heavy components, large lists or expensive computations.

## When to use

- Adding or reviewing a heavy component, large list or expensive computation.
- Deciding code splitting, lazy loading or memoization.
- Optimizing images, loading states or bundle size.
- Profiling or debugging slow renders.
- Setting performance budgets.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared components, read `react-shared`. For the library stack, read `react-stack`. For routing and lazy routes, read `react-routing`. For a11y of fallbacks, read `react-a11y`. For data caching/prefetch, read `react-data`. For testing, read `react-testing`. For effect policy, read `react-effects`.

## Principle: measure first

- **Profile before optimizing.** Identify the bottleneck, apply the smallest change, re-measure.
- Premature optimization is an anti-pattern. Do not add memoization, virtualization or splitting without evidence.

## Code splitting and lazy loading

- Route-level splitting is owned by `react-routing`/`react-feature` (lazy pages, one Suspense per feature layout). This skill does **not** redefine that; it references it.
- Component-level: use `React.lazy` + `Suspense` for heavy, non-critical widgets (charts, editors, rich modals).
- Library-level: dynamic `import('lib')` inside the event handler, not in the initial bundle.
- Prefetch on hover/intent for critical routes and components.
- Place Suspense boundaries with intent: not too high (blocks whole UI) nor too low (spinner waterfalls).

## Suspense and fallbacks

- Fallbacks must be accessible: `aria-busy`, skeletons, not just spinners (see `react-a11y`).
- Avoid waterfalls: load data in parallel, not sequentially.
- Wrap lazy components in error boundaries (see `react-routing`).

## Memoization (justified)

- `React.memo`: only for components that receive the same props frequently and whose render is costly. Requires profiling evidence.
- `useMemo`: expensive calculations, or stable identity required by memoized children / hook deps.
- `useCallback`: only for stable identity passed to memoized children or used in approved effect deps.
- **Not by default.** Aligned with `react-effects` (#8): memoization is a measured decision, not a default.
- Pitfalls: inline objects/arrays/functions in props break `React.memo`. Stabilize with `useMemo`/`useCallback` only when justified.
- React Compiler is out of scope for now.

## Virtualization of lists

- Virtualize when a list exceeds **100 items** (or measured slow render).
- Recommended library: `@tanstack/react-virtual` (new dependency → requires approval per `react-architecture`).
- Accessibility: virtualized lists need `aria-setsize`/`aria-posinset` and correct roles (see `react-a11y`).
- Do not virtualize small lists.

## Concurrency

- `useTransition` for non-urgent updates (filters, search) keeping the UI responsive.
- `useDeferredValue` to defer expensive render values.
- `startTransition` to mark updates as non-urgent.
- Do not use them to mask render problems without measuring first.

## Image optimization

- `loading="lazy"` + `decoding="async"` for below-the-fold images.
- `srcset`/`sizes` to serve the right size.
- Modern formats (WebP/AVIF) when the pipeline supports them.
- Priority (`fetchpriority="high"` / `loading="eager"`) for the LCP image.
- Explicit `width`/`height` to avoid CLS.

## Data and cache (reference `react-data`)

- `staleTime`, prefetching, `placeholderData`/`keepPreviousData`, React Query deduplication.
- Avoid unnecessary refetches; configure `refetchOnWindowFocus` with intent.
- Do not duplicate server state in Zustand.

## Render and DOM

- Avoid layout thrashing (read/write DOM in loops).
- Avoid massive re-renders: push state down, split components.
- Stable keys in lists (no index as key in dynamic lists).
- Never create components inside components (new identity every render).

## Web Vitals (reference)

- LCP < 2.5s, CLS < 0.1, INP < 200ms.
- Monitoring is optional (e.g. `web-vitals` lib, RUM) and requires approval.

## Budgets (recommended)

- Set bundle size budgets per route/chunk (e.g. initial JS < 200KB gzipped).
- Alert in CI when a budget is exceeded (non-blocking recommended at first).
- Track and review regressions in PRs.

## Profiling

- React DevTools Profiler: identify slow renders and causes.
- `why-did-you-render` (dev, optional, requires approval) to detect unnecessary re-renders.
- Bundle analyzer (dev, optional) for bundle size.
- Measure in production-like conditions when possible, not only dev.

## Anti-patterns (do not)

- Premature optimization without measurement.
- `useMemo`/`useCallback`/`React.memo` by default.
- Virtualizing small lists.
- Excessive code splitting (waterfalls).
- Inline objects/arrays/functions in props of memoized components without need.
- Creating components inside components.
- Ignoring stable keys.
- Measuring only in dev.

## Checklist

- [ ] Profiling done before and after the change.
- [ ] Heavy routes/components lazy-loaded (coordinated with `react-routing`/`react-feature`).
- [ ] Suspense fallbacks accessible.
- [ ] Memoization justified and documented.
- [ ] Lists > 100 items virtualized.
- [ ] Images optimized (lazy, srcset, dimensions).
- [ ] Concurrency used where it improves UX.
- [ ] Web Vitals within thresholds (if monitored).
- [ ] Budgets respected (if defined).
- [ ] Lint and typecheck pass.

## Template

### Lazy component with accessible Suspense

```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

export function Dashboard() {
  return (
    <Suspense fallback={<div role="status" aria-busy="true" className="skeleton" />}>
      <HeavyChart />
    </Suspense>
  );
}
```

### Justified memoization

```tsx
import { memo, useMemo, useCallback } from 'react';

interface Props { items: Item[]; onSelect: (id: string) => void; }

// Justified: renders often with same props and render is costly (profiling evidence).
export const ItemList = memo(function ItemList({ items, onSelect }: Props) {
  const sorted = useMemo(() => [...items].sort((a, b) => a.name.localeCompare(b.name)), [items]);
  const handleSelect = useCallback((id: string) => onSelect(id), [onSelect]);

  return (
    <ul>
      {sorted.map((item) => (
        <li key={item.id}>
          <button onClick={() => handleSelect(item.id)}>{item.name}</button>
        </li>
      ))}
    </ul>
  );
});
```

### Virtualized list

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

export function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((row) => (
          <div
            key={row.key}
            data-index={row.index}
            ref={virtualizer.measureElement}
            style={{ position: 'absolute', top: row.start, width: '100%' }}
            aria-setsize={items.length}
            aria-posinset={row.index + 1}
          >
            {items[row.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Concurrency (responsive filter)

```tsx
import { useState, useTransition } from 'react';

export function Search({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('');
  const [filtered, setFiltered] = useState(items);
  const [isPending, startTransition] = useTransition();

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // urgent: input stays responsive
    startTransition(() => {
      setFiltered(items.filter((i) => i.name.includes(value)));
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <span aria-live="polite">Filtrando…</span>}
      <ul>{filtered.map((i) => <li key={i.id}>{i.name}</li>)}</ul>
    </>
  );
}
```

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared components and shadcn location | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |