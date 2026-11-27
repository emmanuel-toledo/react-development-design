---
name: react-a11y
description: Use when creating or reviewing UI for accessibility semantic HTML, labels, focus, keyboard, ARIA, contrast, skip links, landmarks, reduced motion, modals or loading states. Makes the app usable by everyone. Load this before adding or changing UI.
---

# React Accessibility

Owns **accessibility (a11y)** in a React application: semantic HTML, labels, focus, keyboard, ARIA, contrast, skip links, landmarks, reduced motion and modals. Load this before adding or changing UI.

## When to use

- Creating or reviewing a component, page or layout.
- Adding forms, modals, dynamic content or loading states.
- Deciding landmark structure, skip links or focus behavior.
- Reviewing contrast, keyboard support or ARIA usage.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared components and shadcn/Radix, read `react-shared`. For the library stack and tooling, read `react-stack`. For forms, read `react-forms`. For routing and layouts, read `react-routing`. For testing (axe), read `react-testing`. For effects (focus must not use useEffect by default), read `react-effects`. For data loading states, read `react-data`.

## Principles

### 1. Semantic HTML first
- Use the right element: `button`, `a`, `nav`, `main`, `header`, `footer`, `aside`, `section`, `article`, `form`, `label`, `table`, `ul`/`ol`.
- Never use `div`/`span` with `onClick` for interactive elements.
- One `<h1>` per page; heading hierarchy without skips.
- Lists for lists, tables for tabular data. Set `lang` on `<html>`.

### 2. Labels and accessible names
- Every form control has an accessible name (visible `label` preferred; `aria-label`/`aria-labelledby` only when no visible label).
- Associate `label` with `htmlFor`/`id` or by nesting the control.
- Links and buttons use descriptive text (avoid "click here", "read more" without context).
- Images: descriptive `alt`; decorative images use `alt=""`.

### 3. ARIA (use sparingly)
- Use ARIA only when semantic HTML is insufficient.
- Do not override native semantics unnecessarily.
- `aria-live` (polite/assertive) for dynamic content (toasts, search results, loading).
- `aria-hidden` for decorative elements.
- `aria-expanded`, `aria-controls`, `aria-haspopup` for widgets (menus, accordions, tabs).
- Roles only when needed (`role="dialog"`, `role="tablist"`, etc.).
- Follow the ARIA Authoring Practices (APG) for composite widgets.

### 4. Keyboard
- Everything interactive is focusable and operable with the keyboard.
- Logical tab order; avoid positive `tabindex` (use 0 or -1).
- Activation with Enter/Space; Escape to close; arrows in composite widgets (tabs, menus, lists).
- No keyboard traps (focus trap only in modals, with an exit).
- Roving tabindex for composite widgets.

### 5. Focus
- Focus is always visible (`:focus-visible`); never remove `outline` without a replacement.
- Modals/dialogs: focus trap + initial focus + restore focus on close.
- Forms: focus the first error via `setFocus` in the submit handler (event-driven), not an effect.
- Route changes: move focus to `main` on navigation.
- Use ref callbacks, `autoFocus` or imperative calls in handlers. **Do not use `useEffect` for focus by default**; if an effect is truly required (e.g. focus after async load or route change), follow the approval process in `react-effects` (`docs/decisions/effects.md` + `// effect: EFF-XXX`). The template below uses an approved effect for route-change focus.
- Skip link "Saltar al contenido principal" at the top of the root layout, visible on focus.

### 6. Contrast
- WCAG AA minimum: 4.5:1 normal text, 3:1 large text and UI components.
- Verify in light and dark themes when applicable.
- Color is never the only way to convey information (add icons/text).

### 7. Landmarks and page structure
- Use `header`, `nav`, `main`, `footer`, `aside`; one `main` per page.
- Skip link targets `main`.
- Landmarks of top level (header/nav/main/footer/skip link) live in the **app/root layout**, not per feature. Feature layouts provide `<Suspense>` and, optionally, a local section landmark (`section`/`article`) — see `react-feature`.
- The feature layout does not own the top-level landmark structure.

### 8. Motion and media
- Respect `prefers-reduced-motion` (mandatory): disable animations/transitions.
- Tailwind: use the `motion-reduce:` variant.
- Media has controls; no autoplay without a pause option.

### 9. Dynamic states and loading
- Use `aria-busy` and `aria-live` regions for async updates (loading, search results).
- Toasts are announced (Sonner handles `aria-live`).
- Empty and error states are announced.

### 10. Modals and overlays
- `role="dialog"`, `aria-modal="true"`, `aria-labelledby`/`aria-describedby`.
- Focus trap + focus restore on close.
- Close with Escape and an explicit button.

### 11. Tables and data
- `th` with `scope`; `caption` when applicable; never use tables for layout.

### 12. Utilities
- Tailwind `sr-only` for screen-reader-only text.
- Minimum touch targets: 44×44 (recommended on mobile).

## Tooling
- `eslint-plugin-jsx-a11y` (dev, recommended; adding it requires approval per `react-architecture`).
- `axe-core` / `jest-axe` in tests (see `react-testing`).
- CI gate: recommended, non-blocking (axe in Playwright).
- Manual testing with screen readers (VoiceOver, NVDA) recommended for critical flows.

## Template: skip link + focus on route change

```tsx
// src/app/layouts/RootLayout.tsx (or shared layout)
import { useEffect, useRef } from 'react';
import { Outlet, useLocation } from 'react-router';

export function RootLayout() {
  const mainRef = useRef<HTMLElement>(null);
  const location = useLocation();

  // Move focus to main on navigation. This is an approved effect? Prefer
  // an event-driven approach if possible; otherwise follow react-effects.
  useEffect(() => {
    mainRef.current?.focus();
  }, [location.pathname]);

  return (
    <>
      <a
        href="#main-content"
        className="sr-only focus:not-sr-only focus:absolute focus:top-2 focus:left-2 focus:z-50 focus:rounded focus:bg-white focus:px-3 focus:py-2 focus:text-black"
      >
        Saltar al contenido principal
      </a>
      <header>...</header>
      <nav aria-label="Principal">...</nav>
      <main id="main-content" ref={mainRef} tabIndex={-1} className="outline-none">
        <Outlet />
      </main>
      <footer>...</footer>
    </>
  );
}
```

> Note: the focus-on-navigation effect above is one of the few cases where an effect may be justified. It must be approved and recorded per `react-effects` (`docs/decisions/effects.md` + `// effect: EFF-XXX`) before use.

## Anti-patterns (do not)
- `div onClick` instead of `button`.
- Removing focus outline without replacement.
- `aria-label` on non-interactive elements without need.
- Keyboard traps.
- Insufficient contrast.
- Ignoring `prefers-reduced-motion`.
- Fragmenting top-level landmarks across features.
- Inaccessible confirmation dialogs (e.g. unsaved-changes guard) — use a native `confirm` or an accessible dialog with focus trap.

## Checklist
- [ ] Semantic elements used correctly.
- [ ] Accessible names on all controls.
- [ ] Keyboard operable; focus visible.
- [ ] Contrast meets WCAG AA.
- [ ] Skip link present and functional.
- [ ] Landmarks correct (single `main`).
- [ ] `prefers-reduced-motion` respected.
- [ ] Modals with trap + focus restore.
- [ ] axe without violations on critical components.
- [ ] `lang` set on `<html>`.
- [ ] Lint (jsx-a11y) and typecheck pass.

## References
| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared components and shadcn/Radix location | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Data layer, clients, React Query, mocks | `react-data` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |